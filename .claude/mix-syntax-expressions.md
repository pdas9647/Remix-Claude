# Mix Language — Expressions, Patterns, Types & Cases

> Source: [Mix Syntax](https://www.notion.so/1061d464528f81d492f4e1b0f8c8adf4)
> Parent: [The Mix Programming Language](https://www.notion.so/259ede6504e34505982dde5dc4b63d10)

See also: [mix-syntax-core.md](./mix-syntax-core.md) | [mix-syntax-semantics.md](./mix-syntax-semantics.md) | [mix-syntax-style.md](./mix-syntax-style.md)

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

### Functional updates (no `update` keyword since PR#959)

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