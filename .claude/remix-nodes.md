# Remix Nodes — Studio User Guide: Nodes Section

> Parent: Remix Documentation > User Guides and
> Tutorials > [Remix Studio User Guide](https://www.notion.so/1051d464528f8007a5f5d40969c49427) > [Nodes](https://www.notion.so/13b1d464528f802c90b7c60485b36842)

**Nodes section pages (fetched):** In Params/Out Params, Cards & Components, Data Objects/Values/Transforms, Queries, Query Syntax, Working with State
**Not yet fetched:** Decisions & State Machines (1051d464528f80a48689d2743814b1af), Functions & Code Modules (8c02beb038ec40fea16762bbd8aabfe8), Actions (1051d464528f80c9b498daa44df9b685),
Annotations (1051d464528f8018a17fd69389243631)

---

## In Params and Out Params

> Source: [In Params and Out Params](https://www.notion.so/1141d464528f809fbfa6d9983ad074fe)

### Reordering Bindings

Bindings in in/out-params nodes can be reordered with up/down arrows. After reordering and publishing, **Push, Navigate, and Agent Connect nodes** referencing those params are automatically "patched"
to reflect the updated order when next opened.

### Agent Error Out-Params

Out-params in agents have 2 extra bindings beyond the normal data bindings:

- **error trigger** — invoke this to make the agent return an error
- **error string** — the error message returned

In the calling code:

- The `error next` binding fires
- The `error data` binding holds the error string

Normal data bindings (0 or more) are returned only if the agent ran correctly.

---

## Cards & Components

> Source: [Cards & Components](https://www.notion.so/10ba63e7a90848f29d5a50f8bd4ac61d)

### Cards

**Style Units** — CSS properties that take numerical values now support units beyond `px` (added December 2024): `em`, `rem`, `%`, `vw`, `vh`, etc.

**Text Inputs** — `debounce` property: milliseconds before the value in a text input is sent to the server. Default: `150ms`.

### Components

**Headless Components** — Components that return only data, not card data for display. Mark via: In/Out-Params → "Set as Headless" option. The Card node marked as "Screen" changes format; above in the
stack, the component view shows in/out param bindings in a more useful manner.

**Triggered Components (advanced)** — Similar to screens: no action in/out bindings. A runner can start and stop these components externally.

- Runner workspace: `https://agt.remixlabs.com/ws/com_remixlabs_simon`
- Example: `https://remix-dev.remixlabs.com/e/edit/builder-examples/triggered_comp`

### File Uploading

> Source: [File Uploading](https://www.notion.so/12e1d464528f8088900dcf2f4154cc24)
> See also: page [1571d464528f80e9a9fee7fbbf80a0c5](https://www.notion.so/1571d464528f80e9a9fee7fbbf80a0c5) (unfetched)

**Setup requires two nodes:**

- **Local File Controller** — provides `Local File` output to the app; set on its own node
- **Card with File Input and/or dropzone** — bound to the `default-screen` binding

**Controller properties:**

| Property   | Description                                                                                                       |
|------------|-------------------------------------------------------------------------------------------------------------------|
| `multiple` | Whether one or several files can be selected (applies to drag & drop too)                                         |
| `accept`   | File type filter; see [MDN accept attribute](https://developer.mozilla.org/en-US/docs/Web/HTML/Attributes/accept) |

**Important:** Switching from multiple to single file → also toggle the action binding.

**Directory drop** (since November 2024) — Files inside a dropped directory are returned with their path as the name.

---

## Data Objects, Values and Transforms

> Source: [Data objects, values and transforms](https://www.notion.so/1051d464528f8022a535d4ab9342d1b0)
> Note: Page content was truncated on fetch — re-fetch for full detail.

### Transform: Join

The **Join** node performs an **inner join** (SQL inner join semantics). It projects the union of fields from the joined data — it does **not** handle field name collisions.

---

## Working with State

> Source: [Working with State](https://www.notion.so/1431d464528f8060964ecdcbfba99fd7)

### Three State Scopes

| Scope            | Description                                          | Use Case                                                                                    |
|------------------|------------------------------------------------------|---------------------------------------------------------------------------------------------|
| **Local State**  | Operates at one level in the component graph         | Component-local data                                                                        |
| **Screen State** | Available throughout the component stack of a screen | Building autonomous components for assembly                                                 |
| **App State**    | Available across screen navigation (global)          | Persistent data across navigation; powerful but hard to debug, potential performance issues |

Get State / Set State nodes work with all three types.

### State Node

A read/write object where you specify fields (and/or enable arbitrary location access).

- **Initial value:** Connect the main in-binding to set initial value
- **Important:** Subsequent changes to the main in-binding are **ignored** — use Set State methods to update
- Only specified fields can be written to via the green navbar menu

### Set State

**Static locations:** Update specified values in state. Available on value out-bindings of actions, in-params, and components.

**Dynamic locations:** The Set State action node enables setting values at dynamically configured fields or paths. A path is an array of field names/indexes, e.g. `["output", "name"]` sets
`.output.name` within the state.

**How to use:**

1. Add "Set State" node: `+ NEW` → `Actions` → `SET STATE`
2. Select target state cell: "Screen State" or a named state node
3. Set or delete values at a top-level "Field" or at a "Path" in the cell

### Get State

**Static locations:** Analogue of Set State but for in-bindings.

**Dynamic locations:** If no field is set, Get State acts as a preview of the current state value.

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

---

## Decisions & State Machines

> Source: [Decisions & State Machines](https://www.notion.so/1051d464528f80a48689d2743814b1af)
> Note: Page content was truncated on fetch — re-fetch for full detail.

---

## Functions & Code Modules

> Source: [Functions & Code Modules](https://www.notion.so/8c02beb038ec40fea16762bbd8aabfe8)
> Note: Page content was truncated on fetch — re-fetch for full detail.

---

## Actions

> Source: [Actions](https://www.notion.so/1051d464528f80c9b498daa44df9b685)
> Note: Main page content was truncated on fetch. Sub-pages fully captured below.

### File Register

Used as part of a flow to upload files to a remote mixer. Visual implementation of `file.register`.

- Uses the term `location` instead of `app_designator` (unlike the underlying stdlib API)
- **Best practice:** Do registration from within a **service agent** (using Main or Other DB) — Admin (Remote) registrations require builder privileges
- Reference: [`file` stdlib module](https://www.notion.so/file-1061d464528f8136b486db76ee013016#1061d464528f811cb892c2db1b25a6a5)

---

## Agent Connect Tiles

> Source: [Agent Connect tiles](https://www.notion.so/12f1d464528f80a4a372e439a6d074f4)

Agents are modules called with parameters that return values. Three types:

| Type              | Description                                                                              | Setup                                                  |
|-------------------|------------------------------------------------------------------------------------------|--------------------------------------------------------|
| **Service Agent** | Deployed on a cloud server                                                               | Enter Cloud URL → Refresh → select workspace/app/agent |
| **Local Agent**   | Runs on same server as caller (cloud server if from cloud agent; Amp if from Amp screen) | No URL needed                                          |
| **Amp Agent**     | Rarely used; targets a specific Amp server                                               | Provide `ampPrefix` (URL to Amp server)                |

**Refresh button** — updates agent bindings when the agent's signature changes or when pointing to a different agent. In/out-params are mirrored on the action node.

**Client Side toggle** — runs the agent call from the client rather than the VM. Useful when: calling an MCP on `localhost`, or when the caller needs to access local resources.

---

## Api (Action Node)

> Source: [Api](https://www.notion.so/1571d464528f80e9a9fee7fbbf80a0c5)

Performs HTTP requests to servers. Opinionated (most API requests simple; special cases need Mix).

### URL Templates

Use `{...}` syntax in the URL for dynamic segments. *(Section truncated on fetch — re-fetch for full detail.)*

### Token

Turn on the token binding but leave it **unconnected** to use the current user's Remix token.

### Content-Type: multipart/form-data

Set the `Content-Type` header to `multipart/form-data` and pass an object as the body — encoded automatically.

- **VM side**: object values must be `String`
- **Client-side**: can include `Local File` types from an upload widget
- Manual fallback example: `https://remix-dev.remixlabs.com/e/edit/builder-examples/multipart_manual`

### Content-Type: application/x-www-form-urlencoded

Set the `Content-Type` header accordingly and pass an object as the body — encoded automatically. Also handles nested fields.

### Client-Side Operation

Runs the API request from the **client** (browser/app) rather than the VM server. Useful when:

- Calling `localhost` (e.g. local MCP server)
- Uploading local files — client does the upload directly without passing data into VM (significant performance benefit). Use together with
  the [File Upload pattern](https://www.notion.so/12e1d464528f8088900dcf2f4154cc24).

### Params

`outputEscEnabled=true` — see [referenced page](https://www.notion.so/92c2ae9e2068459a9af3178525a8a906) (unfetched).

---

## Database Actions: Save and Delete

> Source: [Database Actions: Save and Delete](https://www.notion.so/1151d464528f8047b20fd8809e2bb766)

### Create and Update

*(Placeholder — "Text to add". No content yet on page.)*

### Delete

Requires: **DB location** + **ref** of the record to delete.

*(Note: A previous version of this page mentioned error bindings added in Oct 2024, but that text was struck through — current status unclear. Re-fetch for latest.)*

---

## Annotations

> Source: [Annotations](https://www.notion.so/1051d464528f8018a17fd69389243631)
> Blank page — no content.

---

## Service Agent Modules

> Source: [Service Agent modules](https://www.notion.so/22d1d464528f80278259e3f6ee68dcd1)
> Parent: Remix Studio User Guide
> See also: [Agent Connect tiles](https://www.notion.so/12f1d464528f80a4a372e439a6d074f4)

Service Agent modules have an additional **"Edit Cloud Server"** button in the Navbar. Clicking it lets you see the details of a Cloud Server that has data on it — all queries, save actions, and delete actions are then run against that server. This makes building new agents against a remote dataset much easier.

**Important:** Choosing a Cloud Server here does **not** change the generated code — all nodes will still point to the workspace where they are installed (not the preview server).
