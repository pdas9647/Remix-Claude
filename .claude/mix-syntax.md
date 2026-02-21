# Mix Language Syntax Reference

> Source: [Mix Syntax](https://www.notion.so/1061d464528f81d492f4e1b0f8c8adf4)
> Parent: [The Mix Programming Language](https://www.notion.so/259ede6504e34505982dde5dc4b63d10)

---

## Directives & Typing Modes

> Source: [directives](https://www.notion.so/1061d464528f81c08c15e5ae817171ee)

### Lexing directives
```
#line <line> "<filename>"     // set line number + filename for next line (filename optional)
#push <line> "<filename>"     // like #line but scoped; undo with #pop (since PR 547)
#pop                          // restore previous line/filename state (since PR 547)
```
Must be at leftmost column. `#push`/`#pop` can be nested.

### Source redirect (since PR 557)
```
module m
_source_ { url:"https://wherever" }
```
Redirects amp to fetch source from a remote URL (HTTP GET → 200). No other definitions or directives allowed; no module params. Must be publicly accessible. Cannot nest redirects.

### Double semicolon `;;`
Forces end of current `def` or `cell` phrase. Useful for separating known-good and questionable code sections so error line numbers are accurate:
```
def x = 10 ;;
[0] ;;        // error here points here, not to x above
def y = 20 ;;
```

### Pragmas (`_pragma_`)
Placed between `def`s and `cell`s; effective until end of module or next pragma:
```
_pragma_ { name: value, name: value, ... }   // values are any JSON
```

**Warning control** (since PR#1424):
- `warningFilter:"<spec>"` — comma-separated warning names with optional `+`/`-` prefix. `"-deprecated"` disables deprecation warnings; `"+deprecated"` re-enables. See `mix_client -warn-print` for full list.

**Typing pragmas** (since PR#534) — `auto*` are ON by default; `strict*` are OFF by default:
- `autoTypeAssertion:false` — disable auto runtime type assertions for downcasts
- `autoMakeStream:false` — disable auto conversion of values to singleton streams (e.g. `1 ++ 2`)
- `autoMakeLink:false` — disable auto conversion of cell names to links where needed
- `strictContainerTyping:true` — container literals (tuples, maps) get concrete types instead of `data`
- `strictTyping:true` — enables all of the above plus stricter typing rules (since PR#892):
  - Default (non-strict): inserts runtime assertions, falls back to `data`, compound literals typed as `data`
  - Strict: no auto-conversions, types kept minimal, compound literals get concrete types (e.g. tuple literals → tuple types)

**Spreadsheet pragmas**:
- `withCells:false` — forbids adding cells to the module (lighter translation)
- `withCells:true` — enforces spreadsheet machinery availability (for stdlib use only)

---

## Meta Data

Modules, functions, and cells can carry a `_meta_` annotation (always a constant JSON object):
```
module M _meta_ { ... }
def f : t _meta_ { ... }
cell c : t _meta_ { ... }
```

Common meta forms:
- **Deprecation**: `_meta_ { deprecated:true, expires:"yyyy-mm-dd", alternatives:["alt1"] }`
- **Renaming**: `_meta_ { renamed:true, expires:"yyyy-mm-dd", replacement:"newname" }`
- **Module type**: `_meta_ { modType:"screen" }` — values: `screen`, `agent`, `component`

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

## Expressions

### Order of evaluation
Eager: left-to-right, top-to-bottom. Params computed before function body. `let x = e1; e2` → `e1` then `e2`.

### `if` in expressions
```
if (<expr>) <expr> else <expr>    // else is mandatory in expressions
// Example:
def max(x,y) = if (x > y) x else y
```

### `do` blocks
```
let x = do {
  var s = 0;
  for (let k in array.valueRange(y)) { s = s + k };
  return s
}
```

### Undefined
- Some accesses (`m.field`) may return `undefined`
- Bubbles up through many operators
- Cannot be put into arrays/maps
- `x?` — true when x is defined
- `x === y` — equality covering definedness (true if both defined+equal, or both undefined)

### Literals
```
null, true, false, 10, 10.5, 1E3, "string", """raw string no \"""

// Arrays / tuples (same representation, differ by typing context):
[], [1,2,3], [1,2,3]:array
[1,2,3][#0 of 3]        // tuple index (PR#892)
[1,2,3]:array[0]        // array index

// Labeled tuples (require type def):
type person = [#name: string, #age: number]
[#name:"Tom", #age:28]:person
[#name:"Tom", #age:28]:person[#name]

// Maps:
{}, {a:1, b:2}, {a:1, b:2}.a, {"a b c":1}, {(x):1}, {a}   // {a} = {a:a}

// Streams:
[1,2,3][]          // array to stream
[1,2,3][0..1]      // stream slice (indices 0–1 inclusive)
```

### Functional updates (no `update` keyword since PR#959):
```
a with [2] = "foo"
m with .num = 66
m with .(k) = 66
m with .idx[2] = true
m with .x = "a" with .y = "b"    // chainable
```

### Lambda functions
```
(x -> <expr>)
((x1,x2) -> <expr>)

// Multi-arm lambda (with cases):
(self.red -> "red" | self.green -> "green" | self.blue -> "blue")
```

### Function calls
```
f(x)               // classic
x |> f             // reverse application (pipe)
f(x1,x2)
x2 |> f(x1)        // mixed
r.f(x)             // function in record element (since PR#1295)
(y -> ...)(x)      // immediately-invoked lambda (since PR#1295)
```
**Automatic currying**: fewer params than declared → returns function for remaining params.

### Catching runtime errors
```
try f(arg)         // catches runtime error; returns result type (ok(...) or error(...))
// Alternatively use: co.apply()
```

### Operators
```
// Definedness:
x?               // true if defined
a[k]?            // true if index k exists
m.(k)?           // true if key k exists

// Logic: x && y, x || y, not x

// Arithmetic: +, -, *, /, %   (+ also for string+*, array+array, map+map)
// floor(x), ceil(x)

// Comparisons (return undefined if either operand is undefined):
==, !=, <, <=, >, >=

// Equality including definedness:
===, !==     (undefined === undefined is true)

// Case-insensitive string comparisons (since PR#1821):
==:ci, !=:ci, <:ci, <=:ci, >:ci, >=:ci

// Stream append: x ++ y
// Pipe: x |> f
```

### JSON constants with escaping (since PR#1121)
```
def c0 = { "name": "Gerd", "age": 50 }   // plain JSON
def c1 = _escJSON_ { _rmx_type: "{tag}case:ok", _rmx_value: "hello" }  // escaped JSON
// equivalent to: def c1 = ok("hello")
```

---

## Patterns

Used in `let` destructuring, lambda multi-arm (`|`), and `switch case`.

Must be **exhaustive** — add `case _` as catch-all. Non-strict mode may still yield runtime errors.

### Matching constants
`undefined`, `null`, `true`, `false`, numbers, strings

### Matching tuples / arrays
```
let [x,y] = pair               // destructure 2-tuple; runtime error if wrong type
def f = ( [true,0] -> "c1" | [true,_] -> "c2" | [false,_] -> "c3" )
def g(a) = a |> ( [] -> "empty" | [x] -> x | _ -> "many" )
```
Array modifiers:
```
def insideParens =
  ( ["(", ~multiple x, ")"] -> x   // x is an array of middle elements
  | _ -> fail("not matched")
  )

def parseRecord =
  ( [surname, ~optional middlename, ~optional prename] ->
        [surname, option.create(middlename), option.create(prename)]
  | _ -> fail("not matched")
  )
```

### Matching records/maps
```
def f =
  ( {_rmx_type:"dom_group", children:children} -> children
  | _ -> []
  )
```

### Matching cases
Use `self.` prefix (or module prefix) to distinguish case match from variable binding:
```
case transparent
case rgb([number,number,number])
type color = transparent | rgb

def colorName =
  ( self.transparent -> "#00000000"
  | self.color([r,g,b]) -> "#ff" + number.toHex(r) + number.toHex(g) + number.toHex(b)
  )
```
Use `global.` for predefined cases: `x |> (global.some(v) -> ... | null -> ...)`

### Matching data cases (runtime type dispatch)
```
def typeName =
  ( type(string) -> "string"
  | type(number) -> "number"
  | type(bool) -> "bool"
  | _ -> "other"
  )
```

### Capturing values
```
def toString =
  ( s @ type(string) -> s
  | n @ type(number) -> "" + n
  | _ -> "other"
  )
```

### Comparisons, AND/OR, guards
```
def assess = ( n < 5 -> "less than 5" | n == 5 -> "exactly 5" | _ -> "more" )
def inRange = ( n >= 3 && n < 7 -> "in range" | _ -> "out" )

// Guards (when):
def length =
  ( [] -> "empty"
  | a @ type(array) when array.length(a) > 10 -> "many"
  | a @ type(array) -> "some"
  )
```

### Match against computed value
```
def limit = 10
def inLimit = ( x < limit -> "below" | _ -> "beyond" )
// Computed values only allowed on right-hand side of comparison; must be simple vars computed outside the pattern
```

---

## Types

```
// Simple:
null, bool, number, string, word   // word = number | string
data                               // any serializable value

// Functions:
number -> string
number -> bool -> string

// Streams:
stream(string)
string*         // DEPRECATED

// Containers:
[bool, number]                         // tuple (since PR#892)
array(number)                          // array (since PR#892)
[#name: string, #age: number]          // labeled tuple (since PR#892)
{x:number, y:string}                   // record
map(word)                              // map
```

See [Type system](https://www.notion.so/259ede6504e34505982dde5dc4b63d10) for full details.

---

## Cases

```
case red                    // constant case (no params)
case euro(number)           // case with param

type color = red | green | blue     // union type
type currency = euro | dollar

def my_wallet : currency = euro(10.5)

// Inverse constructor (extract param):
def n = dollar^(dollar(10))   // returns 10

// Pattern matching cases:
def print_my_wallet() =
  my_wallet |>
    ( self.euro(value) -> debug("euro " + value)
    | self.dollar(value) -> debug("dollar " + value)
    )
```

---

## Other Idioms

### Short projections
```
array.map(.field)            // same as: array.map(r -> r.field)
map.map({ .a, b:10 })        // same as: map.map(r -> {a: r.a, b: 10})
```

---

## Code Style Guide

> Source: [styleguide](https://www.notion.so/1061d464528f819099d2e81ae669855d)

Rules for the indentation-based parser:

**Definitions and cells**: starting keyword (`def`, `cell`) on column 1. Content either entirely on same line or indented on following lines — never partly on the first line and partly below.
```
def x = 1                    // good: all on one line
def y =                      // good: indented on next line
  2
def f = (x ->                // BAD: content split across first + following line
  x+1)
```

**Indentation**: subexpressions spanning multiple lines must be indented; indentation must fit the bracket structure. Entire subexpression on one line → no indentation needed.

**Closing brackets**: right bracket on the same column as left bracket (or "bracket trail" — all closings on one line). Forbidden: right bracket more to the left than the left bracket.
```
cell x =                     // good: matching columns
  [ { foo: 1 },
    { foo: 2,
      bar: "x"
    }
  ]
cell y =                     // good: bracket trail
  [ { foo: 1 },
    { foo: 2, bar: "x" } ]
```

**Function calls**: all on one line, or name on line 1 and args on line 2 indented, or args across lines aligned on the same column.
```
f(1, 2, 3)                   // good
f                            // good
  (1, 2, 3)
f(1,                         // good: args aligned
  2,
  3)
longname(1,                  // BAD: args not aligned
  2, 3)
```

**Long arithmetic**: continuation line starts with the operator symbol.
```
"hello " + "world"           // good: + at start of continuation
  + " again"
"hello " + "world" +         // BAD: operator at end of continued line
  " again"
```

**Commas**: all elements on one line, or every element on a new line with trailing comma (except last).

**`let` sequences**: each `let` on a new line, aligned; `;` at line end.
```
def f =
  (x ->
    let a = x+2;
    let b = x*x*x;
    a+b )
```

---

## Action Closures

> Source: [actionclosures](https://www.notion.so/1061d464528f818cb833fd55e9077e77)

An action closure wraps an action for serialization into a view as data. The runtime invokes it when triggered.

### Serialized format
```
action { c = 42 }    // serializes to:
// { "_rmx_type": "msg_view_invoke", "name": "$closure_XXXX_0", "env": "$cloenv_1" }
```
- `_rmx_type`: identifies it as an invokable action
- `name`: identifies the closure code
- `env`: identifies captured values (closure environment)

Globals (`def`ed values) are **not** captured — the current value at invocation time is used. This is intentional for development (redefinitions take effect immediately).

### Chain of actions
Inside an action body, you can inspect and manipulate what runs next:
```
getNextActions()                       // → array of action objects to run after current
setNextActions(appstate, newChain)     // replace the next-action chain
pushNextActions(appstate, addChain)    // append to the next-action chain
pushIdleActions(appstate, addChain)    // runs after all requests done, even if client disconnected
                                       // NOTE: idle actions only implemented for non-persistent agents
```
Any object with `_rmx_type` other than `msg_view_invoke` is a **client action** — forwarded to client for execution.

### Closures as agent entry points (since PR#307)
A cell set to an action closure inside a module gets a predictable name:
```
module M
  cell myAction = action(v) { ... }  // name: "M.myAction"
module end
// Invokable with: #invoke M.myAction "$agent_environment" <value>
```
Constraints:
- Module may have parameters (uses params from last push)
- Cell value must be exactly the closure — no wrappers, no conditionals
- Only closures in the **last pushed module** can be invoked; not in imported sub-modules

---

## Statements vs Expressions — Key Rules

> Source: [syntax-stmt-expr](https://www.notion.so/1061d464528f8150a392ddc30fbdbb9a)

**Expressions** are result-oriented (composed by combining sub-expressions into a result). **Statements** are effect-oriented (execute in sequence, mutate state).

### Not allowed inside expressions
- `return` — no jumping out; result is the last computed value
- `for` / `while` — use functional tools (map/reduce/recursion) instead
- `if` without `else` — expression must always have a result
- `if` with curly braces — curlies indicate statement mode
- `switch` — use pattern-matching lambda instead: `x |> (pat1 -> e1 | pat2 -> e2 | ...)`
- assignment — expressions should not have effects

`fail("message")` can be used to force a runtime error inside an expression.

### Statements syntax
```
{
  var x = 0;
  x = x + 1;
  return x       // delivers result from statement block
}
```
Statement group: **curly braces**, separated by **semicolons**. Missing `return` → implicit `null`.

Allowed in statements: `let`, `var`, assignment, `return`, `if { }`, `while`, `for`, `switch`, expressions (result dropped).

### `do` blocks — statements inside expressions
```
let x = do {
  var s = 0;
  for (let k in array.valueRange(y)) { s = s + k };
  return s       // return exits the do block, not the function
}
```
`do` is required after `->` (arrow) since the arrow expects an expression, not a statement block.

### Cell mutation — requires `appstate`
Direct cell mutation only inside `action { }`. For helper functions:
```
def set_x(appstate:appstate) { x = 10 }   // appstate param = permission to mutate cells

cell a = action { set_x(appstate) }
```
**A function that takes `appstate` as parameter is allowed to access and mutate cells.**

---

## Undefined — Semantics

> Source: [undefined](https://www.notion.so/1061d464528f81c1bb53e27eb10a9aea)

`undefined` is distinct from `null`. It represents a "soft error" — unusual but not always fatal. In the builder, `undefined` = non-existing binding; `null` = programmatic missing value (user hasn't entered anything yet).

### When undefined occurs
- Missing field: `{field1:"hello"}.field2` → `undefined` (old Mix returned `null`)
- Out-of-range array index via `safeGet`
- Any operation where input is `undefined` (bubbles up)

### Bubbling
Operations that propagate `undefined` from inputs to output:
- Arithmetic: `+`, `-`, `*`, `/`, `%`, `floor`, `ceil`
- Comparisons: `==`, `!=`, `<`, `<=`, `>`, `>=`
- Negation: `not`
- Object field access, `array.length`, `map.keys`, etc.

**`&&` and `||` do NOT simply bubble** — they use Kleene 3-valued logic:

```
x && y:                          x || y:
      false  true  undef               false  true  undef
false false  false false         false false  true  undef
true  false  true  undef         true  true   true  true
undef false  undef undef         undef undef  true  undef
```

Key rule: `false && anything` = `false`; `true || anything` = `true` (shortcut evaluation preserved).

### Cannot store undefined
Runtime error if undefined is placed in: arrays, objects/records, cells, `var` variables, database.

Use `x?` to test before storing:
```
withDefault("", x)              // return "" if x is undefined
noDefault(x)                    // runtime error if x is undefined
x?                              // true when x is defined
x === undefined                 // equality including undefinedness
```

### Deep field access safety
```
person.team.company.name        // undefined if any link missing (not a runtime error)
```

### Undefined in filters
Undefined filter condition → row **excluded** (not included).
```
array.filter(row -> row.city != "New York")   // rows without 'city' field are excluded
```
Use `null` (not `undefined`) for explicit missing values in data: `null != "New York"` is `true`.

### Other facts
- JSON serialization of `undefined`: the string `"{undefined}"`
- `null` is a subtype of `data`; `undefined` is element of **all** types
- `null` can be placed in arrays/maps; `undefined` cannot
- `undefined` can be parameter and return value of user-defined functions (bubbles in/out)

---

## Operator Semantics (Builtin Operations)

> Source: [Builtin operations](https://www.notion.so/1061d464528f81a5a5bbff09ca40f2cf)

### `+`
- One operand `null` → `null`
- Two numbers → added; both int → tries to return int, otherwise float
- Two strings → concatenated
- String + number → number converted to string, then concatenated
- Two arrays → concatenated
- Two objects/maps → fields from second override first

### `-`, `*`, `/`, `%`
- One operand `null` → `null`
- Two numbers → result; **`/` and `%` are always float** (never integer)

### `++` (stream concat)
Type: `T* ++ T* -> T*`. Type-checker auto-converts `T` to `T*`, so non-streams work:
```
1 ++ 2 ++ 3              // stream of [1, 2, 3]
allRows ++ "end"         // stream elements from allRows, then "end"
```

### `&&` and `||`
Shortcut evaluation — second operand only evaluated when needed:
- `e1 && e2` ≡ `if (e1) e2 else false`
- `e1 || e2` ≡ `if (e1) true else e2`

### `==` / `!=` (deep equality)
- Two `null` → equal
- Same scalar kind (bool/int/float/string) → equal when same value
- Integer and float → equal when float is integral and numerically equal (so `1 == 1.0`)
- Two arrays → equal when same length and all elements equal pairwise
- Two objects → equal when same key set and all values equal by key
- `1` and `"1"` → **not** equal; `1` and `1.0` → equal

### `<`, `<=`, `>`, `>=` (ordering)
- `null` is the smallest value; only equal to itself
- `false < true` for bools
- Integers and floats: compared numerically
- Strings: lexicographic on UTF-8 bytes
- Arrays: lexicographic element-by-element
- Objects: implementation-defined
- Mixed kinds (e.g. string vs number) → non-comparable

### Built-in functions
```
datacase(x)          // → "null"/"bool"/"number"/"string"/"array"/"object"/"ref"
                     // Use pattern matching instead (type(string) etc.) — more idiomatic

debug(msg)           // → null; prints msg to debug console
                     // use: cell x = let _ = debug("x=" + x); x + y

withDefault(dfl, x)  // → x if defined, dfl otherwise
noDefault(x)         // → x if defined, runtime error otherwise
not(x)               // boolean negation
```
