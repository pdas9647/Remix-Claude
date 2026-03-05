# Mix Standard Library — Database Modules

>
Sources: [db](https://www.notion.so/1061d464528f81d9b074cf5fd60bc600) · [db-changes](https://www.notion.so/1061d464528f81e18307e33a1de8b990) · [dbRemote](https://www.notion.so/1061d464528f81ceb14cc634b40da637) · [minidb](https://www.notion.so/1061d464528f81b18087c5c5b68df646)
> Parent: [The Mix Standard Library](https://www.notion.so/1061d464528f8010b0cfc60836c20290)

---

## `db` — Primary Database Module

### Types

`database = data` | `record = map(data)` | `recordID = data` (actually a ref) | `query = <opaque>`

### Database Kinds

| Kind       | Disk | Server | Browser | Helper                   |
|------------|------|--------|---------|--------------------------|
| main       | Y    | Y      | Y       | `db.mainDB` / `null`     |
| app DB     | Y    | Y      | Y       | `db.appDB(name)`         |
| session    | N    | Y      | N       | `db.localInMemDB(name)`  |
| in-mem app | N    | Y      | N       | `db.globalInMemDB(name)` |
| remote     | N/A  | Y      | Y       | `db.remoteDB(url, name)` |

**Database as map:** `{"type":"rmx","scope":"main"}` | `{"type":"rmx","scope":"session","db":"name"}` | `{"type":"rmx","scope":"global","db":"name"}` |
`{"type":"rmx","scope":"global","db":"name","inmem":true}` | `{"type":"rmx_remote","url":"baseURL","db":"name"}` | `null` = main DB.

**Save options** (string array `"options"` in database map): `KeepOriginal` (stores original metadata under `_rmx_original/`) | `OriginalIdAlways` (always stores `_rmx_original_id`/
`_rmx_original_rev`) | `DontUseId` (disables preserving well-formed `_rmx_id` from input) | `Upsert` (merge, don't replace).

**Environment:** If `remoteDatabaseServer` is set, local main DB access redirects to that amp server.

### CRUD Operations

All write functions require `appstate` (in actions use `_`). Each operation has variants for single/array/stream, plus `GetID`/`GetIDs` and `Stm` (statement, returns null) variants.

**Pattern:** `{op}One`, `{op}Array`, `{op}ArrayGetIDs`, `{op}Stream`, `{op}StreamGetIDs`, `{op}` (stream shorthand)

- **get**: `getOne(db, id) -> record` (null if not found) | `getOneWithDefault` | `getArray` | `getStream` | `get` | `getWithDefault`
- **save**: `saveOne(_, db, record) -> record` — if `_rmx_id` present, updates; otherwise inserts | stream variants execute eagerly (records saved even if stream not consumed)
- **upsert**: merges fields — new overwrite, missing preserved
- **delete**: `deleteOne(_, db, id) -> record` — returns `{}` if not found | `deleteOneStm` returns null
- **deref**: `deref(db, data) -> data` — loads record if param looks like ID; otherwise returns unchanged

**Temporary refs** for mutually-referencing records: `[{"_rmx_id": db.makeRef("p"), "entity": "parent"}, {"entity": "child", "parent": db.makeRef("p")}] |> db.saveArray(_, null)` — top-level `_rmx_id`
fields collected; matching refs mapped to actual saved IDs.

### Query Pipeline

**Sources:** `db.head(db)` (latest versions) | `db.all(db)` (all versions) | `db.query(db, string)` (native query) | `db.database(q)` (extract db from query)

**Modifiers:** `includeDeleted(bool)` | `includeSuperseded(bool)` | `unrestricted(bool)` | `label(string)`

**Filter:** `db.filter(r -> ..., q)` — compiles to db engine query (error if unsupported). `db.filterWithFallback` — falls back to Mix if db engine can't handle it.

**Compilable filters:** `r.field == value` / `!= value` (strings/bools/null; deep: `r.a.b`) | `r.field?` (existence) | `string.startsWith(r.field, prefix)` | `string.like(r.field, "pattern%")` (
literal, ends with %) | `string.contains("sub", r.field)` | `array.contains(r.field, [...])` | `array.featuresOne(r.field, [...])` (any element in common) | `array.featuresAll(r.field, [...])` (
superset) | `&&`, `||`, `not`, `true`, `false`. Non-row-referencing sub-expressions can be arbitrary Mix (switch filters on/off). Literal ASTs via `db.eval(ast)` inside filter.

**Map:** `db.map(r -> ..., q)` — field accesses executed by db engine, rest by Mix. Deep fields + dbref crossing supported. `r.field1.*` follows reference. `db.mapWithFallback` — usable after any
step. `db.map` only after `db.filter`/`db.head`/`db.all`/`db.query` (no prior Mix postprocessing).

**Sort:** `db.sort(f, q)` ascending | `db.dirSort(reverse, f, q)`. Not rewritten to db engine; use `db.process(query.invoke("order", [f, r]))` for indexed sort.

**Execute:** `toArray` | `toStream` | `length` | `isEmpty` | `isNotEmpty` | `exists(pred)` | `first` | `firstWithDefault` | `last` | `lastWithDefault` | `skip` | `skipn(n)` | `firstn(n)` | `lastn(n)`.
Queries are reusable (execute multiple times).

**Pipeline order:** source → `db.filter` (multiple) → `db.map` (once) → then `filterWithFallback` / `mapWithFallback` / `sort` in any order → execute.

### AST & Low-Level

`db.eval(ast)` — insert AST into filter/map. `db.process(ast, q)` — add AST to db query. `db.processWithFallback(ast, fallback, q)` | `db.processPipeline(asts, q)`. `db.parse(string) -> ast` (single
clause) | `db.parseQueryString(db, string) -> ast` (uses db engine parser).

### Live Queries

`live cell q = db.head() |> db.filter(.entity == "news") |> db.toArray` — `db.head`/`all`/`query` must appear directly on RHS of `live cell`.

### References

`db.makeRef("") -> null` | `db.makeRef(id) -> db.ref(id)` | `db.stringRef(ref) -> string` | `db.isRef(val) -> bool`. JSON: `{"_rmx_type": "{tag}case:db.ref", "_rmx_value": "<id>"}`.

### Anonymous Links

`db.makeAnonymousLink(appstate, db, endpoint) -> result(string, string)` — shareable link for GET/POST/PUT; invoked as the creator's identity.

---

## `db-changes` — Old → New API Migration

1. **Functions moved into module**: `db_save` → `db.save`, `db_save_one` → `db.saveOne`, etc.
2. **`query` is no longer a stream**: Must call `db.toStream`/`db.toArray`/`db.first` to execute. Insert `db.toStream` before stream operations.
3. **No auto-fallback in `db.filter`**: Unsupported ops cause compile-time error. Use `db.filterWithFallback` or convert to stream first.
4. **No auto-expansion of dbrefs**: Recursive expansion (`{ ** }`) must be explicit via `db.map(_ -> db.eval(query.cloneMembersWithExpansion))`.
5. **`db.map` projection pushdown**: Field accesses delegated to db engine; deep fields + dbref traversal in projections (not filters yet).
6. **Live queries**: Just `live cell` — detected automatically.

---

## `dbRemote` — HTTP Database Access

Same CRUD ops as `db` but forces HTTP protocol. Database param: `db.remoteDB("baseURL", "dbname")`. Additional: `queryASTToStream(db, asts, includeDeleted, includeSuperseded) -> stream(record)`. *
*Limitations:** no live queries; requires server privileges.

---

## `minidb` — In-Session Pure-Mix Database

Session-only (not persisted), no indexes except `_rmx_id` (small datasets only), pure Mix (works in Wasm), accessible via `minidb.DB` (`{"type":"minidb"}`), supports transactions. Unsupported: string
queries, grouping, `_rmx_rev`/`_rmx_previous`/`_rmx_content_hash`.

**API:** `populate(appstate, records)` — fast load from previous export (wipes existing; empty array erases); must not be used with incomplete records or regular DB records | `commit(appstate)` |
`rollback(appstate)` | `committedRecords(sel) -> stream` | `currentRecords(sel) -> stream` | `changedRecords(sel) -> stream` | `head`/`all` selections | `includeDeleted`/`includeSuperseded` modifiers.

**Usage:** `db.head(minidb.DB) |> db.filter(r -> r.name == "Vijay") |> db.toArray`. Copy from regular DB: `db.head() |> db.filter(r -> r.entity?) |> db.toStream |> db.save(minidb.DB)` (IDs change on
save).