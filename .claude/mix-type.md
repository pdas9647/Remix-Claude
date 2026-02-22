# Mix Type System

> Source: [Type system](https://www.notion.so/1061d464528f81c0ba7bdc8fc2ce8556)
> Parent: [The Mix Programming Language](https://www.notion.so/259ede6504e34505982dde5dc4b63d10)

Most of this applies to **strict typing mode** (`_pragma_ {strictTyping:true}`). See [mix-syntax-style.md](./mix-syntax-style.md) for directive usage.

---

## Type Hierarchy Overview

| Type        | Values                                                                       | Notes                                                                            |
|-------------|------------------------------------------------------------------------------|----------------------------------------------------------------------------------|
| `null`      | `null`                                                                       | Bottom type for data                                                             |
| `bool`      | `true`, `false`                                                              |                                                                                  |
| `number`    | integers, floats                                                             |                                                                                  |
| `string`    | string literals                                                              |                                                                                  |
| `word`      | all `number` ∪ `string`                                                      |                                                                                  |
| `data`      | `null`, `bool`, `number`, `string`, and any tuple/array/record/map of `data` | Serializable values; cast from `data` to smaller type generates compiler warning |
| `any`       | every value                                                                  | Top type; do not use in your own code                                            |
| `undefined` | —                                                                            | Element of **all** types                                                         |

### Type accessors by kind

| Kind          | Value literal                    | Type notation                 | Access                                        |
|---------------|----------------------------------|-------------------------------|-----------------------------------------------|
| tuple         | `[1, "hello"]`                   | `[number, string]`            | pattern `let [a,b] = t` or `t[#0 of 2]`       |
| labeled tuple | `[#name:"Tom", #age:28]:person`  | `[#name:string, #age:number]` | `p.[#name]`                                   |
| array         | `[1,2,3]:array`                  | `array(number)`               | `a[i]` (computed index); `a[i]?` (membership) |
| record        | `{f1:10, f2:true}`               | `{f1:number, f2:bool}`        | `r.field` or `r."field"` (no `r.(expr)`)      |
| map           | any record upcast: `{f1:10}:map` | `map(number)`                 | `m.(key)`; `m.(key)?` (membership)            |

In **strict mode**, bracket literals `[...]` default to **tuples**; reinterpreted as arrays when the context demands (e.g. `array.length([1,2])`).

---

## Type Variables

| Notation      | Meaning                                                                      |
|---------------|------------------------------------------------------------------------------|
| `T`           | Unconstrained — starts with uppercase; replaced by any type on instantiation |
| `_`           | Anonymous — each `_` is a fresh unconstrained variable                       |
| `T@datarange` | Constrained to `data`                                                        |
| `T@wordrange` | Constrained to `word`                                                        |

(Other constrained forms are used internally by the compiler and not exposed to users.)

---

## Type Definitions (Aliases)

Type definitions are **purely notational abbreviations** — not new nominal types. `myrecord` and `{name:string, age:number}` are interchangeable.

```
type myrecord = { name:string, age:number }

// With parameters (must start with uppercase):
type myrecord(G) = { name:string, age:number, group:G }
type employees = { employeeID:string }
// Usage: myrecord(employees), myrecord(customers)
```

---

## Case Types and Union Types

> Source: [case-types](https://www.notion.so/1061d464528f81dcaf60cf82adc1d1e9)

```
case green                          // constant case (no args)
case grey(number)                   // case with arg
type color = red | green | blue     // union type
```

`null` is also a case type. See [mix-syntax-expressions.md § Cases](./mix-syntax-expressions.md) for matching and inverse constructor (`^`) syntax.

### Private cases

`private case rgb(data)` — the constructor cannot be called from outside the module; the module controls all possible representations. The **deconstructor** (`rgb^`) and **pattern matching** remain
accessible from outside:

```
module color
  private case rgb(data)
  type t = rgb(data)    // "type t = rgb" is a permitted abbreviation

  def create : number -> number -> number -> t
  def create = ((r,g,b) -> rgb([clamp(r), clamp(g), clamp(b)]))

  def red : t -> number
  def red(col) = col |> rgb^ |> (triple -> triple[0])
module end
// color.rgb^ is still callable from outside the module
```

### Data compatibility

Any **parameterless** case, or any case whose parameter is a **subtype of `data`**, is also a subtype of `data`:

```
case myTag          // subtype of data ✓
case age(number)    // subtype of data ✓ (number is data)
case fun(S -> T)    // NOT data — function param is not data
```

This means data-compatible cases can be stored in `data`-typed cells or database records. Recover the original type with a type assertion:

```
cell mycol : data = color.create(12,45,198)     // store as data
cell realcol : color.t = mycol |> type(color.t) // downcast back
```

### Union constraints

- Can unite case types and existing union types: `type bw_or_color = bw | color.t`
- Each case may appear at most once in any union
- `null` can be included: `type maybe_bw = bw | null`
- **Cannot** form unions of non-case types — `number | string` is **not** valid

### Predefined cases and unions

```
case some(T)
type option(T) = some(T) | null

case ok(S)
case error(T)
type result(S, T) = ok(S) | error(T)

case unset    // alternative to null for "not yet initialized"; distinct from any operating value
```

`unset` is useful when a `var cell` must start with a placeholder that is distinct from every valid operating value.

### Caveats

- Case name must be **lowercase** (case names are both values and types)
- `private case` and `private def` exist, but there is **no `private type`** — types are always public; only construction can be protected
- Types **cannot be recursive** yet (no linked lists or trees)
- No exhaustiveness check on `switch` or pattern matching over union types (yet)

---

## Type Hints

Hints influence **type inference only** — they generate no code. They make inferred types more specific or less specific:

```
let x : data = expression           // upcast expression to data
null : data                         // null literal as data type
[10, 20, 30] : array(_)             // array literal; element type inferred
{ a:f1, b:f2 } : map(string->number)
f : (string -> number)              // note: parentheses required for function hints

def x : data = null
cell c : data = null
```

**When to use hints:**

- `null` literal gets type `null` by default — hint needed when a larger type is required
- Type errors produce confusing messages — hints simplify them
- Type checker gets on the wrong track — hints guide it

### var cell pattern — always hint

```
var cell x = null      // BAD: type is `null`, can only ever assign null
var cell x : person = null  // GOOD: can assign any person value
```

---

## Type Assertions (`type(...)`)

Assertions generate a **runtime check** (not compile-time). Limited to `data`-range types.

```
expr |> type(T)                // assert expr has type T at runtime
type(T, expr)                  // same
type(T)                        // as a function (for use with |>)
```

Constraints:

- Only assertable: `bool`, `number`, `string`, tuples, arrays, records, maps
- **Cannot** assert: functions, streams, or any non-data type
- Type term `T` may only contain `_` (anonymous variable), not named variables
    - Allowed: `type([_,_])` (assert it's a pair)
    - Forbidden: `type([T,T])` (can't check element equality)

### Type hint vs type assertion

|                      | Type Hint         | Type Assertion     |
|----------------------|-------------------|--------------------|
| When                 | Compile-time only | Runtime check      |
| Code generated?      | No                | Yes                |
| Can hint functions?  | Yes               | No                 |
| Input type required? | Known             | Any (even untyped) |

---

## word, data, any — Upcasting & Downcasting

```
// Upcasting (to a more general type):
def x : word = 10
def y = let p : data = [15] in { p:p, q:null }
var cell c : data = null        // allows assigning any data value later

// Downcasting (type assertion — generates runtime check):
def x : word = 10
def x1 = x |> type(number)     // x1 : number
```

The type inference engine auto-generates `word` and `data` when generalizing:

- Heterogeneous tuple → array: element types unified to `word` or `data`
- Heterogeneous record → map: values unified to `data`

---

## Labeled Tuples

Require a type definition. Labels exist only at compile time (not at runtime).

```
type person = [#name: string, #age: number]

// Creation (all labels must be present; order can differ):
[#name:"Tom", #age:33]:person
[#age:33, #name:"Tom"]:person   // also valid

// Access:
let name = p.[#name]            // requires p to be known as a labeled tuple type
```

If the type is not known, add a type hint: `let p:person = ...`

---

## The `+` Operator Overloading

`+` is overloaded — return type depends on operand types:

| Operands                                | Result                                   |
|-----------------------------------------|------------------------------------------|
| `number + number`                       | `number`                                 |
| `word + word` (at least one non-number) | `string` (e.g. `"x" + 10` → `"x10"`)     |
| `array + array`                         | concatenated array                       |
| `map + map`                             | merged map (second map wins on conflict) |

For polymorphic definitions, the return type is written `A + B` (delayed resolution):

```
def incr = (n -> n+1)
// incr : A -> A + number
// incr(1) → number; incr("x") → string
```

---

## Typed Queries Pattern

```
db.head()
  |> db.filter(._rmx_type == "person")
  |> db.toArray
  |> type(array({_rmx_id:string, first_name:string, last_name:string}))
```

- `db.head()` is dynamically typed (`map(data)*`)
- `db.filter(...)` filters without narrowing the type
- `type(...)` at the end asserts and narrows the type
- A record assertion does **not** fail when there are extra fields in the record
- `type(...)` is a runtime check and cannot be database-accelerated — always place at the end

---

## Locally Polymorphic Types (since PR#1424)

Requires strict typing mode. Syntax: `~forall <vars>: <function-type>`

```
_pragma_ {strictTyping: true}

def identity(x) = x
// identity : ~forall A : A -> A

def special(x:string) = x
// special : string -> string  (no quantifier — not generalized)

identity           // quantifier stripped → A -> A
polyFunction(identity)   // quantifier retained → ~forall A : A -> A
polyFunction(identity)(10)  // → 10 : number
```

- Quantifier is automatically stripped when a named function is used or called
- `polyFunction(f)` looks up `f` but leaves the quantifier in place
