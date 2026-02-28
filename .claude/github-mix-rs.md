# mix-rs

**Repository:** https://github.com/remixlabs/mix-rs
**Stack:** Rust, TypeScript/Vite (desktop frontend), C (FFI/wasm2c), Java/JNI, Swift, Kotlin
**Purpose:** Rust runtime stack — value system, query engine, VM FFI bridge, Wasm runtime, agent server CLI, Tauri desktop app
**Explored:** L1–L5 (~48 files, ~10500 lines)

## Build & Deploy

- **Cargo workspace**: resolver v2, 10 members, Rust 2021. Release: `opt-level="z"`, strip, LTO. Widgets profile: `panic="abort"`, `codegen-units=1`.
- **CI**: CircleCI — Linux amd64/arm64, macOS arm64, iOS, Android, Windows, WASI. sccache; artifacts → `gs://remix-dev/artifacts/`. `update-users` job triggers downstream on `main` merge.
- **Docker**: Debian trixie-slim, port 8000, agent timeout 60s (max 300s). SPCS: separate slim image (`mixer/server-snowflake/`).
- **Desktop**: Tauri v2, Apple-signed, release channels (dev, branch-specific). `rcm` manages deps/artifacts/uploads.
- **Feature flags**: `all` (full native), `agent_server`, `snowflake_server`, `wasi`, `test`; FFIs: `all_ffis` (+ DuckDB), `some_ffis`, `wasi_ffis`; backends: `mixrun_all_backends` = wasm2c +
  wasmtime.

## Architecture

### Crate Map

| Crate         | Role                                                                                |
|---------------|-------------------------------------------------------------------------------------|
| `remix`       | Core `Value` enum, u32 codec, builder metadata, `#[remix::bind]` proc macro         |
| `remix_macro` | `#[bind]` (export Rust→Wasm), `#[derive(HasType)]`                                  |
| `mixfs`       | FS abstraction: `backend_std`/`backend_tokio`/`backend_noop`                        |
| `mixquery`    | Query engine: AST, parser, filter/proj/aggr, indexed storage, varcode, WAL          |
| `mixcore`     | FFI bridge: DB, HTTP, crypto, files, DuckDB, Wasm, WebSocket, MQTT, secrets         |
| `mixrun`      | Wasm runtime host: mixcore + MixRT (C VM), wasmtime/wasm2c, LRU cache, C/JNI API    |
| `mixer`       | CLI (`serve`, `run`, `mcp`, `db`, `wasm`, `error-decode`); Rocket HTTP server       |
| `desktop/*`   | Tauri v2 app (4 sub-crates: `src-tauri`, `src-shared`, `src-aux`, `src-mcp-bridge`) |

### Runtime Modes

| Component | Application                      | Target      | Host env     |
|-----------|----------------------------------|-------------|--------------|
| mixcore   | Web/Desktop appclip, iOS appclip | Wasm/native | JS groovebox |
| mixrun    | iOS/Android widgets              | native      | Rust         |
| mixer     | Agent server (`serve`)           | native      | Rust         |
| mixer     | Slack plugin (`run`)             | Wasm        | JS deno      |

## API Surface

- **CLI**: `serve`, `run`, `mcp`, `db`, `wasm`, `error-decode`, `codec` (hidden)
- **HTTP**: Rocket + CORS + optional TLS. Swagger at `/v1/swagger/`.
    - v0 (deprecated Feb 2026): workspace CRUD, run-agent, install .remix, SSE, query, permissions, remixgen
    - v1: `/v1/ws/<ws>`, `/v1/ws/<ws>/app/<app>`, `.../agent/<agent>`, `.../documents`, `.../query/<name>`, `.../save`, `.../auth-configs`, `.../permissions`, `/v1/ext/<ext>/<path..>`, `/v1/auth/spcs`
