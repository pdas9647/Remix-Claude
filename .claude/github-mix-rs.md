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
| `mixfs`       | FS abstraction: `backend_std`/`backend_tokio`/`backend_noop`/`backend_opfs`      |
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

**FFI** (`mixcore/`): `HashMap<String, RawFFI>` (Sync/Async). `GROOVEBOX_FFIS` (db, files, token, duckdb, secrets) + `NON_GROOVEBOX_FFIS` (core, http, mqtt, crypto, wasm, websocket). State holds env,
actions queue, HTTP interceptors, DB seekers, MQTT publisher, DuckDB conn, S3.

**VM Loop** (`mixrun/`): State machine `InitDone→Running→Idle/FFICall/Panic/Exit/Timeout`. Pipeline: `.load()→.link()→.init(env)→.run(name, params)`. Agent caching: snapshot = `CachedCode::Qcode{mem}`
or `Wasm{items,mem}`; restore = reinit + restore mem (no recompile).

**DB FFI:** ~17 FFIs — `$db_get_one`, `$db_save_one/a/ar`, `$db_upsert_a/ar`, `$db_delete_one/a/ar`, `$db_execute_next`, etc. 100-record batches via `QueryIter`.

**Query Engine** (`mixquery/`): Builder pattern, `QueryElement` pipeline (IndexPredicate, Projection, Aggregate). Prefilter excludes `_rmx_deleted`/`_rmx_superseded`. Varcode: 7 bits/byte, MSB
continuation.

**Storage:** `db.off` (u64 offset index) + `db.dat` (varcode-prefixed MessagePack). Append-only; compaction via RoaringBitmap. Record fields: `_rmx_id` (Ref), `_rmx_type`, `_rmx_row`, `_rmx_deleted`,
`_rmx_superseded`. Workspace layout: `ws_dir/<workspace>/v2/` with `name.txt`, db files, indexes.

**Auth:** JWT via JWKS (remote) + locally-signed HS256 (48h). Multiple signing algorithms since #1001. `email` optional; `sub` required.

**Desktop:** Tauri v2. Config TOML at `<data_dir>/com.remixlabs.desktop/config.toml` (`mixer_port`, token, workspace, app versions). Setup: logging → tray → Claude config → deep-links → auth →
workspace apps → home.

**Env:** `hostEnvironment` (AgentServer/Builder/Server/Web/Widget), user, app, surface, token, workspace, org, timezone, platform, URLs.

## Key Patterns

- Error: `thiserror` in libs, `anyhow` at binary boundaries
- Async: Tokio throughout; FFIs async on native, sync on Wasm; `spawn_blocking` for slow drops
- Logging: `tracing` + `RUST_LOG`; `MIXRT_DEBUG`; VM logs on `mix-vm` target
- HTTP FFI: `$http_do`/`$http_do_server`; `BlockUnsafeHttp` guards non-HTTPS/localhost on agent server

## PR Archive — Feb 22 – Mar 21, 2026

★ Key: #948 configurable HTTP port · #1001 JWT multi-alg · #1022 rmx_id index · #1023 named Desktop client actions · #1038 revamp app install · #1048 remix:// intercept fix · #1057 renamed agent
support · #818 OPFS (major, 196 commits) · #1063 idempotent .remix V2 reload · #1069 monorepo CI.

## Recent PRs — Mar 22 – Apr 1, 2026 (9 merged)

| PR          | Summary                                                                                                                                                                                                             | Author   |
|-------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|----------|
| **#1082** ★ | **Agent GET + form-urlencoded** (3 files, +258/-57) — adds `GET /run-agent`, `POST application/x-www-form-urlencoded`; `AgentPathRewrite` fairing canonicalizes legacy `/agents/<agent>` → `/<agent>`; fixes #1051. | Benedikt |
| **#1083** ★ | **S3 Cache-Control: no-cache** (1 file, +16/-4) — all S3 puts + presigned register URLs include `Cache-Control: no-cache`; sets `_rmx_upload_cache_control` in metadata (pairs with protoquery#2308).               | Benedikt |
| **#1071** ★ | **Download file client action** (3 files, +43/-6) — new `remix/download-file` Desktop event: Tauri save dialog + `writeFile`; workaround for `<a download>` unsupported in Tauri; pairs with turntable#11837.       | Simon    |
| **#1084**   | **Remove http-plugin** (7 files, +1/-102) — Tauri http plugin broke other client actions; removed; download works without it.                                                                                       | Simon    |
| **#1081**   | **Better JS error messages** (1 file, +22/-8) — anyhow context chains in `mixcore/js_api.rs`: createDatabase, createMixcore, URL parse, .remix install.                                                             | Benedikt |
| **#1079**   | **open-location anchors** (1 file, +5) — Desktop `open-location` now handles `#anchor` in URLs (was browser-only).                                                                                                  | Simon    |
| **#1076**   | **Dox → rcm folder** (1 file, +1) — Desktop Makefile copies protoquery `.dox` JSON to `rcm/` for upcoming DocViewer plugin.                                                                                         | Gerd     |
| **#1080**   | Builtin app versions bump (`make download-builtin-apps`).                                                                                                                                                           | Daniel   |
| **#1056**   | Disable spellcheck in Desktop UI elements.                                                                                                                                                                          | Benedikt |

## Recent PRs — Apr 5–10, 2026 (1 merged)

