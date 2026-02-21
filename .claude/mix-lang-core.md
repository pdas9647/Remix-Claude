# Mix Standard Library — Core Data Types

> Parent: [The Mix Standard Library](https://www.notion.so/1061d464528f8010b0cfc60836c20290)
> Part of: [The Mix Programming Language](https://www.notion.so/259ede6504e34505982dde5dc4b63d10)

---

## `bool`

> Source: [bool](https://www.notion.so/1061d464528f8157ae0dfdb495fb5c94)

```
module bool
  def _not : bool -> bool
  def and : bool -> bool -> bool
  def or : bool -> bool -> bool
  def toString : bool -> string
module end
```

Implements **Kleenean logic** (3-valued), with `undefined` as the unknown element — not pure Boolean.

---

## `number`

> Source: [number](https://www.notion.so/1061d464528f81bfb165f1ef2ccf84ac)

Numbers are **IEEE-754 double-precision** floating point. (May switch to decimal floats in the future.)

```
module number
  // Arithmetic
  def add/sub/mul/div/rem/mod : number -> number -> number
  def neg : number -> number

  // Rounding
  def round : number -> number    // half away from zero
  def trunc : number -> number    // toward zero
  def floor : number -> number
  def ceil : number -> number

  // Rounded arithmetic
  def divInt : number -> number -> number   // div truncated to int
  def remInt : number -> number -> number   // rem truncated to int
  def modInt : number -> number -> number   // mod floored to int

  // Utilities
  def abs : number -> number
  def min : number -> number -> number
  def max : number -> number -> number

  // Math (standard trig/exp/log — OCaml conventions)
  def acos/acosh/asin/asinh/atan/atanh : number -> number
  def cos/cosh/sin/sinh/tan/tanh : number -> number
  def exp/expm1/log/log10/log1p/sqrt : number -> number
  def copysign : number -> number -> number   // copysign(x,y) = sgn(y)*|x|
  def hypot : number -> number -> number      // sqrt(x²+y²)
  def pow : number -> number -> number        // x^y

  // Conversions
  def toString : number -> string
  def format : data -> number -> string
  def formatHexInt : number -> number -> string   // formatHexInt(minlen, x)
  def parseInt/parseIntResult : string -> number / result(string,number)
  def parseHex/parseHexResult : string -> number / result(string,number)
  def parse/parseResult : string -> number / result(string,number)

  // Bitsets (32-bit, bits 0-31)
  def bitTest : number -> number -> bool
  def bitSet/bitClear : number -> number -> number
  def bitAnd/bitOr/bitXor : number -> number -> number
module end
```

### Key behaviors

- **`rem(x,y)`** — remainder of x/y; sign follows `x`. E.g. `rem(-12.3, 5.5) == -1.3`
- **`mod(x,y)`** — remainder where sign follows `y`. E.g. `mod(-12.3, 5.5) == 4.2`
- **`parse*` functions** — fail on bad input (use `*Result` variants for safe error handling)
- **`bitXor(bitSet, -1)`** — inverts the bitset

### `number.format` reference

`number.format(options, n)` — options object fields:

| Field       | Description                            | Default                |
|-------------|----------------------------------------|------------------------|
| `locale`    | Locale (only `en-US` supported so far) | `en-US`                |
| `format`    | Pattern string (see below)             | `"-#.###############"` |
| `decimal`   | Decimal separator string               | locale-dependent       |
| `group`     | Group separator string                 | locale-dependent       |
| `plusSign`  | Positive sign string                   | locale-dependent       |
| `minusSign` | Negative sign string                   | locale-dependent       |

**Pattern characters:**

| Char | Meaning                                                                  |
|------|--------------------------------------------------------------------------|
| `+`  | Print plusSign if non-negative                                           |
| `-`  | Print minusSign if negative                                              |
| `0`  | Required digit (padded with zero). Only directly before/after decimal.   |
| `#`  | Optional digit. Before decimal: grouping position. After: max precision. |
| `.`  | Decimal separator position                                               |
| `,`  | Group separator position                                                 |

Examples: `"-0.00"` → 2 decimal places with minus; `"#,###.0#####"` → grouped thousands, 1-6 decimals; `"+-#,###.00"` with `decimal:","`, `group:"."` → European format.

---

## `string`

> Source: [string](https://www.notion.so/1061d464528f81a1bfa2e1524e618e3a)

Strings are **immutable**. Positions are in **Unicode characters** (not UTF-16 code units — U+1F535 counts as 1 character, not 2). If any parameter is `undefined`, the result is `undefined`.

```
module string
  // Primitives
  def empty : string                       // or: ""
  def add : string -> word -> string       // or: s1 + s2
  def length : string -> number
  def fromCodePoint : number -> string
  def codePointAt : string -> number -> number

  // Substrings
  def sub : number -> number -> string -> string       // sub(k_start, k_end, s) — end exclusive
  def indexOfLeft : string -> string -> number          // returns undefined if not found
  def indexOfLeft_ci : string -> string -> number       // case-insensitive
  def indexOfRight : string -> string -> number
  def contains : string -> string -> bool
  def contains_ci : string -> string -> bool            // case-insensitive
  def hasPrefix : string -> string -> bool              // hasPrefix(prefix, s)
  def startsWith : string -> string -> bool             // startsWith(s, prefix) — args swapped
  def hasSuffix : string -> string -> bool
  def endsWith : string -> string -> bool               // args swapped
  def stripPrefix : string -> string -> string
  def stripSuffix : string -> string -> string
  def replaceAll : string -> string -> string -> string // replaceAll(old, new, s)

  // Case
  def uppercase/lowercase : string -> string
  def capitalize : string -> string                     // first char uppercase

  // Trimming (all Unicode space chars)
  def trimLeft/trimRight/trim : string -> string

  // Split/Concat
  def splitSep : string -> string -> array(string)          // splitSep(sep, s)
  def concat : string -> array(string) -> string            // concat(sep, array)
  def concatStream : string -> stream(string) -> string

  // Patterns
  def like : string -> string -> bool     // like(s, pat) — % = wildcard, %% = literal %
  def rlike : string -> string -> bool    // rlike(pat, s) — args swapped

  // JSON
  def parseJSON/parseJSONResult : string -> data / result(string, data)
  def parseEscJSON/parseEscJSONResult : string -> data / result(string, data)
  def formatJSON : data -> string
  def formatEscJSON : data -> string      // uses escape tags for non-JSON values

  // Token
  def ofToken : token -> string           // can fail if permission bitset disables conversion
module end
```

### Key behaviors

- **Comparisons**: `==`, `!=`, `<`, `<=`, `>`, `>=` work on strings. Case-insensitive: use `:ci` suffix operator, e.g. `"hello" ==:ci "HeLlO"`
- **Main string passed last** — enables chaining: `string.uppercase(s) |> string.sub(2, 5)`
- **`splitSep` edge cases**: `splitSep("", "")` → `[]`; `splitSep("", "abc")` → `["a","b","c"]`; `splitSep(",", "")` → `[""]` (not empty array); leading/trailing sep → empty string element
- **`like` patterns**: `%` = wildcard, `%%` = literal percent
- **EscJSON**: `formatEscJSON` adds escape tags for values unsupported in standard JSON; `parseEscJSON` restores them

---

## `option`

> Source: [option](https://www.notion.so/1061d464528f81c2a019eb7b87dce80a)
> **Note: docs may be out of date — check against source code**

Also: module `dataOption` — identical but limited to `option(data)`.

The `option(T)` type: `null` or `some(x)`. Both `some` and `null` are in global scope (no import needed; also available as `global.some`).

```
module option
  def create : S -> option(S)              // null if input undefined, some(x) otherwise
  def withDefault : S -> option(S) -> S
  def get : option(S) -> S                 // fails if null
  def safeGet : option(S) -> S             // undefined if null
  def exists : option(S) -> bool           // opt != null
  def map : (S -> T) -> option(S) -> option(T)       // null → null
  def map2 : (S -> T -> U) -> option(S) -> option(T) -> option(U)  // null if either null
  def andThen : (S -> option(T)) -> option(S) -> option(T)
  def toArray : option(S) -> array(S)      // 0 or 1 elements
  def toStream : option(S) -> stream(S)
  def toMap : string -> option(S) -> map(S)   // {fieldname: value} or {}
module end
```

### Pattern matching

```
// Expression
x |> ( null -> ... | some(y) -> ... )
// Statement
switch(x) ( case null: ... | case some(y): ... )
// Inverse: x |> type(some) |> some^
```

---

## `result`

> Source: [result](https://www.notion.so/1061d464528f81abada4eb1d1645c59c)

Also: module `dataResult` — identical but limited to `result(Err, data)`. Modeled on Elm's `Result` package.

The `result(Err, T)` type: `error(msg)` or `ok(x)`. Both in global scope (also `global.error`, `global.ok`).

```
module result
  def withDefault : T -> result(Err, T) -> T
  def get : result(Err, T) -> T                // fails with error if not ok
  def safeGet : result(Err, T) -> T            // undefined if error
  def exists : result(Err, T) -> bool          // res == ok
  def map : (T -> U) -> result(Err, T) -> result(Err, U)
  def map2 : (T -> U -> V) -> result(Err, T) -> result(Err, U) -> result(Err, V)  // first error wins
  def andThen : (T -> result(Err, U)) -> result(Err, T) -> result(Err, U)
  def orElse : (Err -> result(U, T)) -> result(Err, T) -> result(U, T)   // map error case
  def toOption : result(Err, T) -> option(T)   // ok→some, error→null
  def mapError : (Err -> U) -> result(Err, T) -> result(U, T)
  def toArray : result(Err, data) -> data       // 0 or 1 elements
  def toStream : result(Err, T) -> T*
module end
```

### Pattern matching

```
res |> ( error(msg) -> ... | ok(val) -> ... )
// Inverse: x |> type(ok) |> ok^
```

---

## `bitset`

> Source: [bitset](https://www.notion.so/1061d464528f814abf0de4906be6232b)

Sets of integers (including negative). Implemented as search tree of 128-bit vectors. Moderately fast/compact for general use.

```
module bitset
  type t
  def empty : t
  def isEmpty/isNotEmpty : t -> bool
  def contains : number -> t -> bool
  def add : number -> t -> t
  def remove : number -> t -> t
  def singleton : number -> t
  def ofStream : stream(number) -> t
  def ofArray : array(number) -> t
  def toStream : t -> stream(number)
  def toArray : t -> array(number)
  def length : t -> number
  def equal : t -> t -> bool
  def isSubset : t -> t -> bool
  def union/intersection/difference : t -> t -> t
module end
```

---

## `passThrough`

> Source: [passThrough](https://www.notion.so/1061d464528f81a6bbf9e720f7011c21)

Creates an anonymous cell from a computed value. Useful when you need to convert a value into a cell reference.

```
module passThrough(input:data)
  cell output = input
module end
```

Usage: `link(passThrough(r).output)` — wraps value `r` in a cell so it can be used where `link()` is required (e.g., conditional module selection).

---

## `regex`

> Source: [regex](https://www.notion.so/1061d464528f8195a302ca60a5a3ef98)

Regular expressions. API follows Go's `regexp` package conventions. Only longest matches supported.

```
module regex
  type pattern = <"regexp">

  // Compile
  def compile : string -> result(string, pattern)
  def mustCompile : string -> pattern          // runtime error on bad pattern
  def quote : string -> string                 // escape meta chars for literal matching

  // Find (longest, left-most)
  def find : pattern -> string -> string                          // matched portion, or ""
  def findIndex : pattern -> string -> array(number)              // [start, end] or [-1,-1]
  def findSubmatch : pattern -> string -> array(string)           // [full, group1, group2, ...]
  def findSubmatchIndex : pattern -> string -> array(number)      // [s,e, s1,e1, ...]
  def findAll : number -> pattern -> string -> array(string)             // num=-1 for unlimited
  def findAllIndex : number -> pattern -> string -> array(array(number))
  def findAllSubmatch : number -> pattern -> string -> array(array(string))
  def findAllSubmatchIndex : number -> pattern -> string -> array(array(number))

  // Test
  def match : pattern -> string -> bool

  // Replace
  def replaceAll : pattern -> string -> string -> string   // (pat, repl, s); $0/$n in repl

  // Split
  case delim(string)
  case text(string)
  type splitResult = delim | text
  def split : pattern -> string -> array(string)
  def boundedSplit : number -> pattern -> string -> array(string)
  def fullSplit : number -> pattern -> string -> array(splitResult)  // includes delimiters

  // Templates (since PR1542)
  def template : string -> array(splitResult)                      // << NAME >> placeholders
  def instantiate : map(string) -> array(splitResult) -> string    // fill in values

  // Lexer (since PR1817)
  type lexer(A) = [pattern, string -> A]
  def lex : array(lexer(A)) -> string -> stream(result(string, A))
module end
```

### Key behaviors

- **Compile shorthand**: `~regex "pattern"` — compiles once and caches (constant patterns only)
- **Multiline strings**: use `"""..."""` to avoid double-escaping backslashes
- **Anchors**: `^` = beginning; `$` = end. Pro tip: use `^` to match only at start of string
- **Replacement**: `$0` = full match, `$n` = n-th group, `${n}` also valid, `$$` = literal `$`
- **Templates**: `regex.template("Hello << name >>")` → `regex.instantiate({name: "World"}, t)`. Missing placeholders → empty string.
- **Lexer**: patterns should be left-anchored (`^...`) for performance. Returns `stream(result(string, A))` — `ok(token)` on match, `error(msg)` on no match.

### Regex syntax (ERE-based)

| Pattern                 | Meaning                                                           |
|-------------------------|-------------------------------------------------------------------|
| `.`                     | Any char (not newline in `(?m)` mode)                             |
| `[xyz]`/`[^xyz]`        | Character class / negated                                         |
| `[[:alpha:]]`           | ASCII class (alnum/alpha/digit/space/upper/lower/word/xdigit/...) |
| `xy`                    | Sequence                                                          |
| `x\|y`                  | Alternation (prefer x)                                            |
| `x*`/`x+`/`x?`          | Repetitions (longest match)                                       |
| `x{n,m}`/`x{n,}`/`x{n}` | Bounded repetitions                                               |
| `(re)`                  | Capturing group                                                   |
| `(?i)`                  | Case-insensitive flag (Wasm VM: beginning only)                   |
| `(?m)`                  | Multi-line: `^`/`$` match line boundaries; `.` excludes `\n`      |
| `^`/`$`                 | Start/end anchors                                                 |