- **C/FFI**: cbindgen headers (`mixcore.h`, `mixrun.h`); Swift, Java/JNI, Kotlin mobile bindings; JS bindings (tauri, wasm, webkit)
- **MCP Bridge**: queries `_rmx_desktop/list_mcp_tools`, forwards calls → `run-agent` → Mix agent. Stdio transport. Auto-writes `claude_desktop_config.json`.
- **Server args**: `--ws-dir`, `--host`, `--port` (8000), `--jwks-url`, `--body-limit` (32MiB), `--params-limit` (1MiB), `--tls-certs/key`, `--dangerous-http-mode`, `--base-url`,
  `--remote-compile-server`

## Core Modules

- **Value** (`remix/src/value.rs`): `Undef`, `Null`, `Bool`, `Int64`, `Number`, `String`, `Ref` (DB ID, `{ref}` prefix), `Binary`, `Array`, `Map`, `Token`, `Case` (tagged union). JSON:
  `{"_rmx_type":..,"_rmx_value":..}`.
- **FFI** (`mixcore/src/ffi.rs`): `FFIs` = `HashMap<String, RawFFI>` (Sync/Async). Two sets: `GROOVEBOX_FFIS` (db, files, token, duckdb, secrets) and `NON_GROOVEBOX_FFIS` (core, http, mqtt, crypto,
  wasm, websocket). Macro-generated dispatch for 1–9 arg tuples.
- **FFI State** (`ffi_state.rs`): `State` holds FFIs, env, actions queue, HTTP interceptors, DB seekers, MQTT publisher, DuckDB connection, S3, managed type map.
- **Action System** (`action.rs`): `Invoke` | `PerformHostEffect` | `OpenLink` | `LocationCapture`. Chainable via `action: Option<Box<Action>>`.
- **VM Loop** (`mixrun/src/mixrun.rs`): State machine `InitDone→Running→Idle/FFICall/Panic/Exit/Timeout`. On `FFICall`: read name+args → `ffi_dispatch_defer()` → write result. Pipeline:
  `.load()→.link()→.init(env)→.run(name, params)`.
- **Agent Caching** (`run_agent.rs`): Snapshot = `CachedCode::Qcode{mem}` or `CachedCode::Wasm{items,mem}`. Restore = reinit + restore mem + new seed (no recompile). Keyed: workspace × app × agent.
- **Widget System** (`mixrun/src/widgets.rs`): `execute_agent()` → JSON. C boundary: `"ok:..."` or `"error:{json}"`.
- **HTTP FFI** (`ffi/http.rs`): `$http_do`/`$http_do_server`. `MixRequest`: method, URL, headers, OAuth/basic, timeout. `BlockUnsafeHttp` guards non-HTTPS/localhost on agent server.
- **DB FFI** (`ffi/db.rs`): ~17 FFIs — `$db_get_one`, `$db_save_one/a/ar`, `$db_upsert_a/ar`, `$db_delete_one/a/ar`, `$db_execute_next`, `$db_getWatermark`, `db_flush`, `$db_parse`. 100-record batches
  via `QueryIter`.
- **Query Engine** (`mixquery/`): Builder pattern, `QueryElement` pipeline (IndexPredicate, Projection, Aggregate). Prefilter excludes `_rmx_deleted`/`_rmx_superseded`. Varcode: 7 bits/byte, MSB
  continuation.
- **Storage** (`data/store.rs`): `db.off` (u64 offset index) + `db.dat` (varcode-prefixed MessagePack). Append-only; compaction via RoaringBitmap. Each record gets `_rmx_row`.
- **Auth** (`serve/auth.rs`): JWT via JWKS (remote) + locally-signed HS256 (48h). Supports multiple signing algorithms (not just ES256) since #1001. `email` optional; `sub` required.
- **Desktop** (`desktop/src-tauri/src/app.rs`): Setup: logging → tray → Claude config → deep-links → auth → workspace apps → home. Config TOML at `<data_dir>/com.remixlabs.desktop/config.toml` (
  `mixer_port`, token, workspace, app versions). MCP bridge auto-written to `Claude/claude_desktop_config.json`.
- **Env** (`env.rs`): `hostEnvironment` (AgentServer/Builder/Server/Web/Widget), user, app, surface, token, workspace, org, timezone, platform, URLs, origin secret.

