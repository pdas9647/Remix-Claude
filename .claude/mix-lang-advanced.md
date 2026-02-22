# Mix Language — Advanced Patterns

> Core language patterns: recursion, stream programming, pattern-matching additions, and advanced module features.
> See also [mix-lang-advanced-runtime.md](./mix-lang-advanced-runtime.md) for links, EscJSON, DB queries, embedding viewstacks.

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
> Core patterns are documented in [mix-syntax-expressions.md § Patterns](./mix-syntax-expressions.md).

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
> Core module syntax is documented in [mix-syntax-core.md § Modules](./mix-syntax-core.md).

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

## Imperative — Effects Cannot Escape

> Source: [imperative](https://www.notion.so/1061d464528f81de9e27c7f475f0071a) (page is outdated;
> current statement syntax is in mix-syntax-core.md)

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