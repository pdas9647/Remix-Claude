# Remix Agent Server — Live API Endpoints (Swagger)

Sourced from: `https://agt.remixlabs.com/v1/_doc/openapi.json`  
Spec version: 0.1.0 (OAS 3.1) — last fetched 2026-05-06

**Auth:** Bearer JWT required on all endpoints. JWT expires ~30 days.  
Re-fetch spec: `curl -s -H "Authorization: Bearer <token>" "https://agt.remixlabs.com/v1/_doc/openapi.json"`

---

## root

| Method | Path | Description                                |
|--------|------|--------------------------------------------|
| GET    | `/`  | Health check — returns build number string |

---

## v0 (Legacy — still active)

Many v0 endpoints come in two flavors: `{ws_form}` (multipart/form-data) and `{ws_json}` (application/json) — same operation, different content type.

| Method | Path                               | Description                                                                                                                                 |
|--------|------------------------------------|---------------------------------------------------------------------------------------------------------------------------------------------|
| POST   | `/v0/count-activities/{ws_form}`   | Count activities (form)                                                                                                                     |
| POST   | `/v0/count-activities/{ws_json}`   | Count activities (JSON)                                                                                                                     |
| POST   | `/v0/flush/{ws}`                   | Flush (hot-reload) all apps in a workspace                                                                                                  |
| POST   | `/v0/grant-permission/{ws_form}`   | Grant permission to a user/agent (form)                                                                                                     |
| POST   | `/v0/grant-permission/{ws_json}`   | Grant permission to a user/agent (JSON)                                                                                                     |
| POST   | `/v0/install-remix-file/{ws_file}` | Install a `.remix` file by uploading binary (`octet-stream`). Query (all optional): `force` (bool), `platform` (string), `surface` (string) |
| POST   | `/v0/install-remix-file/{ws_json}` | Install a `.remix` file by JSON descriptor (`InstallJson` — describes where to find the file remotely)                                      |
| POST   | `/v0/query/{ws}/{app}`             | Create a registered (persistent) query on an app (`CloudQuery` body)                                                                        |
| DELETE | `/v0/query/{ws}/{app}/{name}`      | Delete a registered query by name                                                                                                           |
| POST   | `/v0/remixgen/{ws}/{app}`          | Remixgen — trigger code generation for an app (`GenerateOptions` body)                                                                      |
| GET    | `/v0/run-agent/{ws}/{app}/{agent}` | Run an agent via GET with URL query params. Also: `/run-agent/{ws}/{app}/agents/{agent}?<args>`                                             |
| POST   | `/v0/run-agent/{ws}/{app}/{agent}` | Run an agent via POST with JSON body. Optional query: `include_raw_body` (bool), `timeout` (int64 ms)                                       |
| GET    | `/v0/subscribe/{topic}`            | Subscribe to an SSE topic — response is an event stream of JSON values                                                                      |
| GET    | `/v0/ws`                           | List all workspaces the caller has access to                                                                                                |
| POST   | `/v0/ws`                           | Create a workspace (`CreateWorkspace` — JSON or form-encoded, e.g. `name=my-ws`)                                                            |
| GET    | `/v0/ws/{ws}`                      | Describe a single workspace                                                                                                                 |
| DELETE | `/v0/ws/{ws}`                      | Delete a workspace                                                                                                                          |
| DELETE | `/v0/ws/{ws}/{app}`                | Delete a specific app from a workspace                                                                                                      |
| POST   | `/v0/ws-clone/{ws}`                | Clone workspace to another local or remote agent server. Query: `dest_ws`, `url`, `token`                                                   |
| GET    | `/v0/ws-export/{ws}`               | Export a full workspace as a mastertape binary (`octet-stream`)                                                                             |
| POST   | `/v0/ws-import`                    | Import a workspace from a mastertape binary. Query: `ws` (identifier)                                                                       |

---

## v1 (Current — prefer over v0 equivalents where available)

### Workspaces & Apps

| Method | Path                                 | Description                                                                                                      |
|--------|--------------------------------------|------------------------------------------------------------------------------------------------------------------|
| GET    | `/v1/ws`                             | List all workspaces (`WorkspacesInfo`)                                                                           |
| GET    | `/v1/ws/{ws}`                        | Get info about a workspace (`WorkspaceInfo`)                                                                     |
| POST   | `/v1/ws-import`                      | Import workspace as `.remix` file. Optional query: `ws`. Body: JSON (`WorkspaceImportPayload`) or `octet-stream` |
| GET    | `/v1/ws/{ws}/app/{app}`              | Get app info → `AppInfo[]`                                                                                       |
| POST   | `/v1/ws/{ws}/app/{app}`              | Create an app with metadata (`APIAppMetadata`). Error if app already exists.                                     |
| DELETE | `/v1/ws/{ws}/app/{app}`              | Delete an app (204)                                                                                              |
| GET    | `/v1/ws/{ws}/app/{app}/appMeta.json` | Get app metadata — amp-compat path (`APIAppMetadata`)                                                            |

