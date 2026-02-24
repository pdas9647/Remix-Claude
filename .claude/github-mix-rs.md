# mix-rs

**Repository:** https://github.com/remixlabs/mix-rs
**Stack:** Rust (primary), TypeScript/Vite (desktop frontend), C (FFI/wasm2c), Java/JNI, Swift, Kotlin (mobile bindings), Docker
**Purpose:** Rust runtime stack for the Remix platform: value system, query engine, VM FFI bridge, Wasm runtime, agent server CLI, and Tauri desktop app
**Explored layers:** L1-L5 (~48 files, ~10500 lines)

## Build & Deploy

- **Workspace**: Cargo workspace (resolver v2) with 10 members, Rust 2021 edition
- **Release profile**: `opt-level = "z"`, strip, LTO; separate `mixrun-release-widgets` profile with `panic = "abort"` + `codegen-units = 1`
- **Component manager**: `rcm` (Remix Component Manager) for setup, deps, artifacts, uploads
- **CI**: CircleCI — Linux amd64/arm64, macOS arm64, iOS, Android, Windows, WASI
    - sccache for build caching; artifacts to `gs://remix-dev/artifacts/<component>/<build_tag>/...`
    - Docker images to `us-west1-docker.pkg.dev/rmx-artifacts/agent-server/agent-server`
    - `update-users` job triggers downstream consumers on `main` merges
- **Docker**: Debian trixie-slim, port 8000, agent timeout 60s (max 300s), postpone timeout 60s (max 3600s)
- **Desktop releases**: Tauri builds signed with Apple cert, channels (dev, branch-specific)
- **Snowflake SPCS**: Separate slim Docker image (`mixer/server-snowflake/`)

### Feature Flag Architecture (mixer)

Key composite features: `all` (full native), `agent_server` (cloud deploy), `snowflake_server` (SPCS), `wasi` (WASI target), `test`. FFI selections: `all_ffis` (includes DuckDB), `some_ffis` (
contained FFIs only), `wasi_ffis`. Backend: `mixrun_all_backends` = wasm2c + wasmtime.

## Architecture

### Crate Map

| Crate         | Path           | Role                                                                                                                     |
|---------------|----------------|--------------------------------------------------------------------------------------------------------------------------|
| `remix`       | `remix/`       | Core types: `Value` enum, codec (ser/de to u32 buffers), builder metadata, proc macros (`#[remix::bind]`), stdlib        |
| `remix_macro` | `remix_macro/` | Proc macros: `#[bind]` (export Rust fns to Wasm), `#[derive(HasType)]` (builder type decls)                              |
| `mixfs`       | `mixfs/`       | Filesystem abstraction: compile-time backend selection (`backend_std`/`backend_tokio`/`backend_noop`)                    |
| `mixquery`    | `mixquery/`    | Embedded query engine: AST, parser, execution (filter/proj/aggr), indexed storage, varcode, write-ahead log              |
| `mixcore`     | `mixcore/`     | FFI bridge: dispatches calls (DB, HTTP, crypto, files, DuckDB, Wasm, WebSocket, MQTT, secrets) between Mix VM and native |
| `mixrun`      | `mixrun/`      | Wasm runtime host: mixcore + MixRT (C VM), backends (wasmtime, wasm2c), widget lifecycle, LRU cache, C/JNI API           |
| `mixer`       | `mixer/`       | CLI & server: subcommands `serve`, `run`, `mcp`, `db`, `wasm`, `codec`, `error-decode`                                   |
| `desktop/*`   | `desktop/`     | Tauri v2 desktop app (4 sub-crates: `src-tauri`, `src-shared`, `src-aux`, `src-mcp-bridge`)                              |

### Runtime Modes

| Component | Application          | Target | Host env     | HTTP FFI     |
|-----------|----------------------|--------|--------------|--------------|
| mixcore   | Web/Desktop appclip  | Wasm   | JS groovebox | JS groovebox |
| mixcore   | iOS appclip          | native | JS groovebox | JS groovebox |
| mixrun    | iOS/Android widgets  | native | mixrun       | Rust         |
| mixer     | Agent server (serve) | native | mixrun       | Rust         |
| mixer     | Slack plugin (run)   | Wasm   | mixrun       | JS deno      |

## API Surface / Entry Points

### `mixer` CLI (clap)

`serve`, `run`, `mcp`, `db`, `wasm`, `error-decode`, `codec` (hidden)

### Server HTTP API

**Framework**: Rocket with CORS, optional TLS, Swagger at `/v1/swagger/`

**v0** (legacy, deprecated Feb 2026): workspace CRUD, run-agent, install .remix, export/import, SSE, query, permissions, remixgen.

**v1** (resource-oriented, JSON responses): `/v1/ws/<ws>` (CRUD), `/v1/ws/<ws>/app/<app>` (management), `/v1/ws/<ws>/app/<app>/agent/<agent>` (run), `/v1/ws/<ws>/app/<app>/documents` (CRUD),
`/v1/ws/<ws>/app/<app>/query/<name>`, `/v1/ws/<ws>/app/<app>/save`, `/v1/ws/<ws>/auth-configs`, `/v1/ws/<ws>/permissions`, `/v1/ext/<ext>/<path..>` (desktop), `/v1/auth/spcs` (Snowflake).

