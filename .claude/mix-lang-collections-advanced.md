# Mix Standard Library — Collection Types: gset, uniqArray, circuitarray

> Parent: [The Mix Standard Library](https://www.notion.so/1061d464528f8010b0cfc60836c20290)
> Part of: [The Mix Programming Language](https://www.notion.so/259ede6504e34505982dde5dc4b63d10)
> Split from mix-lang-collections.md — covers gset, uniqArray, circuitarray.

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