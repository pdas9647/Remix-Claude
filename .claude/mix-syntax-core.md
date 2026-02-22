# Mix Language — Core Syntax Reference

> Source: [Mix Syntax](https://www.notion.so/1061d464528f81d492f4e1b0f8c8adf4)
> Parent: [The Mix Programming Language](https://www.notion.so/259ede6504e34505982dde5dc4b63d10)

See also: [mix-syntax-expressions.md](./mix-syntax-expressions.md) | [mix-syntax-semantics.md](./mix-syntax-semantics.md) | [mix-syntax-style.md](./mix-syntax-style.md)

---

## Modules

```
module M                             // no params
  ...
module end

module M(p1:string, p2:number->data) // with params
  ...
module end
```

`module end` can be omitted when module is stored in a file; required in REPL.

### Imports

```
import P                          // access as P.name
import newname = P                // alias
import P(x, {headline:"This is M"}) // with params at import time
```

- `P.name` returns value if `name` is a def; returns a **link** if `name` is a cell (since PR#1331)
- If params not given, `P` becomes a function taking the missing params and returning the module
- `self.name` — reference something in current module (since PR#483)
- `global.name` — reference something in global scope (since PR#800)

### Declarations

```
declare P(x:number, r:data)   // declare expected parameter types for module P
```

---

## Spreadsheet (Cells)

### Defining cells

```
var cell c1 = <expr>           // assignable (var)
cell c2 = <expr>               // propagating (read-only)
private var cell c3 = <expr>   // private + assignable
private cell c4 = <expr>       // private + propagating
```

Only private cells can be optimized by the spreadsheet optimizer.

**Cells are invisible inside `def` bodies** — defs are pure functions with no spreadsheet dependencies (intentional). Workarounds:

- `contents(c)` — get current value of cell `c` without creating a dependency (works everywhere, including defs)
- When `appstate` is in scope — cell values are accessible
- Change `def` to `cell` when the computation genuinely depends on a cell:

```
var cell n = 0
def nBiggerThan20 = (_ -> n >= 20)    // ERROR: cell inaccessible in def
cell nBiggerThan20 = (_ -> n >= 20)   // CORRECT: cell formula can reference other cells
```

**Declaration before definition**:

```
cell c2 : number
var cell c1 : data
cell c2 : number = 5           // merged declaration+definition
```

**Function cell shorthand**:

```
cell c4(n) = "value: " + n                  // instead of: cell c4 = (n -> ...)
cell c4(n) { return "value: " + n }         // statement form
```

### Live cells

```
live cell x = db.head() |> ...    // repeating query; must start with db.head/all/query
```

### Aliases

```
alias x = y                // alias cell y in same module
alias x = lnk              // alias cell referenced by link lnk
alias x = M(p).y           // alias cell y inside M (must not be private)
```

Since PR#1235 aliases are always considered private; must follow a private declaration:

```
private cell x : number
alias x = y
```

### Links

> Source: [cells-links-and-aliases](https://www.notion.so/1061d464528f81cba83fddeb2e80a3af)

```
module S(l:link(string)<var>)
  alias x : string = l      // dereference the link
module end

module T
  var cell store = ""
  import S(store)            // pass var cell as link param
  alias S_view = S.view
module end
```

**Module link params vs data params**: passing a data value creates a new module instance each time the value changes (breaking existing cell links). Passing a `link(...)` parameter ties the module to the cell *identity*, not its value — stable spreadsheet connections survive value changes.

**Shared link pattern** — multiple components editing the same `var cell`:

```
import Slider
import TextField
var cell some_val = 0
cell link(slider_view) = link(Slider(link(some_val)).view)    // Slider edits some_val
cell link(text_view)   = link(TextField(link(some_val)).view) // TextField edits same cell
// Moving the slider updates the text field and vice versa
```

**Double-link alias syntax** — alias a cell from another module:

```
import X
var cell link(myx) = link(X.x)   // myx is an alias of X.x; reads/writes propagate
cell link(myy)     = link(X.y)   // myy is an alias of X.y (read-only cell)
```

### Initializers (since PR#598)

```
module M
  var cell x1 = 0
  initializer {
    let p = f(...);
    x1 = p.first
  }
module end
```

- Must be the **last** definition in the module
- `var cell`s are always defined when the initializer runs
- Common use: call `viewstack.schedule` to run an action shortly after module init

### Observers (since PR#732)

```
on x do { q = p }                     // runs after every evaluation of x
on (x,y,z) do { ... }                 // watch multiple cells
on x when x >= 20 do { ... }          // conditional: guard on the watched cell
on x when x'? do { ... }              // skip initial evaluation (x' = previous value)
on () do { ... }                       // run once per module instance (after init, similar to initializer)
```

**Special `when` conditions**:

- `cellUpdated(x)` — true when x has been updated since last run (also true on first run)
- `cellChanged(x)` — true when x has been changed since last run (also true on first run)
- Example: `on (x,y) when cellChanged(x) && cellChanged(y) do { ... }`

Notes:

- `on` triggers fire only once per user interaction (no loops via `on`)
- `when` check happens before debouncing; put value equality checks inside `do { if ... }` to apply after debounce
- Workaround for loops: use `viewstack.schedule` (creates a new user interaction)

### Special cell notations

- `x'` — previous value of cell x
- `contents(c)` / `contents(lnk)` — get current value of cell or link without creating a spreadsheet dependency (works everywhere, even inside `def` bodies)
- `link(x)` — create a link to cell x
- `cellName(x)` — internal unique name of cell (pattern: `screen.name`)
- `cellVersion(x)` — increasing version number
- `cellAge(x)` — age of the cell
- `cellSelect(x1, x2, ...)` — value of the most recently updated input cell (can mix cell types; also works for aliases)
- `cellSelectNotNull(x1, x2, ...)` — like cellSelect but ignores null inputs
- `cellSelectNotUnset(x1, x2, ...)` — like cellSelect but ignores unset inputs
- `freeze(c)` — freeze a cell: stops responding to updates and propagating changes; irreversible (no unfreeze)
- `isCellAlive(x)` (since PR#1072) — true if cell behind link still exists (useful in circuitarray actions)
- `cellMutate(appstate, link(x), y)` — assign var cell x=y without "outside action" warning
- `thisCell` (since PR#599) — the cell currently being defined
- `onRefresh` — recompute this cell when `viewstack.refresh` is called
- `onView` — recompute this cell when view is shown again (e.g. after pop)
- `newConstCell(expr)` — create a new constant cell; returns a link
- `newVarCell(expr)` — create a new var cell; returns a link
- `never` — predefined cell that is never updated, always returns `null`

---

## Definitions

```
def x = <expr>
def x : <type>                      // declaration
def x : <type> = <expr>             // merged (does not check generality)
```

**Functional forms**:

```
def f = (x -> <expr>)               // one param
def f = ((x1,x2) -> <expr>)         // two params
def f(x) = <expr>                   // shorthand
def f(x1,x2) = <expr>
def f() = <expr>                    // dummy param
def f(x1,x2) { <statements> }      // statement body
def f(x:number, msg:string):string = msg + x   // param+return types (since PR#1885)
```

**Recursive definitions** — must have a declaration before the definition:

```
def fib : number -> number
def fib(n) = if (n <= 2) 1 else fib(n-1) + fib(n-2)
```

**Foreign function definitions**:

```
foreign def f : <type> = "<name>"   // implemented in host environment (Amp/groovebox/Rust)

// With Mix fallbacks for environments where foreign impl is unavailable:
foreign def f : number -> number = "f"
  ( builderEnvironment -> f_fallback
  | widgetEnvironment -> (x -> x - 1) )
```

Host environment cases: `builderEnvironment`, `serverEnvironment`, `webEnvironment`, `widgetEnvironment`

**Side-effect-only definition**:

```
def _ = <expr>    // run expr for side effects, drop result
// e.g.: def _ = reflect.enableTrace()
// e.g.: def _ = circuitarray.setMaxAge(n)
```

---

## Statements

### Where statements occur

- Inside `def` or `cell` bodies delimited with `{ }` (no `=`)
- Inside `do { }` blocks within expressions

### `let` and `var` bindings

```
let x1 = <expr>; ...           // immutable
var x2 = <expr>; ...           // assignable
let x1 : <type> = <expr>; ...  // with type hint
let f(p1,p2) = <expr>; ...     // local function shorthand
```

Prefer `let` — type variables are generalized, enabling more powerful type checking.

### `let alias` (since PR#599)

```
let alias x1 = c1;   // c1 must be a cell; x1 aliases it (enables cellVersion, cellName, x1')
<expr>
```

### Control flow

```
if (<condition>) { ... } else { ... }       // else optional
if (<cond>) { ... } else if (<cond>) { ... } else { ... }

switch (<expr>) {
  case self.red: s = "red"
  case self.green: s = "green"
}

while (<condition>) { ... }

for (var i = 0; i < 10; i = i + 1) { ... }   // increment; let also permitted (immutable in body)
for (let x in <stream-expr>) { ... }           // stream iteration; var also permitted
```

Common stream iterators:

- `array.indexRange(a)` / `array.valueRange(a)`
- `map.indexRange(m)` / `map.valueRange(m)`
- `db.head() |> ... |> db.toStream`

### Assignments

```
varname = <expr>               // local var, var cell, or link to var cell
v[k] = <expr>                  // update array index k
v[#2 of 4] = <expr>            // update tuple index (since PR#892)
v[#label] = <expr>             // update labeled tuple position (since PR#892)
v.(k) = <expr>                 // update map key k
v.name = <expr>
v.style.width = 45             // multi-level
```

Values are immutable — structured assignment creates a modified copy.

### Return

```
return             // null
return <expr>
```

---

## Actions

```
action () { <statements> }
action (v) { <statements> }
action { <statements> }        // also valid (no variable)
```

Actions have access to `appstate` (auto-created variable, type `appstate`). Also `_` works as an alias (but prefer `appstate` for clarity). Used for: DB writes, var cell mutations, and side effects:

```
def myAction = action { db.write(appstate, null, {entity:"person", name:"Vijay"}) }

// Pass appstate to helper functions:
def setText : appstate -> string -> null
def setText(appstate, s) { text = s }

cell myDom = { "_rmx_type": "dom_text", "ontap": action(s) { setText(appstate, s) } }
```

---

*Expressions, patterns, types, cases → [mix-syntax-expressions.md](./mix-syntax-expressions.md)*