### C/FFI & Mobile Bindings

- C headers via cbindgen: `mixcore.h`, `mixrun.h`, `mixrun-decls.h`
- Mobile: Swift (`MixCore.swift`, `MixRun.swift`), Java/JNI, Kotlin (`MixRun.kt`)
- JavaScript: `mixcore/bindings/javascript/` (tauri, wasm, webkit variants)

### MCP Bridge

- Queries `_rmx_desktop/list_mcp_tools` for tool discovery
- Forwards tool calls → `run-agent` → Mix agent execution
- Stdio transport; desktop auto-configures `claude_desktop_config.json`

## Core Modules

### Value System (`remix/src/value.rs`)

Dynamic `Value` enum: `Undef`, `Null`, `Bool`, `Int64`, `Number(Number)`, `String`, `Ref(Ref)` (DB record ID, `{ref}` prefix), `Binary`, `Array(Vec<Value>)`, `Map(BTreeMap<String, Value>)`, `Token` (
opaque), `Case(Box<Case>)` (tagged union for option/result/enum).

JSON uses tagged objects: `{"_rmx_type": "{tag}...", "_rmx_value": ...}`

### FFI System (`mixcore/src/ffi.rs`)

- **Registration**: `FFIs` is a `HashMap<String, RawFFI>` with `Sync`/`Async` variants
- **Two FFI sets**: `GROOVEBOX_FFIS` (db, files, token, duckdb, secrets, decode_error) and `NON_GROOVEBOX_FFIS` (core, http, mqtt, crypto, wasm, websocket)
- **Type-safe dispatch**: Macro-generated implementations for 1-9 arg tuples, auto-deserializing from u32 codec buffers
- **Registration API**: `register_sync`, `register_async`, `register_simple`, `register_const`, `register_raw`, `register_unsupported`

### FFI State (`mixcore/src/ffi_state.rs`)

`State` struct: FFIs, env, actions queue, proxies, debug symbols, output, timeouts, HTTP interceptors, DB seekers, S3, MQTT publisher, DuckDB connection, temp files, managed type map (extensible:
`BlockUnsafeHttp`, user details, signals).

### Action System (`mixcore/src/action.rs`)

`Action` enum: `Invoke` (VM entry call), `PerformHostEffect`, `OpenLink`, `LocationCapture`. Actions dispatch to either `Invoke` params (forwarded to VM `invoke3` entry) or `ClientAction` (executed by
host). Actions support chaining via `action: Option<Box<Action>>`.

### Mixrun VM Loop (`mixrun/src/mixrun.rs`)

State machine: `InitDone` → `Running` → `Idle`/`FFICall`/`Panic`/`Exit`/`Timeout`. On `FFICall`, reads name+args from VM memory, dispatches via `core.ffi_dispatch_defer()`, writes result back. Async
FFIs deferred to tokio, awaited during idle. Actions: checks deferred FFI results, then pops from queue. Entry via `enter()` (tag/modname lookup). Pipeline: `.load(&codes)` → `.link()` →
`.init(env)` → `.run(name, params)` with timeout.

### Agent Caching (`mixer/src/serve/run_agent.rs`)

- Agents cached as memory snapshots: `CachedCode::Qcode { mem: Vec<u8> }` or `CachedCode::Wasm { items, mem }`
- Restore = reinitialize runtime + restore memory + new random seed (no recompilation)
- Cloud queries also cached (`Cached::Query`)
- Cache is workspace × app × agent keyed, invalidated on .remix install

### Widget System (`mixrun/src/widgets.rs`)

iOS/Android: `execute_agent()` creates full runtime, runs agent, returns JSON. Widget records: name, sizes, database, tags. C FFI boundary: `convert_ffi()` → `"ok:..."` or `"error:{json}"`. Errors
include raw backtraces for appclip error reporting.

### HTTP FFI (`mixcore/src/ffi/http.rs`)

`$http_do`/`$http_do_server` (async). `MixRequest`: method, URL, headers, params, body, OAuth/basic auth, timeout, media types. Interceptors (`Intercept` trait) route requests internally (e.g.,
cross-agent calls). `BlockUnsafeHttp` blocks non-HTTPS/localhost/private-IP on agent server. Auto-detects charset (UTF-8 → Latin-1 fallback).

### DB FFI (`mixcore/src/ffi/db.rs`)

~17 registered FFIs: `$db_get_one`, `$db_save_one/a/ar`, `$db_upsert_a/ar`, `$db_delete_one/a/ar`, `$db_execute_next`, `$db_getWatermark`, `db_flush`, `$db_parse`, `$db_fromAST_result`. Query
iteration via `QueryIter` proxy with 100-record batches.

### Query Engine (`mixquery/`)

