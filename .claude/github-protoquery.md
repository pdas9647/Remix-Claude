# protoquery

**Repository:** https://github.com/remixlabs/protoquery
**Stack:** OCaml 4.12.1, Mix (.mix/.dox), C (FFI), JS/Webpack (Wasm worker), WebAssembly
**Purpose:** Mix language compiler, standard library, VM runtime bridge, compiler server
**Explored:** L1–L5 (all layers)

## Build & Deploy

- **Build:** OASIS + OMake + opam; `make deps-install && make` → binaries in `progs/`
- **CI:** CircleCI — amd64, arm64, macOS; static/dynamic/Wasm/Groovebox/rebuild tests
- **Artifacts:** `artifacts/bin/<target>/` (mixc, mix_client, mix_migrate, mixc_server); `.mixlib`/`.mixexe` + `.wasm` libs
- **Deploy:** GCS via `rcm comp publish`; Docker for WebSocket server (Debian + Node 22, port 8000); npm (mixc-starter, mixc-worker)
- **RCM deps:** groovebox (^0.2.0), mixrun (^0.3.0), mixer/mix-rs (^0.2.0)
- **ABI:** Currently 27 (`1000*major + minor`). Dev builds use ABI=0. ABI-27: ~45 opcodes + typed builtin catalog.
- **Wasm build:** `wasm-build/` via OMake; wasicaml compiles OCaml → Wasm

## Architecture

### Compilation Pipeline

```
Source (.mix) → Syntax.ml (Lexer+Parser) → AST
  → Typing.ml (Hindley-Milner) → checked phrases
  → Translate.ml → QCode (register-based bytecode, max 1023 instr/section)
  → Linker.ml (global/builtin/foreign remapping) → linked iqccontainer
  → Omixrun.ml (native VM via ctypes)  OR  Driver.ml → ToAsm.ml → .wasm
```

### Transport Layer

| Transport   | Entry                 | Use case                                               |
|-------------|-----------------------|--------------------------------------------------------|
| stdio pipes | `Mixc_server_main.ml` | Host spawns `mixc -server`, length-prefixed binary IPC |
| WebSocket   | `Websocket_server.ml` | Browser/remote, SCRAM auth, session tokens             |
| Wasm Worker | `Worker_main.ml`      | Browser Web Worker / Node worker_threads               |

JS bootstrap: `mixc-starter` spawns Wasm worker, coordinates via pub/sub on `/org/{org}/ws/{ws}/mixc`.

## Executables

