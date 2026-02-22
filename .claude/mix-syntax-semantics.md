# Mix Language — Semantics: Undefined, Operators, Stmt/Expr & Action Closures

> Sources:
> - [actionclosures](https://www.notion.so/1061d464528f818cb833fd55e9077e77)
> - [syntax-stmt-expr](https://www.notion.so/1061d464528f8150a392ddc30fbdbb9a)
> - [undefined](https://www.notion.so/1061d464528f81c1bb53e27eb10a9aea)
> - [Builtin operations](https://www.notion.so/1061d464528f81a5a5bbff09ca40f2cf)
>
> Parent: [Mix Syntax](https://www.notion.so/1061d464528f81d492f4e1b0f8c8adf4) → [The Mix Programming Language](https://www.notion.so/259ede6504e34505982dde5dc4b63d10)

See also: [mix-syntax-core.md](./mix-syntax-core.md) | [mix-syntax-style.md](./mix-syntax-style.md)

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