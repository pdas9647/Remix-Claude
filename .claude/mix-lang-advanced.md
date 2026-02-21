# Mix Language — Advanced Patterns

> Topics that supplement [mix-syntax.md](./mix-syntax.md). Covers recursion, stream programming,
> pattern-matching additions, and advanced module features.

---

## Recursion

> Source: [recursion](https://www.notion.so/1061d464528f81a793f6f06a4c38da21)

### Recursive `def` — requires forward declaration

A `def` can call itself only if a **signature declaration** appears before the definition:
```
def fib : number -> number
def fib(n) = if (n <= 2) 1 else fib(n-1) + fib(n-2)
```

### Mutual recursion — declare all signatures first

Write **all** forward declarations before the first definition body:
```
def countNumbers       : data -> data
def countNumbersInArray : data -> data
def countNumbersInMap   : data -> data

def countNumbers =
  (x ->
    let dc = datacase(x);
    if (dc == "number") 1
    else if (dc == "array") countNumbersInArray(x)
    else if (dc == "object") countNumbersInMap(x)
    else 0
  )
def countNumbersInArray =
  (x -> array.reduce((x_k,acc) -> acc + countNumbers(x_k), 0, x))
def countNumbersInMap =
  (x -> map.reduce((x_k,acc) -> acc + countNumbers(x_k), 0, x))
```

### Cells CANNOT be recursive

A recursive `cell` definition is rejected by the compiler — it would be a cycle in the dependency
graph. Use `def` with a forward declaration instead:
```
cell fib = (n -> if (n <= 2) 1 else n + fib(n-1))  // ERROR: cycle in dependency graph
```

### Local recursion with `recurse`

For local recursive functions inside a `def`, use the `recurse` combinator (no forward declaration
needed). Replace the function's own name with `selfcall`:
```
def fibonacci =
  recurse((selfcall, n) -> if (n <= 2) 1 else n + selfcall(n-1))
```

For two parameters: `recurse2((selfcall, x1, x2) -> ...)`

---

## Stream Programming

> Source: [programming-with-streams](https://www.notion.so/1061d464528f8145b58dfe16d3c998c5)

### Streams are mutable

Streams are a **mutable structure** — some operations consume (remove) elements:
- `stream.first(s)` — returns and **removes** the first element from `s`
- `stream.isEmpty(s)` — check whether more elements remain

Stream recursion must therefore use `def` with forward declarations (cells cannot be recursive):
```
def count : stream(data) -> number
def count =
  (s ->
    if (stream.isEmpty(s)) 0
    else (
      let _ = stream.first(s);   // consume and discard
      count(s)
    )
  )
// let s = [1,2,3][] in count(s)  →  3  (and s is now empty)
```

### Stream append `++` is O(1)

`s ++ t` is constant-time. Useful for building result streams recursively:
```
def expand : stream(data) -> stream(data)
def expand =
  (s ->
    if (stream.isEmpty(s)) stream.empty
    else (
      let k = stream.first(s);
      stream.enumerate(k) ++ expand(s)   // prepend 0..k-1, then recurse
    )
  )
// expand([4,2,1][])  →  stream [0,1,2,3, 0,1, 0]
```

---

## Pattern Matching — Additions

> Source: [pattern-matching](https://www.notion.so/1061d464528f8155b917f627971aafb9)
> Core patterns are documented in [mix-syntax.md § Patterns](./mix-syntax.md).

### Optional field in record destructuring

The `?` suffix marks a record field as optional. If the field is absent, the variable is bound to
`undefined` (not a runtime error):
```
let {key1:v1, key2?:v2} = <expr>
// v2 = undefined when key2 is missing from the record
```

### Current limitations (as of PR#518)
- `var` destructuring is **not supported** — only `let` allows patterns on the left-hand side
- `def f(arg) = ...` — `arg` cannot be a pattern; only plain variable names are allowed in `def` params
- Pattern alternatives are checked **linearly in order** (no jump tables)

---

## Modules — Advanced Features

> Source: [modules](https://www.notion.so/1061d464528f813c927efa13f30e9e65)
> Core module syntax is documented in [mix-syntax.md § Modules](./mix-syntax.md).

### Singleton (static) modules (since PR#963)

A singleton module has exactly one instance — useful as a shared state container. No parameters
allowed. Functions in a singleton can **read** `var cell`s directly; writes still need `appstate`:
```
static module Current
  var cell persons = []
  def getPersons() = persons
  def setPersons(appstate:appstate, p) { persons = p }
module end
// import Current  →  always returns the same instance
```
Alternatively: `_pragma_ {singleton:true}` inside the module body.

**Extending var cell lifetime** (call inside `initializer { ... }`):
```
cellSessionScope(appstate, persons)   // cell lives until session ends
cellScreenScope(appstate, persons)    // cell lives until current screen pops
```

### Module instance ID (since PR#1425)

`currentInstanceId()` returns a unique number for each instance of the same module within a view
(returns 0 when called from outside a module, ≥1 inside):
```
module M(s:string)
  def _ = debug("instance: " + currentInstanceId())
module end
def m1 = M("hi")     // different IDs
def m2 = M("hello")
```

### Modules with type variables (since PR#1826)

Declare type variables in the module head with `[A]`:
```
module M[A]
  def identity : A -> A
  def identity(x) = x
module end
```
Substitute a concrete type at import time:
```
import M[A=string]         // identity is string -> string in this scope
```
Or create a named module alias:
```
module P = M[A=string]     // P.identity : string -> string; M.identity still polymorphic
```

### First-class module typing

- A module value is **only type-compatible with itself** — two modules defining identical members are
  still incompatible types
- Module type notation requires parentheses: `(module type M)`
- `M.name` on a cell returns a **link**, not the value; use `alias` for spreadsheet access:
  ```
  alias zzz = M.x    // zzz is spreadsheet-connected to M.x (prints value, propagates changes)
  ```

### REPL commands
```
#load A              // start application backed by module A (searches A.mix)
#push B              // instantiate module B and push as new view
#push C 67 90.2      // with parameters
#pop                 // return to previous screen
#input M.p 6         // set cell M.p to constant value 6 (must be constant JSON)
#quit                // exit the REPL (also: CTRL-C or CTRL-D)
```

---

## Comments

> Source: [gettingstarted](https://www.notion.so/855fbfd7dc8140269ed3a30224acf692)
```
// line comment (to end of line)
/* block comment */
```

---

## Links — Advanced

> Source: [links](https://www.notion.so/1061d464528f8159bc12ea2ae03ecbad)
> Core link/alias syntax: [mix-syntax.md § Spreadsheet (Cells)](./mix-syntax.md)

### `link(c)` does NOT create a dependency

Using `link(c)` in a cell does not add `c` to the dependency graph. The link captures the cell's
*identity*, not its value — so changes to `c` do not re-trigger the cell holding the link.
```
cell y = contents(link(x))   // reads x but y is NOT updated when x changes
cell z = x                   // z IS updated when x changes
```

### Assigning to link variables

A `link(T)<var>` can be the left-hand side of an assignment inside an action. In `def`s, need
`appstate` param:
```
def set_link : appstate -> link(A)<var> -> A -> data
def set_link(appstate, lnk, x) { lnk = x }
```

### `inCellName(link(c))`

Returns the internal string name of the backing cell. Used to tell the UI which cell to write into
(e.g. camera capture, file upload):
```
cell view = { "_rmx_type": "msg_camera_capture", "name": inCellName(link(picture)) }
```

### `dependentLink` (experimental)

Recomputed link — recreated only when the second function's result changes:
```
dependentLink(creator_fn, dependency_fn)
// creator_fn: _ -> link       (builds the link)
// dependency_fn: _ -> value   (link recreated only when this value changes)

alias zz = dependentLink(_ -> M(z).y, _ -> floor(z % 5))
// link to M(z).y; recreated only when floor(z % 5) changes
```
Useful for controlling when module instances are rebuilt (e.g. only when array length changes, not
on every row value update).

### `mapLinkArray` (experimental)

Maps over an array of rows, applying a link-creating function; returns a link to the resulting
array of cell values:
```
alias view = mapLinkArray((k, _) -> M(rows, k).view, rows)
// view updates when rows changes, but individual links not recreated on value-only changes
// Caution: runtime error if rows array length increases — combine with dependentLink
```

### Limitation: no links between views

Links cannot cross view boundaries. A newly pushed view starts a fresh spreadsheet — you cannot
pass a `link(T)` to it as a push parameter.

---

## JSON / EscJSON Representation

> Source: [json-changes](https://www.notion.so/1061d464528f817aacb5db1a7d575617)

Mix types richer than JSON (case types, db refs, calendar dates, color values) are serialized with
**escape tags**. Non-serializable types: streams, closures, tokens.

### Escape tag format
```
{ "_rmx_type": "{tag}case:ok", "_rmx_value": "hello" }  // ok("hello")
{ "_rmx_type": "{tag}ref",     "_rmx_value": "123" }     // db reference
```
To store the literal string `{tag}...` in `_rmx_type`, prefix with `{str}` (double-escape):
```
{ "_rmx_type": "{str}{tag}case:ok" }   // plain string, not a case value
```
`{str}` only appears in `_rmx_type` fields.

### `string` module JSON functions
```
string.formatJSON(x)           // serialize; fails if x contains non-JSON-native values
string.formatEscJSON(x)        // serialize with escape tags
string.parseJSON(s)            // parse; does NOT interpret escape tags
string.parseJSONResult(s)      // same, returns result(string, data)
string.parseEscJSON(s)         // parse AND interpret escape tags
string.parseEscJSONResult(s)   // same, returns result(string, data)
```
Think twice before enabling EscJSON on untrusted input — escape tags can affect db query behavior.

### Full tag type reference

> Source: [escjson](https://www.notion.so/1061d464528f8110aae4ef61230e4783) (canonical reference)

| Tag | Meaning | Notes |
|-----|---------|-------|
| `{tag}case:red` | parameterless case `red` | no `_rmx_value` |
| `{tag}case:meters` + `_rmx_value` | case with arg `meters(v)` | |
| `{tag}case:db.ref` + `_rmx_value` | database reference | canonical form |
| `{tag}bin` + `_rmx_value` (base64) | binary/blob data | |
| `{tag}blob:image/png` + nested `{tag}bin` | blob generator | expanded to saveable db blob record |
| `{tag}undefined` | `undefined` value | |
| `{tag}nan` / `{tag}posinf` / `{tag}neginf` | special float values | |
| `{tag}int` + `_rmx_value` | large integer (bigint) | input-only; output uses JSON number |
| `{tag}float` + `_rmx_value` (hex FP) | precise float | input-only; output uses JSON number |
| `{tag}str` / `{tag}null` / `{tag}bool` | string / null / bool | input-only; output uses native JSON |

Note: the json-changes page uses `{tag}ref`; the escjson reference page uses `{tag}case:db.ref`. Treat `{tag}case:db.ref` as canonical.

### DB refs

Old string form `"{ref}123"` is deprecated. Current canonical form:
```
{ "_rmx_type": "{tag}case:db.ref", "_rmx_value": "abcd" }
```
- `db.makeRef("{ref}123")` — convert old-style string to a db ref
- `db.isRef(x)` — test whether x is a db ref
- Db refs behave like strings in string operations (e.g. `string.length` works)

### `_escJSON_` notation for Mix constants
```
def const = _escJSON_ { _rmx_id: { "_rmx_type": "{tag}ref", "_rmx_value": "123" }, name: "foo" }
```
- Value after `_escJSON_` must be a compile-time constant
- `{tag}` in `_rmx_type` fields is interpreted

### Track protocol escape tag behavior
- **Requests**: escape tags OFF by default; enable per parameter:
  - `msg_view_invoke`: `argEscEnabled: true`
  - `msg_view_load` / `msg_view_push`: `argsEscEnabled: true`
  - `msg_view_input`: `valueEscEnabled: true`
  - Exception: `nextActions` in `msg_view_invoke` always has escape tags enabled
- **Responses**: always formatted with escape tags (any Mix type can appear in output)

---

## DB Query Notes

> Source: [examples](https://www.notion.so/1061d464528f815fad1ed849ced729c1)

### `db.all()` vs `db.head()`
- `db.all()` — returns all non-user-deleted records across **all versions**
- `db.head()` — returns only the **latest version** of each non-user-deleted record; only available when the query engine is directly connected to FigDB (i.e. server-side agents, not remote HTTP queries)

### `def` redefinition scoping
A second `def` with `=` in the same scope **replaces** the first, and **all prior references** see the new value:
```
def x : data
def y = (arg -> x)   // y closes over the scope, not the current value
def x = 1
def x = 2
y(null)              // returns 2, not 1!
```
This is intentional — global `def`s act like named cells in the dependency graph, not like let-bindings.

---

## Embedding Viewstacks

> Source: [embedding-viewstacks](https://www.notion.so/1061d464528f81a7a3a0e65bf1f509f6)

A secondary viewstack can be created, embedded visually into the main one, and controlled independently.

### Creating a secondary viewstack
```
declare myScreen(param:data)

let vs = viewstack.embed(appView myScreen({p1: ...}))          // returns embeddedView
// or by string name:
let vs = viewstack.embed(viewstack.dynamicAppView("myScreen", {p1: ...}))

viewstack.terminate(vs)    // destroy when done (frees memory)
```

### Reading output (DOM stream)
```
let msgStream = viewstack.cellMessages(vs)
// stream of {cellname: string, cellvalue: data}
// pseudo-cells: $location (navigation events), $nextActions (follow-up actions)

// filter to DOM cell + next actions (use for remote embedded apps too):
let domStream = msgStream
  |> stream.filter(msg -> msg.cellname === "$nextActions"
                          || string.endsWith(msg.cellname, ".view"))

// prepend placeholder so UI doesn't wait:
let effStream = stream.singleton({cellname:"dummy.view", cellvalue: <placeholder>}) ++ domStream
```

### Consuming stream into spreadsheet
```
live cell msgToEmbed = co.consumeWithActions(effStream)
// co.consumeWithActions can handle $nextActions; co.consume cannot

private cell domToEmbed = [ msgToEmbed.cellvalue ]
// connect domToEmbed to a container card tile
```

### Complete action template (embed/replace pattern)
```
var cell lastStack : option(embeddedView) = null
var cell effStream = stream.singleton({cellname:"dummy.view", cellvalue:null})
live cell msgToEmbed = co.consumeWithActions(effStream)

// action to embed a screen:
do {
  switch (lastStack) {
    case some(vs): viewstack.terminate(vs)
    case null:
  }
  let vs = viewstack.embed(appView myScreen(param));
  lastStack = some(vs);
  let msgStream = viewstack.cellMessages(vs);
  let domStream = msgStream
    |> stream.filter(msg -> msg.cellname === "$nextActions"
                            || string.endsWith(msg.cellname, ".view"));
  effStream = stream.singleton({cellname:"dummy.view", cellvalue:null}) ++ domStream
}
```

Input side (action closures): local embedded viewstacks handle action routing automatically. Remote apps require the Track protocol.

---

## Imperative — Effects Cannot Escape

> Source: [imperative](https://www.notion.so/1061d464528f81de9e27c7f475f0071a) (page is outdated;
> current statement syntax is in mix-syntax.md)

**Compiler constraint**: a `def` that contains a `var` mutable variable **cannot return another
function**. This prevents effectful closures from leaking local mutable state:
```
def getset =             // REJECTED by compiler
  (initval ->
    var x = initval;
    let f = (newval -> x = newval);
    f                    // cannot return f — it closes over mutable x
  )
```
Use `cell`s or `appstate`-based patterns for shared mutable state instead.
