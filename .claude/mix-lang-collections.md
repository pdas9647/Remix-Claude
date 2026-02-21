# Mix Standard Library — Collection Types

> Parent: [The Mix Standard Library](https://www.notion.so/1061d464528f8010b0cfc60836c20290)
> Part of: [The Mix Programming Language](https://www.notion.so/259ede6504e34505982dde5dc4b63d10)

---

## `array`

> Source: [array](https://www.notion.so/1061d464528f81ef9ff0d976131598d2)

Also: module `dataArray` — identical to `array` but limited to `array(data)`.

```
module array
  // Basics
  def empty : array(S)
  def singleton : S -> array(S)
  def length : array(S) -> number
  def get : number -> array(S) -> S                    // or a[k]; runtime error if out of range
  def safeGet : number -> data -> data                 // returns undefined if out of range
  def set : number -> S -> array(S) -> array(S)
  def add : array(S) -> array(S) -> array(S)           // concatenation; or [1,2] + [3,4]

  // Mapping & filtering
  def map : (S -> T) -> array(S) -> array(T)
  def map2 : (S -> T -> U) -> array(S) -> array(T) -> array(U)
  def indexedMap : (number -> S -> T) -> array(S) -> array(T)
  def indexedMap2 : (number -> S -> T -> U) -> array(S) -> array(T) -> array(U)
  def indexOnlyMap : (number -> T) -> array(S) -> array(T)
  def flatMap : (S -> array(T)) -> array(S) -> array(T)
  def flatten : array(array(T)) -> array(T)
  def zip : array(S) -> array(T) -> array([S,T])       // same-length inputs
  def partition : (S -> bool) -> array(A) -> [array(A), array(A)]  // [true-matches, false-matches]
  def filter : (S -> bool) -> array(S) -> array(S)
  def indexedFilter : (number -> S -> bool) -> array(S) -> array(S)
  def deduplicateKeepFirst : (S -> string) -> array(S) -> array(S)
  def deduplicateKeepLast : (S -> string) -> array(S) -> array(S)

  // Reducing
  def reduce : (S -> T -> T) -> T -> array(S) -> T              // left to right
  def reduceRight : (S -> T -> T) -> T -> array(S) -> T         // right to left
  def indexedReduce : (number -> S -> T -> T) -> T -> array(S) -> T
  def indexedReduceRight : (S -> T -> T) -> T -> array(S) -> T

  // Result chaining
  def andThen : (T -> result(S, U)) -> result(S, array(T)) -> result(S, array(U))

  // Min/Max
  def minIndex/minValue/maxIndex/maxValue : array(S) -> number/S
  def minIndexBy/minValueBy/maxIndexBy/maxValueBy : (S -> T) -> array(S) -> number/S

  // Construction
  def initialize : number -> (number -> S) -> array(S)   // initialize(5, k -> k*k) → [0,1,4,9,16]
  def enumerate : number -> array(number)                 // enumerate(5) → [0,1,2,3,4]
  def keys : array(S) -> array(number)
  def rev : array(S) -> array(S)

  // Streaming
  def indexRange/indexRangeRev : array(S) -> number*
  def valueRange/valueRangeRev : array(S) -> S*

  // Iteration
  def iter : (S -> null) -> array(S) -> null
  def exists : (S -> bool) -> array(S) -> bool
  def forAll : (S -> bool) -> array(S) -> bool
  def find : (S -> bool) -> array(S) -> S                      // undefined if not found
  def findIndex : (data -> bool) -> data -> number              // undefined if not found
  def findMapped : (data -> T) -> data -> T
  def findIndexMapped : (number -> data -> T) -> data -> T

  // Queries (usable inside db.filter)
  def contains : S -> array(S) -> bool
  def featuresOne : array(S) -> array(S) -> bool    // any element in common
  def featuresAll : array(S) -> array(S) -> bool    // a1 is superset of a2

  // Slicing
  def first/last : array(S) -> S                             // runtime error on empty
  def firstWithDefault/lastWithDefault : S -> array(S) -> S
  def skip : array(S) -> array(S)                            // drop first element
  def skipn : number -> array(S) -> array(S)
  def firstn/lastn : number -> array(S) -> array(S)

  // Sort & merge
  def grouped : (S -> data) -> array(S) -> array(array(S))   // split on adjacent similar
  def merge : (S -> data) -> (S -> data) -> array(S) -> array(S) -> array(S)
  def sort : (S -> data) -> array(S) -> array(S)
  def sort_ci : (S -> string) -> array(S) -> array(S)        // case-insensitive
  def dirSort : bool -> (S -> data) -> array(S) -> array(S)  // true=descending
  def dirSort_ci : bool -> (S -> string) -> array(S) -> array(S)
  def sortBy : (A -> A -> bool) -> array(A) -> array(A)      // custom greater-than relation
  def sortLike : string -> array(string) -> array(map(data)) -> array(map(data))

  // Relations
  def innerJoin : (S -> string) -> array(S) -> (T -> string) -> array(T) -> (S -> T -> U) -> array(U)

  // Conversions
  def toStream : array(S) -> stream(S)
  def toStreamPairs : array(S) -> stream([number, S])
module end
```

### Key behaviors

- **Literals**: `[x0, x1, x2, ...]`
- **Update syntax**: functional `update(a with [k] = v)`; imperative `var b = a; b[1] = 42;` (only for `var` variables)
- **`null` handling**: `length` and `keys` treat `null` as empty array; other operations do not
- **`undefined` handling**: `length` and `keys` return `undefined`; other operations signal error
- **`innerJoin`**: key functions must return `string` or `undefined` (undefined rows skipped). Use `|> type(string)` annotation on key fields since the type system can't infer it.
- **`sortLike(key, order, a)`**: sorts array of maps by a field's position in a given order array
- **`deduplicate*`**: projection function must return `string`; compares projected values for equality

---

## `set`

> Source: [set](https://www.notion.so/1061d464528f818fafd7f140445078ab)

Sets of **strings only**. Immutable. Internally maps where key presence = membership (will become opaque type in the future).

```
module set
  type t = data

  // Creation
  def empty : t
  def singleton : string -> t
  def add : string -> t -> t
  def remove : string -> t -> t
  def ofStream : string* -> t
  def ofArray : array(string) -> t

  // Accessing
  def isEmpty/isNotEmpty : t -> bool
  def length : t -> number
  def toStream : t -> string*           // unspecified order
  def toArray : t -> array(string)      // unspecified order
  def toString : t -> string            // comma-delimited, unspecified order

  // Iteration
  def reduce : (string -> A -> A) -> A -> t -> A    // unspecified order
  def map : (string -> string) -> t -> t             // f need not be injective
  def iter : (string -> null) -> t -> null
  def filter : (string -> bool) -> t -> t
  def exists : (string -> bool) -> t -> bool
  def contains : string -> t -> bool

  // Set-theoretic
  def union : t -> t -> t
  def intersection : t -> t -> t
  def difference : t -> t -> t
  def isSubset : t -> t -> bool
module end
```

---

## `map`

> Source: [map](https://www.notion.so/1061d464528f81c1a531d78cea3506ce)

Also: module `dataMap` — identical to `map` but limited to `map(data)`.

```
module map
  // Basics
  def empty : map(S)                                  // or: {}
  def singleton : string -> S -> map(S)
  def length : map(S) -> number
  def keys : map(S) -> array(string)                  // lexicographic order
  def values : map(S) -> array(S)                     // same order as keys
  def bindings : map(S) -> array([string,S])           // same order as keys
  def get : string -> map(S) -> S                      // or m.(k), m."name", m.name
  def safeGet : string -> data -> data                 // undefined even if m is not a map
  def getWithDefault : string -> S -> map(S) -> S
  def set : string -> S -> map(S) -> map(S)
  def remove : string -> map(S) -> map(S)
  def add : map(S) -> map(S) -> map(S)                // merge; m2 values win on conflict

  // Mapping & filtering
  def map : (S -> T) -> map(S) -> map(T)
  def indexedMap : (string -> S -> T) -> map(S) -> map(T)
  def indexOnlyMap : (string -> T) -> map(S) -> map(T)
  def fullMap : (string -> S -> [string, T]) -> map(S) -> map(T)  // remap keys+values; null key = discard
  def filter : (S -> bool) -> map(S) -> map(S)
  def indexedFilter : (string -> S -> bool) -> map(S) -> map(S)

  // Reducing
  def reduce : (S -> T -> T) -> T -> map(S) -> T           // undefined order
  def indexedReduce : (string -> S -> T -> T) -> T -> map(S) -> T

  // Result chaining
  def andThen : (string -> T -> result(S, U)) -> result(S, map(T)) -> result(S, map(U))

  // Streaming
  def indexRange : map(S) -> stream(string)
  def valueRange : map(S) -> stream(S)

  // Iteration
  def iter : (S -> null) -> map(S) -> null
  def contains : string -> map(S) -> bool              // same as m.(k)?
  def find : (S -> bool) -> map(S) -> S                // undefined if not found
  def findIndex : (S -> bool) -> map(S) -> string
  def findMapped : (S -> T) -> map(S) -> T
  def findIndexMapped : (string -> S -> T) -> map(S) -> T

  // Deep access
  def descend : array(string) -> map(data) -> data                     // runtime error if key missing
  def descendWithDefault : array(string) -> data -> map(data) -> data  // substitute on missing key

  // Complex
  def invert : map(array(string)) -> map(array(string))   // swap keys↔values

  // Conversions
  def toStream : map(S) -> stream(S)
  def toStreamPairs : map(S) -> stream([string,S])
  def ofStreamPairs : stream([string, A]) -> map(A)
  def ofStreamMulti : stream([string, A]) -> map(array(A))  // groups duplicate keys
module end
```

### Key behaviors

- **Literals**: `{ name: v }`, `{ "name": v }`, `{ (expr): v }` — 3 forms for map construction
- **Update syntax**: functional `update(m with .name = v)` or `update(m with .(k) = v)`; imperative `var b = m; b.field = 42;`
- **Addition**: `{a:1, b:2} + {b:3, c:4}` → `{a:1, b:3, c:4}` (m2 wins)
- **Missing field**: `m.(k)` returns `undefined` if key not found; `undefined.field` is also `undefined`
- **Cannot set entry to `undefined`**
- **`null` handling**: `length` and `keys` treat `null` as empty map; iterators do not
- **`undefined` handling**: `length` and `keys` return `undefined`; other operations error
- **`descend`**: nested access — `map.descend(["a","b"], {a:{b:10}})` → `10`
- **`invert`**: swaps mapping — `invert({a:["x","y"], b:["x"]})` → `{x:["a","b"], y:["a"]}`

---

## `gset`

> Source: [gset](https://www.notion.so/1061d464528f817ab66ec31970fd0d06)

**Generic sets** — sets of any type where a representative string key can be computed. Immutable. Internally maps from key to value.

```
module gset
  type t(A)
  type kind(A) = [ #getKey: A -> string ]

  def makeKind : (A -> string) -> kind(A)
  def stringKind : kind(string)          // predefined
  def numberKind : kind(number)          // predefined
  def forKind : kind(A) -> ops(A)        // returns record of all set operations
module end
```

### Usage pattern

1. Define a kind: `def colorKind = gset.makeKind(c -> c.code)`
2. Get operations: `def colorSet = gset.forKind(colorKind)`
3. Use like `set` module: `colorSet.add(elem, s)`, `colorSet.contains(elem, s)`, etc.

Predefined kinds: `gset.stringKind`, `gset.numberKind`.

The `forKind` result provides the **same API surface as `set`**: `empty`, `singleton`, `add`, `remove`, `ofStream`, `ofArray`, `isEmpty`, `isNotEmpty`, `length`, `toStream`, `toArray`, `toString`,
`reduce`, `map`, `iter`, `filter`, `exists`, `contains`, `union`, `intersection`, `difference`, `isSubset`.

**Builder recommendation**: define a code module with your custom sets:

```
def boolSet = gset.forKind(gset.makeKind(false -> "f" | true -> "t"))
def numberSet = gset.forKind(gset.numberKind)
```

---

## `uniqArray`

> Source: [uniqArray](https://www.notion.so/604304481fc54c408d28ce8531e3df81)

**Experimental.** Unique arrays — ordered arrays where each element has a string key, and duplicate keys are automatically resolved. Internally an array with uniqueness enforcement.

```
module uniqArray
  type kind(A)
  type t(A)

  // Kind creation
  def kind : (A -> string) -> pref -> kind(A)    // pref = prefLeft | prefRight

  // Creation
  def empty : t(A)
  def singleton : kind(A) -> A -> t(A)
  def fromArray : kind(A) -> array(A) -> t(A)
  def fromStream : kind(A) -> stream(A) -> t(A)
  def appendArray : kind(A) -> t(A) -> array(A) -> t(A)
  def prependArray : kind(A) -> array(A) -> t(A) -> t(A)
  def removeAt : kind(A) -> number -> t(A) -> t(A)
  def removeKey : kind(A) -> string -> t(A) -> t(A)
  def remove : kind(A) -> A -> t(A) -> t(A)

  // Conversion
  def toArray : t(A) -> array(A)
  def toStream/toStreamPairs : t(A) -> stream(A) / stream([number, A])
  def toMap : t(A) -> map(A)
  def toSet : t(A) -> set.t

  // Access
  def length : t(A) -> number
  def isEmpty : t(A) -> bool
  def domain : t(A) -> array(number)
  def get : number -> t(A) -> A
  def keys : t(A) -> array(string)
  def getByKey : string -> t(A) -> A
  def contains : kind(A) -> A -> t(A) -> bool

  // Set operations
  def union/intersection/difference : kind(A) -> t(A) -> t(A) -> t(A)
  def isSubset : t(A) -> t(A) -> bool

  // Iteration
  def reduce : (A -> T -> T) -> T -> t(A) -> T
  def iter : (A -> data) -> t(A) -> null
  def filter : kind(A) -> (A -> bool) -> t(A) -> t(A)
module end
```

### Key behaviors

- **Duplicate resolution** via `pref` parameter:
    - `prefLeft` — keeps leftmost occurrence; adding after existing = silently dropped
    - `prefRight` — keeps rightmost occurrence; adding before existing = silently dropped
- **Kind creation**: `uniqArray.kind(elem -> elem._rmx_id, prefLeft)` for records keyed by `_rmx_id`
- Common kinds: `(s:string) -> s` for strings, `(n:number) -> "" + n` for numbers
- **Future**: `forKind` helper planned (like `gset.forKind`) to avoid passing kind to every call

---

## `circuitarray`

> Source: [circuitarray](https://www.notion.so/1061d464528f81f0a2cef0f7d2f7e850)
> Parent: [The Mix Standard Library](https://www.notion.so/1061d464528f8010b0cfc60836c20290)

Instantiates a Mix module **once per row** in an input array — like `array.map` but for entire reactive modules (with their own cells/state). The result is a "circuit array" from which you extract
individual cell values via `circuitarray.extract`.

### Basic usage

```
import circuitarray
import M   // M(i:data, x:data) ... cell view = ... module end

private cell rows = [ 1, 2, 3 ]
private cell ca = circuitarray.make((i,x) -> M(i,x), rows)
alias M_views = circuitarray.extract(ca, m -> m.view)
```

- `circuitarray.make(factory, rows)` — creates a new circuitarray every time rows change (even if content is identical)
- `circuitarray.extract(ca, m -> m.view)` — extracts a cell value from each module instance

### `circuitarray.makeCached` (since PR #321)

```
private cell ca = circuitarray.makeCached(link(ca), (i,x) -> M(i,x), rows)
```

- Only recreates the circuitarray when **row contents** change (not on every recomputation)
- First param: `link(ca)` — self-reference to the defining cell (required for caching)
- Last param must be a **cell name**, not an arbitrary expression

### `circuitarray.makeOnce` (since PR #282)

```
private cell ca = circuitarray.makeOnce(link(ca), (i,x) -> M(i,x), rows)
```

- Creates the circuitarray **only once**; row changes are **not** reflected (static snapshot)
- Use when rows never change after initial load

### `circuitarray.makeWithKeyedRowCache` (since PR #632)

Keyed cache: reuses component modules across propagation cycles when row identity (by key) is stable.

```
private cell ca =
  circuitarray.makeWithKeyedRowCache(link(ca),
                                     (idx,key,vc,x) -> M(key,vc,x),
                                     (idx,x) -> x._rmx_content_hash,
                                     rows)
```

**Factory function params**: `(idx, key, vc, x) -> M(...)`

- `idx` — numeric index at creation time; do **not** pass to M (position can change without notification)
- `key` — string key (from projection function); stable across repositioning
- `vc` — `link` to a `var cell` with current numeric index; alias inside M to track repositioning
- `x` — input row element (i.e. `rows[idx]`)

**Projection function**: `(idx, x) -> string_key`

- Returns a string key for caching; keys identify rows in the cache
- Return `undefined` or `""` to mark row as non-cacheable (module recreated every time)
- Typical: `(idx, x) -> x._rmx_content_hash`

**Cache age**: By default, cache lives for one propagation cycle. Extend with:

```
def _ = circuitarray.setMaxAge(n)   // n = number of propagation cycles
```

Note: `setMaxAge` is **global** — affects all circuitarrays in the spreadsheet.

### Key behaviors

- Position stability: `makeWithKeyedRowCache` is designed for reorderable lists; cached modules survive position changes
- The visible output of `makeCached` and `makeWithKeyedRowCache` is identical to `make` — only performance differs
- Do not pass `idx` (creation-time position) into module `M` when using keyed cache; use `vc` if position is needed