## Data Model

- **Record**: `BTreeMap<String, Value>`, reserved fields: `_rmx_id` (Ref), `_rmx_type`, `_rmx_row` (u64), `_rmx_deleted`, `_rmx_superseded`
- **Storage**: MessagePack in append-only `db.dat`; u64 offset index in `db.off`; RoaringBitmap field indexes for `contains`/`exact_match`/`prefix`/`range`/`scan`
- **Workspace**: `ws_dir/<workspace>/`; app DBs in `v2/` subdirs with `name.txt`, `db.off`, `db.dat`, index files

## Key Patterns

- Feature flags: ~20 in mixer alone composing FFIs, backends, targets
- Error: `thiserror` in libs, `anyhow` at binary boundaries
- Async: Tokio throughout; FFIs async on native, sync on Wasm; `spawn_blocking` for slow drops
- Logging: `tracing` + `RUST_LOG`; `MIXRT_DEBUG` flags; VM logs on `mix-vm` target

## Open Questions

- Compact/index internals: `compact.rs`/`index.rs`/`index_1.rs` — how field indexes are built/maintained
- DuckDB FFI: `ffi/duckdb.rs` registration details, connection lifecycle
- Wasm2c vs wasmtime: `CodeType::decide()` selection logic
- Codec binary format: exact u32 encoding across Wasm boundary

## Recent PRs — Feb 22–28, 2026

17 PRs merged.

### High-signal

| PR          | Summary                                                                                                                                                                                                                                                                              | Author     |
|-------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|------------|
| **#948** ★  | **Configurable mixer HTTP port** (24 commits, 27 files). `mixer_port` in `config.toml`; `mixer_port=0` disables HTTP → custom protocol `remix://localhost/a`. `baseURL`/`mixerHost` derived from config. Path to disabling HTTP by default once MCP/extension decoupled. Fixes #944. | benozol    |
| **#1001** ★ | **Generic JWT signing algorithms** — no longer hard-coded ES256; defers to JWKS endpoint. Required for Lumber's Descope tokens. `email` optional in JWT claims (only `sub` required).                                                                                                | cvermilion |
| **#1003** ★ | **Fix builder params** — large fix (+354/-122, 8 files); builder params broken on desktop.                                                                                                                                                                                           | benozol    |
| **#1022** ★ | **rmx_id index for record lookups** — eliminates constant disk seeks in normal base. +79/-46, 3 files.                                                                                                                                                                               | fredim     |
| **#1028** ★ | **Reset internal fields on save** — `_rmx_*` fields cleared on incoming record before write (correctness fix).                                                                                                                                                                       | fredim     |
| **#1032** ★ | **Faster oud** — `_rmx_tailwind` re-install: ~1.5 min → seconds.                                                                                                                                                                                                                     | benozol    |
| **#1036** ★ | **System plugins path fix** — double `path.Join` bug (was in #bugbash as flush-path bug).                                                                                                                                                                                            | fredim     |

### Minor fixes & polish

| PR    | Summary                                                                                                | Author            |
|-------|--------------------------------------------------------------------------------------------------------|-------------------|
| #1033 | `REMIX_DESKTOP_WORKSPACE_APPS_PATH` env var in all builds; more app-install tracing; earlier menu init | benozol           |
| #1015 | Fix already-installed check for URL-type apps (was not installing when needed)                         | benozol           |
| #1013 | Fix deep link param name for `.remix` install/run                                                      | benozol           |
| #1012 | Fix check-update client action                                                                         | benozol           |
| #1020 | Menu text capitalization/consistency                                                                   | benozol           |
| #1024 | Cleaner channel names (dashes)                                                                         | benozol           |
| #1025 | Bump builtin tailwind theme version                                                                    | arvindthyagarajan |
| #1027 | Tailwind v2 .remix file update                                                                         | arvindthyagarajan |
| #1029 | README corrections (workspace name)                                                                    | arvindthyagarajan |
| #1030 | group.json update (builtins-5)                                                                         | arvindthyagarajan |