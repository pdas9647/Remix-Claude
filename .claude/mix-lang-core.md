# Mix Standard Library — Core Data Types

> Parent: [The Mix Standard Library](https://www.notion.so/1061d464528f8010b0cfc60836c20290)

---

## `bool`

> [Notion](https://www.notion.so/1061d464528f8157ae0dfdb495fb5c94)

```
module bool
  def _not/and/or : bool -> bool (-> bool)
  def toString : bool -> string
module end
```

Implements **Kleenean logic** (3-valued) — `undefined` is the unknown element, not pure Boolean.

---

## `number`

> [Notion](https://www.notion.so/1061d464528f81bfb165f1ef2ccf84ac)

IEEE-754 double-precision floats.

```
module number
  // Arithmetic: add/sub/mul/div/rem/mod/neg, abs/min/max
  // Rounding: round (half away from zero), trunc (toward zero), floor, ceil
  // Rounded: divInt/remInt/modInt (truncated/floored to int)
  // Math: acos/acosh/asin/asinh/atan/atanh/cos/cosh/sin/sinh/tan/tanh
  //       exp/expm1/log/log10/log1p/sqrt, copysign/hypot/pow

  // Conversions
  def toString : number -> string
  def format : data -> number -> string
  def formatHexInt : number -> number -> string
  def parseInt/parseIntResult : string -> number / result(string,number)
  def parseHex/parseHexResult : string -> number / result(string,number)
  def parse/parseResult : string -> number / result(string,number)

  // Bitsets (32-bit, bits 0-31)
  def bitTest/bitSet/bitClear : number -> number -> bool/number
  def bitAnd/bitOr/bitXor : number -> number -> number
module end
```

**Key:** `rem(x,y)` — sign follows `x`; `mod(x,y)` — sign follows `y`. `parse*` fail on bad input (use `*Result` variants). `bitXor(bitSet, -1)` inverts.

### `number.format`

`number.format(options, n)` — options: `locale` (only `en-US`), `format` (pattern), `decimal`, `group`, `plusSign`, `minusSign`.

Pattern chars: `+` (plus sign if non-neg), `-` (minus if neg), `0` (required digit, padded), `#` (optional digit/grouping), `.` (decimal), `,` (group separator).

Examples: `"-0.00"` → 2 decimals; `"#,###.0#####"` → grouped thousands, 1-6 decimals; `"+-#,###.00"` with `decimal:","`, `group:"."` → European format.

---

## `string`

> [Notion](https://www.notion.so/1061d464528f81a1bfa2e1524e618e3a)

Immutable. Positions in **Unicode characters** (not UTF-16 code units — U+1F535 = 1 char). Undefined param → undefined result.

```
module string
  def empty/add/length/fromCodePoint/codePointAt

  // Substrings
  def sub : number -> number -> string -> string       // sub(start, end_excl, s)
  def indexOfLeft/indexOfRight : string -> string -> number  // undefined if not found
  def indexOfLeft_ci : string -> string -> number       // case-insensitive
  def contains/contains_ci : string -> string -> bool
  def hasPrefix/hasSuffix : string -> string -> bool    // hasPrefix(prefix, s)
  def startsWith/endsWith : string -> string -> bool    // startsWith(s, prefix) — args swapped
  def stripPrefix/stripSuffix : string -> string -> string
  def replaceAll : string -> string -> string -> string // replaceAll(old, new, s)

  // Case & trim
  def uppercase/lowercase/capitalize : string -> string
  def trimLeft/trimRight/trim : string -> string

  // Split/Concat
  def splitSep : string -> string -> array(string)     // splitSep(sep, s)
  def concat : string -> array(string) -> string       // concat(sep, array)

  // Patterns
  def like : string -> string -> bool    // % = wildcard, %% = literal %
  def rlike : string -> string -> bool   // rlike(pat, s) — args swapped

  // JSON
  def parseJSON/parseJSONResult : string -> data / result(string, data)
  def parseEscJSON/parseEscJSONResult : string -> data / result(string, data)
  def formatJSON/formatEscJSON : data -> string

  def ofToken : token -> string          // can fail if permission bitset disables
module end
```

**Key:** Comparisons (`==`, `<`, etc.) work on strings. Case-insensitive: `:ci` suffix, e.g. `"hello" ==:ci "HeLlO"`. Main string passed last for chaining. `splitSep("", "abc")` → `["a","b","c"]`; `splitSep(",", "")` → `[""]`. EscJSON adds escape tags for non-JSON values.

---

## `option`

> [Notion](https://www.notion.so/1061d464528f81c2a019eb7b87dce80a) — docs may be out of date

Also: `dataOption` (limited to `option(data)`). Type: `null` or `some(x)`. Both in global scope.

```
module option
  def create : S -> option(S)              // null if undefined, some(x) otherwise
  def withDefault : S -> option(S) -> S
  def get/safeGet : option(S) -> S         // get fails if null; safeGet returns undefined
  def exists : option(S) -> bool
  def map/map2/andThen                     // null-safe transforms
  def toArray/toStream/toMap               // 0-or-1 element conversions
module end
```

Pattern matching: `x |> ( null -> ... | some(y) -> ... )`. Inverse: `x |> type(some) |> some^`.

---

## `result`

> [Notion](https://www.notion.so/1061d464528f81abada4eb1d1645c59c)

Also: `dataResult` (limited to `result(Err, data)`). Modeled on Elm's Result. Type: `error(msg)` or `ok(x)`. Both in global scope.

```
module result
  def withDefault : T -> result(Err, T) -> T
  def get/safeGet : result(Err, T) -> T    // get fails if error; safeGet returns undefined
  def exists : result(Err, T) -> bool
  def map/map2/andThen/orElse/mapError     // transforms; map2: first error wins
  def toOption/toArray/toStream            // conversions
module end
```

Pattern matching: `res |> ( error(msg) -> ... | ok(val) -> ... )`. Inverse: `x |> type(ok) |> ok^`.

---

## `bitset`

> [Notion](https://www.notion.so/1061d464528f814abf0de4906be6232b)

Sets of integers (including negative). Search tree of 128-bit vectors.

```
module bitset
  type t
  def empty/isEmpty/isNotEmpty/contains/add/remove/singleton/length/equal
  def ofStream/ofArray/toStream/toArray
  def isSubset/union/intersection/difference : t -> t -> t/bool
module end
```

---

## `passThrough`

> [Notion](https://www.notion.so/1061d464528f81a6bbf9e720f7011c21)

Creates anonymous cell from computed value: `link(passThrough(r).output)` — wraps value in a cell for `link()` usage.

---

## `regex`

> [Notion](https://www.notion.so/1061d464528f8195a302ca60a5a3ef98)

Follows Go's `regexp` conventions. Only longest matches.

```
module regex
  type pattern = <"regexp">

  // Compile
  def compile : string -> result(string, pattern)
  def mustCompile : string -> pattern
  def quote : string -> string                     // escape meta chars

  // Find (longest, left-most)
  def find/findIndex/findSubmatch/findSubmatchIndex : pattern -> string -> ...
  def findAll/findAllIndex/findAllSubmatch/findAllSubmatchIndex : number -> pattern -> string -> ...

  // Test & Replace
  def match : pattern -> string -> bool
  def replaceAll : pattern -> string -> string -> string   // $0/$n in replacement

  // Split
  case delim(string) | text(string)
  def split/boundedSplit : (number ->) pattern -> string -> array(string)
  def fullSplit : number -> pattern -> string -> array(splitResult)  // includes delimiters

  // Templates
  def template : string -> array(splitResult)              // << NAME >> placeholders
  def instantiate : map(string) -> array(splitResult) -> string

  // Lexer
  type lexer(A) = [pattern, string -> A]
  def lex : array(lexer(A)) -> string -> stream(result(string, A))
module end
```

**Key:** `~regex "pattern"` compiles once and caches. Use `"""..."""` to avoid double-escaping. `$0` = full match, `$n` = nth group, `$$` = literal `$`. Templates: `regex.template("Hello << name >>")` → `regex.instantiate({name: "World"}, t)`. Lexer: left-anchor patterns (`^...`) for performance.

**Syntax (ERE-based):** `.` (any char), `[xyz]`/`[^xyz]` (char class), `[[:alpha:]]` (ASCII class), `x|y` (alternation), `x*`/`x+`/`x?`/`x{n,m}` (repetitions, longest), `(re)` (capturing group), `(?i)` (case-insensitive), `(?m)` (multiline), `^`/`$` (anchors).
