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

## PR Archive — through Apr 25, 2026

★ #818 OPFS · #948 HTTP port · #1001 JWT multi-alg · #1022 rmx_id index · #1038 app install · #1063 .remix V2 · #1071 download-file · #1082 agent GET+form-urlencoded · #1083 S3 no-cache · #1085 fix `and` · #998 nested query fix · #1090 remove rmx-runtime · #1093 Desktop release tags · #1097 fix location-changed.

## Recent PRs — Apr 26 – Apr 30, 2026 (9 merged)

| PR          | Date   | Summary                                                                                                                                                                                                                                                                                           | Author        |
|-------------|--------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|---------------|
| **#1094** ★ | Apr 30 | **Query optimization for subqueries** (long-lived Apr 16→Apr 30) — New `mixquery/optimize.rs`; reworks `exec.rs`, `pred.rs`, `writelog.rs`, `reg.rs`. Sub-PR #1106 merged into branch first.                                                                                                      | fredim        |
| **#1086** ★ | Apr 27 | **Consolidate mixcore-js** (long-lived Apr 3→Apr 27) — Rewrites `mixcore/bindings/javascript/` → `mixcore/bindings/js/`; new `groovebox-starter/worker` + `wasm-opfs-init` TS modules; removes old per-platform split files. Mix-rs leg of 3-repo effort (turntable#11897 + flutter-runtime#442). | benozol       |
| **#1101** ★ | Apr 27 | **Consolidate mixcore js in Desktop** — Makefile: copies `mixcore_bg.wasm`+`mixcore.js` (new paths); `init()` gains jsUrl arg; rcm-lock: turntable→61500, mixcore→11194, groovebox→3388.                                                                                                          | benozol       |
| **#1100**   | Apr 27 | **Blank-query "all" bitmap fix** — When there's no query, the "all" bitmap now only includes rows passing the prefilter (not all rows). Fixes incorrect result counts on unfiltered queries.                                                                                                      | fredim        |
| **#1108**   | Apr 29 | **fix-case-sort** — Fixes sort behavior for case expressions in `mixquery`.                                                                                                                                                                                                                       | fredim        |
| **#1107**   | Apr 29 | **Enable nontrapping float conversion in wasm-opt** — Adds `--enable-nontrapping-float-to-int` flag to wasm-opt in Mixer Wasm build; prevents trapping on NaN/inf float-to-int conversions.                                                                                                       | benozol       |
| **#1105**   | Apr 28 | **Allow X-Requested-With in CORS** — Adds `X-Requested-With` to the allowed CORS headers in the Mixer HTTP server.                                                                                                                                                                                | cvermilion    |
| **#1103**   | Apr 28 | **cargo fmt** — Rust formatting pass.                                                                                                                                                                                                                                                             | gerdstolpmann |
| **#1102**   | Apr 27 | **CI: disable halt-if-component-changed for mixrun** — Comments out the "skip build if mixrun unchanged" CI optimization; was causing missed builds.                                                                                                                                              | gerdstolpmann |

## Recent PRs — May 1–8, 2026 (4 merged)

| PR          | Date  | Summary                                                                                                                                                                                                                                                                                                                                        | Author  |
|-------------|-------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|---------|
| **#1039** ★ | May 8 | **Revamp MCP** (long-lived Mar 2→May 8) — Moves MCP out of standalone `mixer/src/mcp.rs` (deleted, 315 lines) into the v1 API layer as `mixer/src/serve/api_v1/mcp.rs` (new, 216 lines) + `mcp_tool.rs` model. Desktop MCP bridge (`src-mcp-bridge/main.rs`) significantly reworked; `claude.rs` Claude Desktop integration updated (+88/-33). | benozol |
| **#1099**   | May 6 | **Desktop panic hook** (Apr 24→May 6) — `std::panic::set_hook` installed in `run()`; new `panic_hook` captures `payload_as_str()`, `location()`, `Backtrace::force_capture()` and logs via `tracing::error!`. Desktop panics now produce structured logs instead of silent crashes.                                                            | benozol |
| **#1111**   | May 6 | **OPFS Wasm init worker: catch exceptions** — Worker handler wrapped in try/catch; posts typed `Result<T>` (`{ok,result}` or `{ok:false,message}`). `wasm-opfs-init.ts` rejects promise on error. Also adds error context to `.remix` fetch/parse failures in `js_api.rs`.                                                                     | benozol |
| **#1115**   | May 8 | **Channel switch: require confirmation** — `switch_channel` gains `confirm: bool` param; shows dialog before switching. Removes old restriction to home-app-only switching.                                                                                                                                                                    | benozol |

