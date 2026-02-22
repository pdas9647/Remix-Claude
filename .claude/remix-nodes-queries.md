# Remix Nodes — Queries & Query Syntax

> Parent: Remix Documentation > User Guides and Tutorials >
> [Remix Studio User Guide](https://www.notion.so/1051d464528f8007a5f5d40969c49427) >
> [Nodes](https://www.notion.so/13b1d464528f802c90b7c60485b36842)

**Covers:** Queries (QB 2.0, DB architecture, query methods), Query Syntax (full filter/projection/aggregation/calculation reference)

---

## Queries

> Source: [Queries](https://www.notion.so/1051d464528f80dcb464fae23efefcf2)

### Query Builder 2.0 (introduced October 2025)

Significant changes vs QB1:

- **Richer projections**: designed to get all data in one query — speeds up screens
- **Simpler advanced mode**: easier to add expression power when needed
- **No Mix post-processing**: all post-processing removed from queries; must be handled by users. Function projections no longer supported. Certain filters also removed.
- QB 1.0 nodes can be migrated to QB 2.0 when a migration button appears. Revert is possible until leaving the module.
- Can still create QB 1.0 nodes from the "New" menu (for compatibility). Support for 1.0 will not be removed any time soon.

### DB Architecture

Every Remix runtime has a **linear record store + automatic dynamic index**. No user control over which fields are indexed — every indexable field is indexed automatically. Records are untyped and can
include any serializable Mix `data`.

### 3 Ways to Query

| Method            | Use                                                                                                         |
|-------------------|-------------------------------------------------------------------------------------------------------------|
| **Query AST**     | JSON array passed directly; used in registered queries                                                      |
| **Query string**  | Parseable to AST; simple but adds parsing overhead. **Never use with user-provided input** (injection risk) |
| **Mix rewriting** | Compiler rewrites `db.filter`/`db.map` Mix lambdas into query AST                                           |

### Query Execution Layers

1. **Indexed part** → compute set of DB rows from index (fastest)
2. **Non-indexed filters** → applied to materialized record stream from store
3. **Mix post-processing** → arbitrary filters/projections in Mix VM (slowest)

### Mix Query API

**Sources:**

- `db.head(dbConfig)` — all current records (`db.head()` = main DB)
- `db.all(dbConfig)` — same but includes old versions
- `db.query(dbConfig, queryStr)` — start from an arbitrary query string

**Processors:**

- `db.process(ast)` — add one AST element
- `db.processWithFallback(ast, fallback)` — for uncertain expressions
- `db.postprocess(stream.fn)` — Mix-side processing (after this, only postprocess or processWithFallback allowed)

**Filter/map rewriting:**

- `db.filter(r -> ...)` / `db.map(r -> ...)` — compiler rewrites to query AST
- `db.filterWithFallback(r -> ...)` / `db.mapWithFallback(r -> ...)` — both paths generated
- Record expressions supported: `db.filter(.entity == "person")`, `db.map({name: .first_name})`

**Output functions:**

- `db.toArray` — materialize full result array
- `db.toStream` — lazy stream (better when not consuming all results)
- `db.first` — first result (error if empty)
- `db.length` — count

**Example:**

```plain text
db.head() |> db.filter(.entity == "person") |> db.map({first_name: .first_name, last_name: .last_name, name: .first_name + " " + .last_name}) |> db.toArray
```

### HTTP Query API

```bash
curl -H "Authorization: Bearer $AMP_TOKEN" \
  https://example.remixlabs.com/a/myapp/query \
  --url-query 'query="entity" == "person" | { first_name, last_name }'
```

JSON output by default; MessagePack also supported.

History params: `includeHistory=true` (old versions), `includeDeleted=true` (tombstones).

### Index-Capable Operations

- Comparisons (`==`/`!=`/`<`/`<=`/`>`/`>=`) on strings, refs, booleans, null, calendar types
- `string.hasPrefix("foo", .field)` — string prefix test
- `string.contains("text", .field)` — string contains
- `in` / `all in` array comparisons for indexed elements
- Sorts and groupings on string/ref fields
- Boolean combinations of indexable terms (`and`/`or`/`not`)
- Nested field access: `.friend.neighbor.name == "fred"` (works for nested maps or refs across records)
- Note: **nested comparisons with non-string types** like `.friend.neighbor.age == 10` are only index-supported for the indexed types above

### History Queries

- Old versions: `db.includeSuperseded(true)` processor, or `db.all()` source
- Tombstones: `db.includeDeleted(true)` — tombstones lack content fields; query on `_rmx_id` or follow `_rmx_previous` ref
    - Example: `db.filter(._rmx_deleted == true && ._rmx_previous.entity == "person")`

### Auto-Refresh (Live Query)

A Query tile can be set to **auto-refresh**: fires automatically whenever the DB changes (vs. requiring manual REFRESH button press). Enable via: Edit menu of the Query tile → "LIVE" option toggle.

### Referenced Pages (not yet fetched)

- [DB model](https://www.notion.so/1061d464528f80248ae6cfe395ee6a9c)
- [`db` stdlib module](https://www.notion.so/1061d464528f81d9b074cf5fd60bc600)
- [`query` stdlib module](https://www.notion.so/1061d464528f818b9732d6b6f321f187)
- [Platform server HTTP API](https://www.notion.so/1061d464528f80449377e6c07e72b30b)

---

## Query Syntax

> Source: [Query syntax](https://www.notion.so/205ca411bf684f5593c3469e39de2af8)

The query layer is a `filter → projection` pipeline. `|` separates stages in string form; the AST form is an array of query operations.

### Filter Operations

| Operation    | String             | AST opcode                             | Mix (`db.filter(r -> ___)`)   |
|--------------|--------------------|----------------------------------------|-------------------------------|
| equals       | `key == val`       | `["comparison", "==", key, val]`       | `r.key == val`                |
| not equals   | `key != val`       | `["comparison", "!=", key, val]`       | `r.key != val`                |
| greater than | `key > val`        | `["comparison", ">", key, val]`        | `r.key > val`                 |
| less than    | `key < val`        | `["comparison", "<", key, val]`        | `r.key < val`                 |
| contains     | `key contains val` | `["comparison", "contains", key, val]` | `string.contains(val, r.key)` |
| prefix       | `key like val`     | `["comparison", "like", key, val]`     | `string.hasPrefix(val, .key)` |
| exists       | `exists key`       | `["exists", key]`                      | `r.key?`                      |
| after        | `after val`        | `["after", val]`                       | not supported                 |

**Array filters:**

| Operation                       | String                 | AST opcode                                 |
|---------------------------------|------------------------|--------------------------------------------|
| any value in key is in array    | `key in [...]`         | `["comparison", "in", key, [...]]`         |
| key values include all of array | `key all in [...]`     | `["comparison", "all_in", key, [...]]`     |
| key value is NOT in array       | `key not in [...]`     | `["comparison", "not_in", key, [...]]`     |
| no values are in array          | `key not all in [...]` | `["comparison", "not_all_in", key, [...]]` |

**Boolean:** `and`/`&&`, `or`/`||`, `not`/`!`; no-op = `["nullFilter"]`

**Important:** Always quote keys and string values in query strings (rules are not consistent, may change).

### Key Types

| Key           | String        | AST                                                                     |
|---------------|---------------|-------------------------------------------------------------------------|
| Bare string   | `"field"`     | `"field"`                                                               |
| Token         | `field`       | `["token", "field"]`                                                    |
| Object member | `.field`      | `["objectMember", null, ["token", "field"]]`                            |
| Nested        | `inner.field` | `["objectMember", ["objectMember", null, "inner"], ["token", "field"]]` |
| Parameter     | `$field`      | `["param", "field"]`                                                    |

**Nested projection note:** `field` refers to current object; `.field` refers to root object.

### Value Types and Indexability

| Type                                            | Indexed?                                     |
|-------------------------------------------------|----------------------------------------------|
| null, undefined, bool, string (≤256 chars), ref | yes                                          |
| int, number                                     | no                                           |
| list                                            | yes, per element                             |
| map                                             | no (but nested indexable fields are indexed) |
| Mix case types                                  | only if argument has `_rmx_index` field      |

### Projections

| Member                  | String                    | AST                                         |
|-------------------------|---------------------------|---------------------------------------------|
| key: value              | `key: value`              | `["assignMember", "key", value]`            |
| with default            | `key: value ?? "default"` | `["assignMember", "key", value, "default"]` |
| copy field              | `field`                   | `["assignMember", "field", "field"]`        |
| clone all fields        | `*`                       | `["cloneMembers"]`                          |
| clone all + expand refs | `**`                      | `["cloneMembersWithExpansion"]`             |
| remove field            | `-key`                    | `["removeMember", "key"]`                   |
| follow single ref       | `+key`                    | `["followRef", key]`                        |
| follow recursively      | `++key`                   | not supported                               |

### Aggregations

| Operation            | String            | Mix                         |
|----------------------|-------------------|-----------------------------|
| count                | `count()`         | `db.length`                 |
| group by key         | `group(key)`      | not available               |
| order ascending      | `orderAsc(key)`   | `db.sort(.key)`             |
| order descending     | `orderDesc(key)`  | `db.dirSort(true, .key)`    |
| order with direction | `order(key, dir)` | `db.dirSort(not dir, .key)` |
| reverse              | `reverse()`       | not available               |

### Calculations

Wrapped as `["calculation", calc]`. Arithmetic: `+`, `-`, `*`, `/`, `%`.

- Any operation with `null` → `null`
- Any operation with `undefined` → `undefined`
- `+` also concatenates strings, lists, maps

### Subqueries

For nested objects: `inner.(member == "test" and another == "foo")` → `["subquery", "inner", [...]]`
Subqueries are **filter-only** — no projections inside.

### Query Parameters

Right-hand side of comparison can be a parameter: `$field` / `["param", "field"]`.
Available in **cloud agent registered queries only** — not in Go DB engine on platform server.