- **Query**: Builder pattern with `QueryElement` pipeline (IndexPredicate, Projection, Aggregate)
- **Predicates**: `Equals`, `Not`, `And`, `Or`, prefix, range, exists — prefilter excludes `_rmx_deleted`/`_rmx_superseded` by default
- **API**: CRUD operations (`get_one`, `save`, `upsert`, `delete`, `flush`, `peek_writelog`, `get_watermark`, `admin_modify_ss_dl`)
- **Varcode**: Variable-length integer encoding (7 bits per byte, MSB continuation flag) for compact storage
- **Records**: `BTreeMap<String, Value>`, IDs are 32-char alphanumeric strings

### Data Storage (`mixquery/src/data/store.rs`)

File-based record store with two files per database:

- `db.off` — offset index: array of `u64` LE positions (4-byte header + rowid×8 bytes)
- `db.dat` — data file: varcode-length-prefixed MessagePack records
- Records serialized via `rmp-serde` (MessagePack), deserialized through `remix::Value` intermediate
- Append-only writes; compaction via `write_set_to()` using `RoaringBitmap` row selection
- Each record gets `_rmx_row` field (sequential rowid) on write

### Environment (`mixcore/src/env.rs`)

Builder for Mix runtime env: user, app, surface, token, workspace, org, `hostEnvironment` (AgentServer/Builder/Server/Web/Widget), timezone, platform (Web/Desktop/Mobile), OS, URLs, request
headers/body, origin secret.

### Auth (`mixer/src/serve/auth.rs`)

JWT via JWKS (remote) + locally-signed (HS256, random key/process, 48h). User: authenticated, anonymous, or no-token.

### Desktop App (`desktop/src-tauri/src/app.rs`)

Tauri v2 with plugins: process, fs, opener, http, dialog, deep-link, clipboard, updater, single-instance, web-auth (mobile). Setup flow: logging → tray icon → Claude config → deep links → auth →
workspace apps install → home screen. Tauri commands: env, user_info, workspace, mixcore FFI dispatch, builder/runtime/instant open, updater, auth. Custom URI protocol for mixer requests.

**Desktop config**: TOML at `<data_dir>/com.remixlabs.desktop/config.toml` (user token, workspace, release info, app versions). Workspaces at `<data_dir>/workspaces/`. Default workspace `"local"`,
local mixer port 2025.

**Claude integration**: On startup writes MCP bridge to `Claude/claude_desktop_config.json` as `remix-app` server. MCP bridge binary ensures desktop app running (version-checks via
`/v1/ext/remix-app/release-info`), starts it if needed, then delegates to `mixer::mcp::run_async()`.

## Key Patterns & Conventions

- **Feature flags**: Extensive conditional compilation; mixer alone has ~20 features composing FFIs, backends, and deployment targets
- **Error handling**: Per-crate `Error`/`Result`; `anyhow` at binary boundaries; `thiserror` in libraries
- **Async**: Tokio throughout; FFIs async on native, sync on Wasm; `spawn_blocking` for slow drops (wasmtime)
- **Serialization**: serde everywhere; custom `Value` ser/de; binary u32 codec for Wasm boundary
- **Workspace multi-tenancy**: `ws_dir/<workspace>/` filesystem layout; `Reg` → `Seeker` for DB access
- **Logging**: `tracing` + `RUST_LOG`; JSON or pretty; VM logs on `mix-vm` target; `MIXRT_DEBUG` flags
- **Agent lifecycle**: Load from DB → compile → cache snapshot → restore on next call (skip recompile)

### Server Args (`mixer/src/serve/args.rs`)

Key flags: `--ws-dir`, `--host`, `--port` (8000), `--jwks-url`, `--mixrt-wasm`, `--agent-cache-size`, `--body-limit` (32MiB), `--params-limit` (1MiB), `--tls-certs/key`, `--serve-files-from`,
`--env` (JSON), `--dangerous-http-mode`, `--remote-compile-server`, `--workspace-creator`, `--base-url`.

## Data Model

- **Workspace**: Filesystem directory under `ws_dir/`, contains app DBs
- **App DB**: Directory with `v2/` subdirectory containing `name.txt`, `db.off`, `db.dat`, plus index files
- **Record**: `BTreeMap<String, Value>` with reserved fields: `_rmx_id` (Ref), `_rmx_type` (string), `_rmx_row` (u64), `_rmx_deleted` (bool), `_rmx_superseded` (bool)
- **Storage format**: MessagePack serialized records in append-only `db.dat`, offset index in `db.off`, varcode length prefixes
- **Indexing**: RoaringBitmap-based field indexes for `contains`, `exact_match`, `prefix`, `range`, `scan` operations
- **Write-ahead log**: `writelog.rs` tracks pending writes before compaction

## Open Questions

- **Compact/index internals**: How `compact.rs`/`index.rs`/`index_1.rs` build and maintain field indexes from the append-only store
- **DuckDB integration**: FFI registration details in `ffi/duckdb.rs`, connection lifecycle, query bridge
- **Wasm2c vs wasmtime selection**: `CodeType::decide()` logic, when each backend is preferred
- **Codec binary format**: Exact u32 encoding scheme for array headers, tagged types, nested values across Wasm boundary