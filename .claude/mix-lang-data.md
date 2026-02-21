# Mix Standard Library — Data & Database Modules

> Sources:
> - [db](https://www.notion.so/1061d464528f81d9b074cf5fd60bc600)
> - [db-changes](https://www.notion.so/1061d464528f81e18307e33a1de8b990)
> - [dbRemote](https://www.notion.so/1061d464528f81ceb14cc634b40da637)
> - [minidb](https://www.notion.so/1061d464528f81b18087c5c5b68df646)
> - [query](https://www.notion.so/1061d464528f818b9732d6b6f321f187)
    > Parent: [The Mix Standard Library](https://www.notion.so/1061d464528f8010b0cfc60836c20290)

---

## `db` — Primary Database Module

### Types

```
type database = data
type record = map(data)
type recordID = data   // actually a ref
type query = <opaque>
```

### Database Kinds

| Kind       | On Disk | Platform Server | Browser | Helper function          | Notes                                     |
|------------|---------|-----------------|---------|--------------------------|-------------------------------------------|
| main       | ✅       | ✅               | ✅       | `db.mainDB` / `null`     | Default database of the app               |
| app DB     | ✅       | ✅               | ✅       | `db.appDB(name)`         |                                           |
| session DB | ❌       | ✅               | ❌       | `db.localInMemDB(name)`  | In-memory, session-scoped                 |
| in-mem app | ❌       | ✅               | ❌       | `db.globalInMemDB(name)` | Global in-mem, tied to global app DB      |
| remote DB  | N/A     | ✅               | ✅       | `db.remoteDB(url, name)` | HTTP protocol; live queries not supported |

**Database type as map:**

- `{ "type": "rmx", "scope": "main" }` — main on-disk DB
- `{ "type": "rmx", "scope": "session", "db": "name" }` — session in-mem DB
- `{ "type": "rmx", "scope": "global", "db": "name" }` — global on-disk DB
- `{ "type": "rmx", "scope": "global", "db": "name", "inmem": true }` — global in-mem DB
- `{ "type": "rmx_remote", "url": "baseURL", "db": "name" }` — remote DB over HTTP
- `null` — same as main DB

**Save Options** (as `"options"` array in database map):

- `KeepOriginal` — stores original metadata under `_rmx_original/`
- `OriginalIdAlways` — always stores `_rmx_original_id` and `_rmx_original_rev`
- `DontUseId` — disables preserving well-formed `_rmx_id` values from input
- `Upsert` — merge rather than replace when `_rmx_id` exists

**Environment:** If `remoteDatabaseServer` is set, any local main DB access is redirected to that amp server.

### Module Signature

```
module db
  import q = query

  // Get databases
  def mainDB : database
  def appDB : string -> database
  def localInMemDB : string -> database
  def globalInMemDB : string -> database
  def remoteDB : string -> string -> database             // since PR1571

  // CRUD – single record
  def getOne : database -> recordID -> record
  def getOneWithDefault : record -> database -> recordID -> record
  def saveOne : appstate -> database -> record -> record
  def saveOneGetID : appstate -> database -> record -> recordID
  def upsertOne : appstate -> database -> record -> record
  def upsertOneGetID : appstate -> database -> record -> recordID
  def deleteOne : appstate -> database -> recordID -> record
  def deleteOneStm : appstate -> database -> recordID -> null

  // CRUD – array/stream
  def getArray : database -> array(recordID) -> array(record)
  def getStream : database -> array(recordID) -> stream(record)
  def getStreamWithDefault : record -> database -> array(recordID) -> stream(record)
  def get : database -> array(recordID) -> stream(record)
  def getWithDefault : record -> database -> array(recordID) -> stream(record)
  def saveArray : appstate -> database -> array(record) -> array(record)
  def saveArrayGetIDs : appstate -> database -> array(record) -> array(recordID)
  def saveStream : appstate -> database -> stream(record) -> stream(record)
  def saveStreamGetIDs : appstate -> database -> stream(record) -> stream(recordID)
  def save : appstate -> database -> stream(record) -> stream(record)
  def upsertArray : appstate -> database -> array(record) -> array(record)
  def upsertArrayGetIDs : appstate -> database -> array(record) -> array(recordID)
  def upsertStream : appstate -> database -> stream(record) -> stream(record)
  def upsertStreamGetIDs : appstate -> database -> stream(record) -> stream(recordID)
  def upsert : appstate -> database -> stream(record) -> stream(record)
  def deleteArray : appstate -> database -> array(recordID) -> array(record)
  def deleteArrayStm : appstate -> database -> array(recordID) -> null
  def deleteStream : appstate -> database -> stream(recordID) -> stream(record)
  def deleteStreamStm : appstate -> database -> stream(recordID) -> null
  def delete : appstate -> database -> stream(recordID) -> stream(record)
  def deref : database -> data -> data

  // Base queries
  def head : database -> query
  def all : database -> query
  def query : database -> string -> query
  def database : query -> database

  // Query modifiers
  def includeDeleted : bool -> query -> query
  def includeSuperseded : bool -> query -> query
  def unrestricted : bool -> query -> query
  def label : string -> query -> query

  // Filter / Map / Sort
  def filter : (record -> bool) -> query -> query
  def filterWithFallback : (record -> bool) -> query -> query
  def map : (record -> record) -> query -> query
  def mapWithFallback : (record -> record) -> query -> query
  def sort : (record -> data) -> query -> query
  def dirSort : bool -> (record -> data) -> query -> query

  // Execute query
  def length : query -> number
  def isEmpty : query -> bool
  def isNotEmpty : query -> bool
  def exists : (record -> bool) -> query -> bool
  def toArray : query -> array(record)
  def toStream : query -> record*

  // Slice
  def first : query -> record
  def firstWithDefault : record -> query -> record
  def last : query -> record
  def lastWithDefault : record -> query -> record
  def skip : query -> query
  def skipn : number -> query -> query
  def firstn : number -> query -> query
  def lastn : number -> query -> query

  // AST
  def eval : q.ast -> data
  def process : q.ast -> query -> query
  def processWithFallback : q.ast -> (stream(record) -> stream(record)) -> query -> query
  def processPipeline : array(q.ast) -> query -> query
  def parse : string -> q.ast
  def parseQueryString : database -> string -> q.ast     // since PR1651

  // Anonymous link
  def makeAnonymousLink : appstate -> database -> string -> result(string, string)

  // References
  private case ref(string)
  type reference = ref | null
  def makeRef : string -> recordID
  def stringRef : recordID -> string
  def isRef : recordID -> bool
module end
```

### Key Behaviors

- `appstate` parameter required for writes. In action closures, use `_` (underscore): `db.saveOne(_, db, record)`
- `save` / `upsert` / `delete` return streams but execute eagerly — records are saved/deleted even if stream not consumed
- `saveOne`: if record has `_rmx_id`, updates existing record; otherwise inserts new one
- `upsert`: merges fields — new fields overwrite, missing fields preserved
- `deleteOne`: if ID not found, returns `{}`; does not error
- `getOne`: returns `null` if ID not found
- `deref`: loads record if param looks like an ID; otherwise returns param unchanged

**Temporary refs** — save multiple mutually-referencing records:

```
[{"_rmx_id": db.makeRef("p"), "entity": "parent"},
 {"entity": "child", "parent": db.makeRef("p")}]
|> db.saveArray(_, null)
```

### Filter Compilability Rules

`db.filter` compiles filters to db engine queries. Supported:

- Equality: `r.field == value`, `r.field != value` (strings, bools, null only; deep fields: `r.a.b`)
- Existence: `r.field?`
- Prefix: `string.startsWith(r.field, prefix)`
- Like: `string.like(r.field, "pattern%")` (literal pattern ending with `%`)
- Contains (string): `string.contains("substring", r.field)`
- Contains (array): `array.contains(r.field, [x1, x2, ...])`
- `array.featuresOne(r.field, [x1, ...])` — field (array of strings) has any element in list
- `array.featuresAll(r.field, [x1, ...])` — field (array) is a superset of list
- Boolean: `&&`, `||`, `not`, `true`, `false`

Use `db.filterWithFallback` for unsupported ops (e.g. `>=`). Use `db.toStream | stream.filter(...)` for full Mix-level control.

### Query Pipeline Structure

Valid sequence: `db.head/all/query` → (optional) `db.filter` (multiple) → (optional) `db.map` (once) → then `db.filterWithFallback`, `db.mapWithFallback`, `db.sort` in any order → finally execute (
`db.toArray`, `db.toStream`, etc.)

`db.map` cannot follow `db.filterWithFallback` (use `db.mapWithFallback` instead).

### Live Queries

Live queries via the `messaging` module. Syntax:

```
live cell q = db.head() |> db.filter(.entity == "news") |> db.toArray
```

`db.head`, `db.all`, or `db.query` must appear directly on the right-hand side of `live cell`.

### References

`db.makeRef("")` → null; `db.makeRef(id)` → `db.ref(id)`. Use `db.stringRef` to extract ID. `db.isRef` checks if value is reference. JSON encoding:
`{"_rmx_type": "{tag}case:db.ref", "_rmx_value": "<id>"}`.

---

## `db-changes` — Migration Notes (Old → New API)

> Source: [db-changes](https://www.notion.so/1061d464528f81e18307e33a1de8b990)

Key differences from the old db module:

1. **Functions moved into module**: `db_save` → `db.save`, `db_save_one` → `db.saveOne`, etc.

2. **`query` is no longer a stream**: Must call `db.toStream`, `db.toArray`, `db.first`, etc. to execute. Old style `db.head() | stream.map(...)` no longer works — insert `db.toStream` before stream
   operations.

3. **No auto-fallback in `db.filter`**: Unsupported ops (e.g. `>=`) cause compile-time error. Use `db.filterWithFallback` or convert to stream first.

4. **No auto-expansion of dbrefs**: Recursive expansion (`{ ** }`) must be explicit:
   ```
   db.head() |> db.map(_ -> db.eval(query.cloneMembersWithExpansion)) |> db.toStream
   ```

5. **`db.map` projection pushdown**: Field accesses in map projections are executed by db engine; non-field-access parts run in Mix. Deep field accesses and dbref traversal supported in projections (
   not in filters yet).

6. **Live query simplification**: Just use `live cell` — the db query is detected automatically. No extra wrapping needed.

---

## `dbRemote` — HTTP Database Access

> Source: [dbRemote](https://www.notion.so/1061d464528f81ceb14cc634b40da637)

Provides same CRUD operations as `db` but forces HTTP protocol. Database parameter must be:

```
{ "type": "rmx_remote", "url": "baseURL", "db": "dbname" }
```

Create via `db.remoteDB("baseURL", "dbname")`. Alternatively, use corresponding `db.*` functions directly with a remote database value — `dbRemote` functions are the explicit HTTP-only variants.

Additional function:

```
def queryASTToStream : db.database -> array(data) -> bool -> bool -> stream(db.record)
```

**Limitations vs local db:**

- Live queries (db subscriptions): not supported
- Requires appropriate server privileges

---

## `minidb` — In-Session Pure-Mix Database

> Source: [minidb](https://www.notion.so/1061d464528f81b18087c5c5b68df646) — *Note: marked "Not yet merged" at time of doc*

### Characteristics

- Session-only — not persisted to disk
- No indexes except `_rmx_id` — suited for small datasets only
- Pure Mix implementation — works in WebAssembly mode
- Accessible from `db` module via `minidb.DB` as database param
- Supports transactions (commit/rollback)
- Unsupported: string queries, grouping, `_rmx_rev`/`_rmx_previous`/`_rmx_content_hash`

### Module Signature

```
module minidb  // since PR 1195
  type record = data
  type recordOrNull = data
  type strictRecord = map(data)
  type recordID = data
  type selection = { includeSuperseded: bool, includeDeleted: bool }

  def populate : appstate -> array(strictRecord) -> null
  def commit : appstate -> null
  def rollback : appstate -> null

  def head : selection
  def all : selection
  def includeDeleted : bool -> selection -> selection
  def includeSuperseded : bool -> selection -> selection
  def committedRecords : selection -> stream(strictRecord)
  def currentRecords : selection -> stream(strictRecord)
  def changedRecords : selection -> stream(strictRecord)
module end
```

### Usage

Use `minidb.DB` (= `{ "type": "minidb" }`) as the database param with any `db.*` function:

```
db.head(minidb.DB) |> db.filter(r -> r.name == "Vijay") |> db.toArray
```

**Populate** — fast load from previous session export (do not use with records from the regular DB):

```
minidb.populate(_, previousRecords)
```

**Copy from regular DB** (IDs change on save):

```
db.head() |> db.filter(r -> r.entity?) |> db.toStream |> db.save(minidb.DB)
```

**Transactions:**

- `minidb.commit` — accept current changes
- `minidb.rollback` — drop changes since last commit
- `committedRecords` — last committed state
- `currentRecords` — current transaction state
- `changedRecords` — changes since transaction start

---

## `query` — AST Query Builder

> Source: [query](https://www.notion.so/1061d464528f818b9732d6b6f321f187)
> Typically imported as: `import q = query`

### Module Signature

```
module query
  type ast = data

  // Filter comparisons
  def cmp : string -> string -> data -> ast   // op, field, value
  def cmpExpr : string -> ast -> data -> ast
  def eq : string -> data -> ast
  def neq : string -> data -> ast
  def lt : string -> data -> ast
  def gt : string -> data -> ast
  def le : string -> data -> ast
  def ge : string -> data -> ast
  def contains : string -> data -> ast
  def like : string -> data -> ast
  def featuresOne : string -> data -> ast
  def featuresAll : string -> data -> ast

  // Filter logic
  def nullFilter : ast
  def token : string -> ast
  def exists : string -> ast
  def afterWatermark : string -> ast
  def afterRow : number -> ast
  def not_ : ast -> ast
  def and : data -> ast          // array(ast) -> ast
  def or : data -> ast           // array(ast) -> ast
  def invoke : string -> data -> ast   // string -> array(ast) -> ast

  // Projection
  def object : data -> ast       // array(ast) -> ast
  def cloneMembers : ast
  def cloneMembersWithExpansion : ast
  def assignMember : string -> ast -> ast
  def removeMember : string -> ast
  def objectMember : ast -> string -> ast
  def followRef : ast -> ast

  // Scalars/lists for typed comparisons
  def scalarFallback : data -> ast
  def scalarIndexed : data -> ast
  def listFallback : data -> ast
  def listIndexed : data -> ast

  // Aggregation
  def count : ast
  def group : string -> ast

  // Map constructors
  def filterOfMap : data -> ast
  def filterOfMapFallback : data -> ast
  def projectionOfMap : data -> ast
module end
```

### Filter AST Nodes

- `eq("field", value)` / `neq` / `lt` / `gt` / `le` / `ge` — field comparisons (field name as string)
- `exists("field")` — record has this field
- `afterWatermark(watermark)` — record is after `_rmx_rev` watermark
- `not_(ast)`, `and([ast1, ...])`, `or([ast1, ...])` — boolean composition
- `nullFilter` — matches everything (useful as identity in AND/OR)
- `featuresOne(field, list)` / `featuresAll(field, list)` — list should be `listIndexed` or `listFallback`

Example: `.field == "foo"` → `q.eq("field", "foo")`

### Projection AST Nodes

- `objectMember(null, "fieldname")` — access field from current record; nest for deep access
- `followRef(valueAst)` — follow db reference to full record
- `object([...])` — create new object from members:
    - `cloneMembers` — copy all current fields
    - `cloneMembersWithExpansion` — copy + expand all refs recursively
    - `assignMember("key", valueAst)` — set field
    - `removeMember("key")` — remove field
- `scalarIndexed(v)` / `scalarFallback(v)` — typed scalar values
- `listIndexed(a)` / `listFallback(a)` — typed lists
- `invoke("funcName", [args])` — db engine function (e.g. `invoke("count", [])`)

Example: projection `{ cde: { *, -bar } }`:

```
q.object([q.assignMember("cde", q.object([q.cloneMembers, q.removeMember("bar")]))])
```

### Map Constructors

- `q.filterOfMap({"a": "A", "b": true})` → `q.and([q.eq("a", "A"), q.eq("b", true)])` — nested maps flatten to dot-notation
- `q.filterOfMapFallback({"a": "A", "b": 42})` → uses `scalarFallback` for non-indexed types
- `q.projectionOfMap({"a": 1, "b": 1})` → extracts just the keys as field projections (values ignored)

### Using ASTs

Three ways to use AST nodes:

1. Inside `db.filter(r -> db.eval(q.afterWatermark(wm)))` — filter AST at right place
2. Inside `db.map(r -> db.eval(q.object([...])))` — projection AST
3. Directly via `db.process(ast)` or `db.processPipeline([ast1, ast2])` in the pipeline

A full pipeline is an **array** of AST nodes: `part1 | part2 | part3` → `[ast1, ast2, ast3]`.

---

## `resource` — Static Resource Access *(Soft-Deprecated)*

> Source: [resource](https://www.notion.so/d6783f5009fe471a8e626e37b6598a20)
> **Deprecated — migrate to [`file`](./mix-lang-io.md) instead.**

```
module resource
  def url : option(string) -> string -> result(url, string)
  def bundleUrl : option(string) -> string -> string -> result(url, string)
  def get : option(string) -> string -> result(binary, string)
module end
```

First arg: optional DB name (`null` = current DB). Second arg: resource name. `bundleUrl` takes path inside bundle as third arg.

**Storage:** Resources stored as `_rmx_type == "resource"` records in DB during development. At compile time, a `resource.json` snapshot is created. At runtime, only `resource.json` is read. URLs are
environment-specific: `https://` on web, `file://` in widgets.

---

## `registry` — Session-Scoped Value Store *(Experimental)*

> Source: [registry](https://www.notion.so/1061d464528f813f84cad16503e0c479)

```
module registry
  type key
  def add : any -> key
  def addOnce : any -> key        // since PR#1012
  def find : key -> option(any)
module end
```

Stores arbitrary **case values** and allows fast lookup by key. The key encodes the case type name, enabling retrieval without knowing the concrete case type at lookup time.

**Pattern:** Wrap the value in a case type before adding; unwrap at lookup via pattern matching.

```
case myFn(string -> string)
def k = registry.add(myFn(string.uppercase))
// later:
registry.find(k) |> type(any) |> (some(myFn(f)) -> f | _ -> fail("not found"))
```

**`addOnce`** — stores only on first call (subsequent calls are no-ops for the same invocation site). Use for module-level singletons like sequence generators:

```
private def key = registry.addOnce(genstored(gentype(stream.enumUnbound())))
```