| Binary           | Purpose                                                       |
|------------------|---------------------------------------------------------------|
| `mixc`           | Primary compiler CLI (local, remote, REPL)                    |
| `mixc_server`    | Stdin/stdout compiler server                                  |
| `mixc_websocket` | WebSocket compiler server (Docker)                            |
| `mix_client`     | Interactive REPL (#mark, #load, #push, #pop, #invoke, #debug) |
| `worker`         | Wasm worker registration                                      |
| `mixdoc`         | Doc generator (.mix+.dox → JSON/Markdown)                     |

C API: `protoquery_init()` / `protoquery_compile()` — minimal ABI for C/Go embedding (Go wrapper in `extapi/go/`, used by `amp`).

## Server Protocol

**Requests:** Create/Destroy_session, Send_libraries, Check_source, Imports, Compile_modules, Compile_phrases, Request_code (QCODE/WASM), Send_code, Get_stdlib/Get_stdlib_id, To_json/To_binary, About,
Statistics

**Response:** `proto_out` = session + success + messages (info/warning/error) + typed payload

## OCaml Libraries

| Library                            | Purpose                                           |
|------------------------------------|---------------------------------------------------|
| **Mix** (`src/mix/`)               | Compiler core: 36 modules                         |
| **Mixrun** (`src/mixrun/`)         | Native VM bridge (ctypes FFI)                     |
| **Mix_cross** (`src/mix-cross/`)   | Cross-compilation: Wasm (ToAsm), C (ToC), ABI     |
| **Mix_server** (`src/mix-server/`) | Protocol handler, Worker, ABI, MsgPack            |
| **Mix_websocket**                  | WebSocket transport (lwt, SCRAM auth)             |
| **Amp_client**                     | AMP client: HTTP/MQTT, REPL, Track/Docs/Code APIs |

## Key Compiler Modules (`src/mix/modules/`)

- **Types.ml** — `data` (13 variants), `value`, `type_term` (Tany/Tbool/Tnumber/Tfun/Tarray/Trecord/Tcases/Tvar/Tuser/Tmodule…), `library_id`, `mod_id`, `RmxID`
- **Syntax.ml** — Lexing + parsing; `parse_phrase_string`, `parse_module`, `dependencies`
- **Typing.ml** — Hindley-Milner with cells, streams, modules
- **Compiler.ml** — `typecheck` → `translate`; `create_standard_universe`; `compile`/`compile_lib`
- **Translate.ml** — AST → QCode; max 1023 instr/section (10-bit valueOrigin)
- **QCode.ml** — Bytecode IR: `qccontainer`, ~25 opcodes, terminators (Qjump/Qreturn/Qapply/Qforeign/Qcoroutine2/Qsuspend/Qcopylib)
- **Linker.ml** — Library linking: `repository`, global/builtin/foreign remapping, LibQCode/LibWasm
- **Transform/Transmod/Transopt/Transpattern.ml** — Optimization passes

## Mix Standard Library (`src/mix/stdlib/`)

~55 modules (.mix + .dox pairs). Key modules:

- **stdlib.mix** — Prelude: option/result/mutable/lazy, reactive spreadsheet engine (cells/sheets/links), module registry ($module_add/expose/get), host-env dispatch, action system
- **db.mix** — 5 scopes (mainDB/appDB/sessionInMemDB/globalInMemDB/remoteDB), query pipeline (filter/map/sort → AST or stream fallback), live subscriptions via watermark-based change detection
- **compiler.mix** — Compiled artifact model (library/executable records), libID format (`<name>-<version>@<hash>`), compiler channel resolution (remixDev/Beta/Prod), `awaitPresence` with mutex
- **agent.mix** — 3 call paths: subroutineCall (same-session), localAgentCall (pubsub), httpCall (REST); onceKey dedup, HMAC origin verification

Other: core (bool/number/string/option/result/regex), collections (array/set/map/gset/uniqArray), I/O (http/file/binary/crypto/wasm/mime), streaming (stream/xml/websocket), editor/DOM, runtime,
platform (auth/env/secrets/co/messaging/loader)

## Compiler Driver (`src/mix/driver/`)

- **compilerDriver.mix** — Session orchestrator: create/destroy/compilePhrases/compileModules/publishLibrary; result monad (`res(T)` with andThen/resMap); dual transport (MQTT or websocket)
- **remixFile.mix** — .remix archive builder (zip): manifest.json + runtime.json (dbName/appName/settings/styles) + appMeta (asset type/access/collaborators); staging DB → `$db_zipWrite` FFI
- Other: compilerBuild, compilerRule/File/Lib/Exe, compilerREPL, codegenDriver/Util, remixRecipe

## Other Notable Modules

- **Slate** (`slateDef.mix`) — Visual node editor types: nodeType union (screenParams/screenCard/outputParams/compInstType/constType…), binding (connection points with visual type + direction)
- **JWT** (`jwt.mix`) — RFC 7519: create/verify via JWS, standard claim accessors
- **_rmx_makeAgent / _rmx_sessionAgent / _rmx_agentAgent** → compiled into `agents.mixexe`
- **mixdoc** — Doc generator: .mix/.dox pairs → types → Markdown/JSON

## Data Model

- **Core values** (`Mix.Types.data`): Dundefined | Dnull | Dbool | Dint(int64) | Dfloat | Dstring | Dbinary | Dobject(SMap.t) | Darray | Dref(RmxID.t) | Dmutable | Dcasename | Dcase | Dtag | Dbitvec
- **QCode format:** Header (version "qcode-5", ABI, requires, provides/aliases) + Object (builtins/foreigns/sections/globals/imports). MsgPack for protocol; custom binary for files; Wasm uses custom
  sections.
- **Amp Client APIs:** Track (session, compile, VM: Load/Push/Pop/Input/Invoke/Poll), Docs (CRUD + app mgmt), Code (Executable/Library, stdlib), MQTT topics: `/org/{org}/ws/{ws}/...`

## Key Patterns

- Session-based compilation: stateful sessions with incremental code accumulation
- Dual serialization: JSON (Yojson) for messages, MsgPack for binary, custom binary for QCode
- PPX derivers: `ppx_deriving_yojson`, `ppx_deriving.show`, `ppx_blob`, `ppx_getenv`
- Mix stdlib: `$twelve` for type-erasure casts, `_inline_` pragmas, `$DEFPRIMITIVES` for builtins
- Tests: 28+ `T_*.ml` modules (inline Mix → compile → assert); shell integration tests; rebuild vs live envs

## Open Questions

- **Groovebox:** RCM dep (min build 425) — JS/Wasm VM runtime executing QCode
- **mixer/mix-rs:** Rust native VM that Omixrun FFIs into
- **Go wrapper:** `extapi/go/` submodule in `amp` — unexplored

## Recent PRs — Feb 22–28, 2026

7 PRs merged (all gerdstolpmann).

| PR          | Summary                                                                                                                                                            |
|-------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| **#2226** ★ | **Blob support** (+2784 lines, 6 files). Held until end of February. Adds blob handling to the Mix compiler/stdlib HTTP layer.                                     |
| **#2263** ★ | **Expose compress/decompress** (+2743 lines, 4 files). zlib compress/decompress in Mix stdlib. Paired with groovebox/pull/464.                                     |
| **#2277** ★ | **exe-only package asset_type fix** — sets `asset_type=AppExecutable` when .remix has no builder assets. Fixes bugbash: exec-only apps were installing as Project. |
| #2278       | Remix file exporter: handle bad/missing `appMeta` records. Follow-up to #2277.                                                                                     |
| #2280       | Compiler driver: delete `*.mixhdr` on lib deletion; fix over-deletion when only `targetLibs` set.                                                                  |
| #2281       | REPL echo fix when calling agents — use `subscribeFG` to avoid early echoes. Closes #2279.                                                                         |
| #2274       | Fix `localAgentsConfig`: agent agent was not being started (missed in #2272).                                                                                      |