| PR    | Summary                                                                                                                                                                                         | Author |
|-------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|--------|
| #1085 | **Fix `and` processing** (5 commits, 2 files, +582/-62) — fixes query engine `and` operator handling. Branch `fred/fix-in`; no PR description. Substantial diff suggests non-trivial logic fix. | fredim |

## Recent PRs — Apr 13–17, 2026 (4 merged)

| PR          | Summary                                                                                                                                                                                                                                                                          | Author     |
|-------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|------------|
| **#998** ★  | **Nested query revision fix** (4 commits, 6 files, +273/-136) — Nested queries were incorrectly applying old/superseded revisions to the inner query. Fixes issue #911. Long-lived PR (opened Feb 13).                                                                           | fredim     |
| **#1093** ★ | **Desktop release from tags** (13 commits, 4 files, +82/-40) — CI pipeline now triggers desktop release from tags matching `release.CHANNEL.BUILD`; skips "halt if no changes" when running from a tag. Followup: version string to incorporate `BUILD` from tag.                | cvermilion |
| **#1092**   | **Fix Tauri keyboard shortcut clashes** (1 commit, 1 file, +1/-8) — Tauri native shortcut handlers clashed with browser shortcuts, breaking builder and Monaco editor shortcuts. Removed all Tauri edit-menu shortcuts except copy/paste (copy/paste still needs Tauri handler). | tlentz     |
| **#1091**   | **Fix desktop config default** (14 commits, 5 files, +28/-14) — Fixes desktop config default; rcm branch reset.                                                                                                                                                                  | benozol    |

## Recent PRs — Apr 18–25, 2026 (2 merged)

| PR          | Summary                                                                                                                                                                                                    | Author     |
|-------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|------------|
| **#1090** ★ | **remove rmx-runtime** (6 commits, 3 files, +4/-3) — Removes the `rmx-runtime` web component from Desktop, reducing memory footprint. Long-lived (Apr 14→21).                                              | simonh1000 |
| **#1097**   | **fix remix/location-changed for runtime** (1 commit, 1 file, +3/-7) — Desktop was putting app/screen into search params, but on reload parsed them from the path. Reload now returns to correct location. | simonh1000 |

## Recent PRs — Apr 26 – Apr 30, 2026 (9 merged)

★ High-signal marked.

| PR          | Date   | Summary                                                                                                                                                                                                                                                                                                                                                                                                                          | Author        |
|-------------|--------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|---------------|
| **#1094** ★ | Apr 30 | **Query optimization for subqueries** (long-lived Apr 16→Apr 30, 9 files) — Major rework of `mixquery`: `optimize.rs` (new), `exec.rs`, `pred.rs`, `writelog.rs`, `reg.rs`. Sub-PR #1106 ("less passes over predicates", Apr 29) was merged into this branch before landing on main. Also touches `mixer/serve/api_root.rs` and `desktop/window.rs`.                                                                             | fredim        |
| **#1086** ★ | Apr 27 | **Consolidate mixcore-js** (long-lived Apr 3→Apr 27) — Rewrites `mixcore/bindings/javascript/` → `mixcore/bindings/js/`. New TS sources: `groovebox-starter.ts`, `groovebox-worker.ts`, `shared.ts`, `wasm-opfs-init.ts`, `wasm-opfs-init-worker.ts`. Removes old separate `mixcore-tauri.ts`/`mixcore-webkit.ts`/`mixcore-base.ts`/`ffi-filter.ts`. Mix-rs leg of 3-repo consolidation (turntable#11897 + flutter-runtime#442). | benozol       |
| **#1101** ★ | Apr 27 | **Consolidate mixcore js in Desktop** — Desktop Makefile updated: copies `mixcore_bg.wasm` + `mixcore.js` from new path structure (was `mixcore_opfs.wasm`). `instant/main.ts`: `init()` now takes 3 args (remixUrl, wasmUrl, jsUrl). `apps.rs`: better error context on install failures. `rcm-lock.json`: turntable 61456→61500, protoquery 8172→8202, mixcore 11026→11194, groovebox 3356→3388.                               | benozol       |
| **#1100**   | Apr 27 | **Blank-query "all" bitmap fix** — When there's no query, the "all" bitmap now only includes rows passing the prefilter (not all rows). Fixes incorrect result counts on unfiltered queries.                                                                                                                                                                                                                                     | fredim        |
| **#1108**   | Apr 29 | **fix-case-sort** — Fixes sort behavior for case expressions in `mixquery`.                                                                                                                                                                                                                                                                                                                                                      | fredim        |
| **#1107**   | Apr 29 | **Enable nontrapping float conversion in wasm-opt** — Adds `--enable-nontrapping-float-to-int` flag to wasm-opt in Mixer Wasm build; prevents trapping on NaN/inf float-to-int conversions.                                                                                                                                                                                                                                      | benozol       |
| **#1105**   | Apr 28 | **Allow X-Requested-With in CORS** — Adds `X-Requested-With` to the allowed CORS headers in the Mixer HTTP server.                                                                                                                                                                                                                                                                                                               | cvermilion    |
| **#1103**   | Apr 28 | **cargo fmt** — Rust formatting pass.                                                                                                                                                                                                                                                                                                                                                                                            | gerdstolpmann |
| **#1102**   | Apr 27 | **CI: disable halt-if-component-changed for mixrun** — Comments out the "skip build if mixrun unchanged" CI optimization; was causing missed builds.                                                                                                                                                                                                                                                                             | gerdstolpmann |