### Agents & Permissions

| Method | Path                                                   | Description                                        |
|--------|--------------------------------------------------------|----------------------------------------------------|
| GET    | `/v1/ws/{ws}/app/{app}/agent/{agent}`                  | Get agent metadata (`AgentInfo`)                   |
| GET    | `/v1/ws/{ws}/app/{app}/agent/{agent}/permissions`      | List all permissions for an agent → `APIGrant[]`   |
| GET    | `/v1/ws/{ws}/app/{app}/agent/{agent}/permissions/{id}` | Get a specific permission grant by ID (`APIGrant`) |
| DELETE | `/v1/ws/{ws}/app/{app}/agent/{agent}/permissions/{id}` | Delete a permission grant by ID (204)              |

### Auth Configs

| Method | Path                       | Description                                                                               |
|--------|----------------------------|-------------------------------------------------------------------------------------------|
| GET    | `/v1/ws/{ws}/auth-configs` | Get auth config by name (query: `name`) or list all → `APIAuthConfig` / `APIAuthConfig[]` |

### Documents (Mix DB records)

| Method | Path                                    | Description                                                                          |
|--------|-----------------------------------------|--------------------------------------------------------------------------------------|
| POST   | `/v1/ws/{ws}/app/{app}/documents`       | Save one or more documents. Body: `MixMap[]`. Returns newline-joined ID string(s).   |
| GET    | `/v1/ws/{ws}/app/{app}/documents/{id}`  | Get a document by ID (`MixMap`). Returns 410 if the record was deleted.              |
| DELETE | `/v1/ws/{ws}/app/{app}/documents/{id}`  | Delete a document by ID (204)                                                        |
| DELETE | `/v1/ws/{ws_query}/app/{app}/documents` | Batch delete — `ids` as query param (URL-encoded JSON array, e.g. `%5b%22abc%22%5d`) |
| DELETE | `/v1/ws/{ws_body}/app/{app}/documents`  | Batch delete — `ids` as form body (URL-encoded JSON array)                           |

### File Manager

| Method | Path                                         | Description                                                                                   |
|--------|----------------------------------------------|-----------------------------------------------------------------------------------------------|
| GET    | `/v1/ws/{ws}/app/{app}/file-manager/content` | Get file content or list directory at `path` (query). Returns `octet-stream`.                 |
| PUT    | `/v1/ws/{ws}/app/{app}/file-manager/content` | Upload a file (`octet-stream`). Query: `path`, `force` (overwrite). 200=updated, 201=created. |
| POST   | `/v1/ws/{ws}/app/{app}/file-manager/content` | Upload a file via multipart form (`CreateFile` schema). 200=updated, 201=created.             |
| DELETE | `/v1/ws/{ws}/app/{app}/file-manager/content` | Delete file at `path` (query)                                                                 |
| POST   | `/v1/ws/{ws}/app/{app}/file-manager/copy`    | Copy a file. Query: `src` (source path), `src_app` (source app), `path` (destination path)    |
| GET    | `/v1/ws/{ws}/app/{app}/file-manager/props`   | Stat a file — get `FileInfo` metadata at `path` (query)                                       |

### Signals (async coordination)

Signals are used for async wait/notify across agents or between client and server.

| Method | Path               | Description                                                      |
|--------|--------------------|------------------------------------------------------------------|
| POST   | `/v1/signals`      | Create a new signal → `MixSignal` (contains the signal ID)       |
| GET    | `/v1/signals/{id}` | Wait (long-poll) for a signal to be sent → `MixValue`            |
| POST   | `/v1/signals/{id}` | Send a value to a signal (`MixValue` body) — unblocks any waiter |
| DELETE | `/v1/signals/{id}` | Cancel a signal                                                  |

---

## Key identifiers

- **`{ws}`** — workspace identifier (`Ident`), e.g. `sEt2qxPydL` (Funda), `iaEj4QYboi` (Lumber)
- **`{app}`** — app name within a workspace, e.g. `dbp-db`, `lumber_agents`
- **`{agent}`** — agent name within an app, e.g. `snowflake_cortex_generate_sql_service_acct`, `snowflake_generate_ddl_service_acct`
- **`MixMap`** — a JSON object (arbitrary key-value, the Mix DB record format)
- **`MixValue`** — any Mix value (JSON-serialized)

