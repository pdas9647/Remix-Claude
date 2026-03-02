# mix-rs

**Repository:** https://github.com/remixlabs/mix-rs
**Stack:** Rust, TypeScript/Vite (desktop frontend), C (FFI/wasm2c), Java/JNI, Swift, Kotlin
**Purpose:** Rust runtime stack — value system, query engine, VM FFI bridge, Wasm runtime, agent server CLI, Tauri desktop app
**Explored:** L1–L5 (~48 files, ~10500 lines)

## Build & Deploy

- Cargo workspace: resolver v2, 10 members, Rust 2021. Release: `opt-level="z"`, strip, LTO.
- CI: CircleCI — Linux amd64/arm64, macOS arm64, iOS, Android, Windows, WASI. sccache; artifacts → GCS.
- Docker: Debian trixie-slim, port 8000. SPCS: separate slim image.
- Desktop: Tauri v2, Apple-signed, release channels (dev, branch-specific). `rcm` manages deps.
- Feature flags: `all`, `agent_server`, `snowflake_server`, `wasi`; FFIs: `all_ffis` (+DuckDB), `some_ffis`, `wasi_ffis`; backends: wasm2c + wasmtime.

## Crate Map

| Crate         | Role                                                                             |
|---------------|----------------------------------------------------------------------------------|
| `remix`       | Core `Value` enum, u32 codec, `#[remix::bind]` proc macro                        |
| `remix_macro` | `#[bind]` (Rust→Wasm export), `#[derive(HasType)]`                               |
| `mixfs`       | FS abstraction: `backend_std`/`backend_tokio`/`backend_noop`                     |
| `mixquery`    | Query engine: AST, parser, filter/proj/aggr, indexed storage, varcode, WAL       |
| `mixcore`     | FFI bridge: DB, HTTP, crypto, files, DuckDB, Wasm, WebSocket, MQTT, secrets      |
| `mixrun`      | Wasm runtime host: mixcore + MixRT (C VM), wasmtime/wasm2c, LRU cache, C/JNI API |
| `mixer`       | CLI (`serve`/`run`/`mcp`/`db`/`wasm`/`error-decode`); Rocket HTTP server         |
| `desktop/*`   | Tauri v2 app (4 sub-crates: src-tauri, src-shared, src-aux, src-mcp-bridge)      |

**Runtime modes:** mixcore (Web/Desktop/iOS appclip → JS groovebox), mixrun (iOS/Android widgets → Rust), mixer serve (agent server → Rust), mixer run (Slack plugin → JS deno).

## API Surface

- **HTTP:** Rocket + CORS + optional TLS. Swagger at `/v1/swagger/`.
  - v0 (deprecated): workspace CRUD, run-agent, install, SSE, query, permissions
  - v1: `/v1/ws/<ws>/app/<app>/.../agent/<agent>`, `.../documents`, `.../query/<name>`, `.../save`, `.../auth-configs`, `.../permissions`, `/v1/ext/<ext>/<path..>`, `/v1/auth/spcs`
- **C/FFI:** cbindgen headers; Swift, Java/JNI, Kotlin mobile bindings; JS bindings (tauri, wasm, webkit)
- **MCP Bridge:** queries `_rmx_desktop/list_mcp_tools` → `run-agent` → Mix agent. Stdio transport. Auto-writes `claude_desktop_config.json`.

## Core Modules

**Value** (`remix/src/value.rs`): `Undef|Null|Bool|Int64|Number|String|Ref|Binary|Array|Map|Token|Case`. JSON: `{"_rmx_type":..,"_rmx_value":..}`.

**FFI** (`mixcore/`): `HashMap<String, RawFFI>` (Sync/Async). `GROOVEBOX_FFIS` (db, files, token, duckdb, secrets) + `NON_GROOVEBOX_FFIS` (core, http, mqtt, crypto, wasm, websocket). State holds env, actions queue, HTTP interceptors, DB seekers, MQTT publisher, DuckDB conn, S3.

**VM Loop** (`mixrun/`): State machine `InitDone→Running→Idle/FFICall/Panic/Exit/Timeout`. Pipeline: `.load()→.link()→.init(env)→.run(name, params)`. Agent caching: snapshot = `CachedCode::Qcode{mem}` or `Wasm{items,mem}`; restore = reinit + restore mem (no recompile).

**DB FFI:** ~17 FFIs — `$db_get_one`, `$db_save_one/a/ar`, `$db_upsert_a/ar`, `$db_delete_one/a/ar`, `$db_execute_next`, etc. 100-record batches via `QueryIter`.

**Query Engine** (`mixquery/`): Builder pattern, `QueryElement` pipeline (IndexPredicate, Projection, Aggregate). Prefilter excludes `_rmx_deleted`/`_rmx_superseded`. Varcode: 7 bits/byte, MSB continuation.

**Storage:** `db.off` (u64 offset index) + `db.dat` (varcode-prefixed MessagePack). Append-only; compaction via RoaringBitmap. Record fields: `_rmx_id` (Ref), `_rmx_type`, `_rmx_row`, `_rmx_deleted`, `_rmx_superseded`. Workspace layout: `ws_dir/<workspace>/v2/` with `name.txt`, db files, indexes.

**Auth:** JWT via JWKS (remote) + locally-signed HS256 (48h). Multiple signing algorithms since #1001. `email` optional; `sub` required.

**Desktop:** Tauri v2. Config TOML at `<data_dir>/com.remixlabs.desktop/config.toml` (`mixer_port`, token, workspace, app versions). Setup: logging → tray → Claude config → deep-links → auth → workspace apps → home.

**Env:** `hostEnvironment` (AgentServer/Builder/Server/Web/Widget), user, app, surface, token, workspace, org, timezone, platform, URLs.

## Key Patterns

- Error: `thiserror` in libs, `anyhow` at binary boundaries
- Async: Tokio throughout; FFIs async on native, sync on Wasm; `spawn_blocking` for slow drops
- Logging: `tracing` + `RUST_LOG`; `MIXRT_DEBUG`; VM logs on `mix-vm` target
- HTTP FFI: `$http_do`/`$http_do_server`; `BlockUnsafeHttp` guards non-HTTPS/localhost on agent server

## Recent PRs — Feb 22–28, 2026 (17 merged)

| PR         | Summary                                                                      | Author          |
|------------|------------------------------------------------------------------------------|-----------------|
| **#948**   | Configurable mixer HTTP port; `mixer_port=0` disables HTTP → custom protocol | Benedikt        |
| **#1001**  | Generic JWT signing algs (not just ES256); Lumber Descope support            | Chris           |
| **#1003**  | Fix builder params on desktop (+354/-122)                                    | Benedikt        |
| **#1022**  | rmx_id index for record lookups (eliminates disk seeks)                      | Fred            |
| **#1028**  | Reset internal `_rmx_*` fields on save (correctness)                         | Fred            |
| **#1032**  | Faster oud: `_rmx_tailwind` re-install ~1.5min → seconds                     | Benedikt        |
| **#1036**  | System plugins path fix (double path.Join)                                   | Fred            |
| #1013      | Deep link param: `remix_file_url` → `url`                                    | Benedikt        |
| #1015      | Fix already-installed check for URL-type apps                                | Benedikt        |
| #1024-1033 | Minor: channel names, menu text, tailwind bumps, README, env var, tracing    | Benedikt/Arvind |
