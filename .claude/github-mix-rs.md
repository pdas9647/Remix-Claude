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

## PR Archive — through May 8, 2026

★ Older: #818 OPFS · #948 HTTP port · #1001 JWT multi-alg · #1022 rmx_id index · #1038 app install · #1063 .remix V2 · #1071 download-file · #1082 agent GET+form-urlencoded · #1083 S3 no-cache · #1085 fix `and` · #998 nested query fix · #1090 remove rmx-runtime · #1093 Desktop release tags · #1097 fix location-changed.

★ Apr 26–30: #1094 query optimization for subqueries (new `mixquery/optimize.rs`); #1086 + #1101 consolidate mixcore-js (3-repo effort w/ turntable#11897 + flutter-runtime#442). Also #1100 blank-query "all" bitmap fix; #1108 case-sort fix; #1107 nontrapping float-to-int; #1105 X-Requested-With CORS; #1103 cargo fmt; #1102 disable halt-if-component-changed for mixrun.

★ May 1–8: #1039 Revamp MCP (long-lived Mar 2→May 8; moves MCP out of standalone `mixer/src/mcp.rs` → v1 API layer `mixer/src/serve/api_v1/mcp.rs` + `mcp_tool.rs`; Desktop MCP bridge reworked); #1099 Desktop panic hook (`std::panic::set_hook` + structured tracing logs); #1111 OPFS Wasm init worker catches exceptions (typed `Result<T>` envelope); #1115 channel switch requires confirmation (`switch_channel` gains `confirm: bool`).

## Recent PRs — May 11–16, 2026 (5 merged)

★ Key: #1123 port to rcm2 (4 `rcm.config.mjs` → `rcm.config.json`; CI drops gsutil/Python; `desktop-release.rs` adds service-account JWT path via `jsonwebtoken` crate); #1026 arm64 build of mixer server+docker (long-lived Feb 25→May 11).

| PR          | Date   | Summary                                                                                                                                                                                                                                                                                                                                                                                                                                                                                  | Author     |
|-------------|--------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|------------|
| **#1123** ★ | May 16 | **Port mix-rs to rcm2** — Converts 4 `rcm.config.mjs` → `rcm.config.json`. CI drops gsutil+Python venv+`authorize-gcloud`; rcm installed via curl from `rmx-static/rcm/rcm`. `desktop-release.rs` mints GCS token via service-account JWT when `RCM_CI_EDITOR_KEY` set (jsonwebtoken crate, RS256). | benozol    |
| **#1026** ★ | May 11 | **arm64 build of mixer server + Docker** (long-lived Feb 25→May 11) — `amp_exe_arm64` executor; `mixer_linux`/`mixer_build_server` parameterized by arch. `-arm64` build tag suffixes + `--platform=linux/arm64` Docker. New `mixer-snowflake` cargo bin (separate from `mixer`). | cvermilion |
| #1120       | May 15 | **Debug flags for JS API + cleanup** — `MixcoreConfig` types gain optional `debug?: boolean`; console.log calls gated. Refactors `shared.ts`: merges `createMixcoreWasmReg` into `initMixcoreJs`; `wasm-opfs-init.ts` takes single config object. Bumps wasm-bindgen 0.2.114→0.2.121. | benozol    |
| #1118       | May 14 | **Fix rcm build tag** — CI fix for rcm build tag handling. | cvermilion |
| #1116       | May 11 | **Use fork of rust-s3 while awaiting upstream PR** — Cargo.toml points at `github.com/w4nderlust/rust-s3` (forked rev) until upstream merges. | cvermilion |

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