---

## Asset / Component Search (undocumented but stable)

Each library workspace exposes an internal `_rmx_search` app with a `get_assets_light` agent that returns the full asset catalog. This is what the Studio Library Browser uses internally — there's no
XHR call in the Network tab because it likely arrives via WebSocket/MQTT during sync, but the HTTP endpoint still works for direct queries.

**Endpoint:** `GET /v0/run-agent/{library_ws}/_rmx_search/get_assets_light?inputEscEnabled=true&outputEscEnabled=true`  
**Body:** `{}` (POST also works)  
**Auth:** Bearer JWT

**Known library workspaces** (`{library_ws}` values):

- `remix_labs` (~355 assets) — primary Remix-built library
- `remix-libraries` (~240 assets)
- `remix-dbp` (~148 assets) — DBP components, "data visualiser" collection lives here
- `com_remixlabs_john` (~112), `com_remixlabs_wilber` (~110), `com_remixlabs_padmanabha` (~6) — per-user libraries
- `iaEj4QYboi` (~26) — Lumber

**Response shape:** `{"assets": [...]}` where each asset has:

- `name`, `displayType` (Component / Card / Action / Function / WebComp / cloud_agent / screen / widget_agent / Wasm / Data / code / agent)
- `collection` (UI kit, snowflake, ai, util, zendesk, sign in, duckdb, salesforce, stats & charts, webcomps, oauth, data visualiser, etc.)
- `assetType` (e.g. `L2`)
- `metadata.tags[]`, `metadata.description`, `metadata.instructions`
- `bdSignSet[]` — bindings list. Each entry: `{name, direction, type}`. **`direction: "True"` = InParam, `"False"` = OutParam** (counter-intuitive but verified)
- `_rmx_id._rmx_value` — the asset ID used in share URLs
- `_rmx_created_by` / `_rmx_last_modified_by` / `_rmx_last_modified_at` — sometimes empty for older assets; Studio reads author from a separate "source info" record
- `clip` — base64-encoded Mix AST (the copy-paste payload)
- `preview` — base64-encoded visual preview (see decoding below)

### Preview decoding

The `preview` field is **base64 → raw zlib deflate (wbits=-15) → JSON**. Two top-level shapes:

- `{"type": "dom", "sg": <DOM tree>}` — visual component. Walk `sg`:
    - `_rmx_type`: e.g. `dom_group`, `dom_text`, `dom_icon`, `dom_horizontalContainer`, `dom_verticalContainer`, custom web comp tags like `rmx-tui-grid`
    - `display`: human-readable label
    - `props.style._rmx_global[]`: class names (Tailwind for `remix_labs`; "Default class set" naming for `remix-dbp` like `"Default - Text"`, `"Heading or Title"`, `"White / Inverse Text Color"`)
    - `props.style.color` / `props.style.backgroundColor`: actual hex codes (e.g. `#0F243F`, `#ffffff`) — present even when class names are abstract
    - `props.value` / `props.text` / `props.placeholder`: text content
    - `props.name` (for `dom_icon`): the specific icon name (e.g. `chevron-triple-up`)
    - `children[]`: nested nodes
- `{"type": "json", "data": ...}` — for non-visual assets (Functions, etc.); `data` may be `null`

Python decode snippet:

```python
import base64, zlib, json
parsed = json.loads(zlib.decompress(base64.b64decode(asset['preview'] + '=='), -15))
```

### Share link URL format

Deterministic — built from workspace + asset ID:

```
https://remix.app/remix/asset?source=https://agt.remixlabs.com/ws/{library_ws}/{_rmx_id._rmx_value}
```

Example:  
`https://remix.app/remix/asset?source=https://agt.remixlabs.com/ws/remix-dbp/eZZaob1ml0icoA6RK8XN2vMjxG86iVMy`

The page renders a thumbnail and a "Copy to Clipboard" button so the link can be pasted into a Studio screen to drop the asset in.

### Recommended workflow (for "find best asset for X")

1. Fetch `get_assets_light` for the relevant library workspace (default: `remix_labs`; ask user if unsure).
2. Filter by `name` + `metadata.tags` + `metadata.description` + `collection` against the user's purpose.
3. For top candidates, decode `preview` and traverse the DOM to describe layout, colors, text, icons.
4. Output: name + displayType + collection + tags + in-params + out-params + brief visual description + share link.