---

## Groovebox / Mix VM (mixrt + machine-starter)

> Source: [mixrt](https://www.notion.so/11f1d464528f80ca8396c6f29f351d26)
> Parent: The Remix Platform → Platform Engineering (Internal) Home

### Architecture

The Mix VM is a C library (`machine-wasm/src` in groovebox), compiled to Wasm via [`wasi-sdk`](https://github.com/WebAssembly/wasi-sdk) → `mixrt.wasm`. This Wasm is then compiled back to C via
`wasm2c` (from `wabt`) → `mixrt.h`/`mixrt.c` for native use in `mixer` and iOS app clips.

| Artifact                 | Description                                                                 |
|--------------------------|-----------------------------------------------------------------------------|
| `libmixrt.a`             | Static library compiled from `machine-wasm/src` C sources                   |
| `mixrt.wasm`             | Wasm output from `libmixrt.a` via `wasi-sdk`                                |
| `mixrt.h`/`.c`           | wasm2c output for native environments (mixer CLI, iOS app clips)            |
| `machine-wasm.js`        | Node bundle: `mixrt.wasm` baked in as single-file web worker module         |
| `machine-wasm.worker.js` | Distributed via `machine-starter`; for web, `mixrt.wasm` provided by caller |

### machine-starter

JS npm package (internal only, distributed via `rcm` as `file://` dependency). Used by `turntable`.

```js
import {StartWASM2} from "@remix_labs/machine-starter";

const config = {
    baseURL: "https://...",
    org: "local",
    workspace: "local",
    vmID: "0x1234",
    user: "fred",
    token: "abcd",
    noOutputViaMQTT: ...,   // if true: "$outputWithVersion" FFI instead of MQTT output topic
    machType: "WASM",       // always "WASM" even for QCode execution
    mixrtCode: "https://somewhere/mixrt.wasm",  // web only: URL, base64 string, or Uint8Array
};
StartWASM2(hub, config);  // hub = messaging hub or channel
```

### FFI Configuration

Three tiers (increasing precedence):

1. **Worker-internal FFIs** — defined in `machine-wasm/js/ffi.js` (e.g. `$http_do`)
2. **Mixcore FFIs** — via `MixcoreWrapper` in `machine-wasm/js/mixcore.js`; selected by `config.mixcore.kind`:
    - `"webkit"` — Swift/WebKit: `window.webkit.messageHandlers["mixcore_ffi_dispatch"]` (iOS app clips)
    - `"tauri"` — `window.__TAURI__.invoke("mixcore_ffi_dispatch", {co, name, argsBuf})`; needs `dbName`, `dbUser`, `dbDir`
    - `"service-worker"` — browser builder: `mixcore` packaged as SW (`mix-rs/mixcore/sw`); pass `{kind: "service-worker", mixcoreId: "my-id"}`
3. **Local FFIs** — defined in the `StartWASM2` caller context; invoked via MQTT hub message-passing:
   ```js
   config.localFFIs = {
     "name_sync":  ((connector, args) => ...),
     "name_async": (async (connector, args) => ...),
     "name_raw": { isRaw: true, run: ((conn, buf) => ...) },       // Mix-encoded buffer in/out
     "name_can_fail": { canFail: true, useJsonDecoder: true,
                        run: ((conn, args) => {name:"ok",value:...}) },
   };
   ```

### Value Types Across FFI Boundary

Supported: `undefined`, `null`, `bool`, `number`, `string`, `Array`, plain objects (→ Mix maps), `Uint8Array` (→ Mix binary), `Case`, `Token`, `Opaque`. All other JS types are unsupported.
