# protoquery

**Repository:** https://github.com/remixlabs/protoquery
**Stack:** OCaml 4.12.1, Mix (.mix/.dox), C (FFI), JS/Webpack (Wasm worker), WebAssembly
**Purpose:** Mix language compiler, standard library, VM runtime bridge, compiler server — the core compilation toolchain for Remix
**Explored layers:** L1–L5 (all layers)

## Build & Deploy

- **Build:** OASIS (`_oasis`) + OMake + opam; `make deps-install && make` → binaries in `progs/`
- **CI:** CircleCI — amd64, arm64, macOS; tests: static, dynamic, Wasm, Groovebox integration, rebuild vs remixBeta/remixProd
- **Artifacts:** `artifacts/bin/<target>/` (mixc, mix_client, mix_migrate, mixc_server), `artifacts/lib/qcode/mixc/` (.mixlib/.mixexe), `artifacts/lib/wasm32-unknown-unknown/mixc/`
- **Deploy:** GCS via `rcm comp publish`; static to `gs://rmx-static/mixc/<BUILD_TAG>/lib`; Docker for WebSocket server (Debian + Node 22, port 8000); npm packages (mixc-starter, mixc-worker)
- **RCM deps:** groovebox (^0.2.0), mixrun (^0.3.0), mixer/mix-rs (^0.2.0)
- **ABI:** Currently 27 (format `1000*major + minor`), tracked in `abi/abi-<N>.json`; dev builds use ABI=0. ABI-27 enumerates ~45 VM opcodes + large builtin function catalog with typed signatures
- **Wasm build:** `wasm-build/` delegates to OMake; uses wasicaml to compile OCaml → Wasm

## Architecture

### Compilation Pipeline

```
Source (.mix) → Syntax.ml (Lexer + Parser) → AST phrases
  → Typing.ml (type inference) → checked phrases
  → Translate.ml → QCode (register-based bytecode, max 1023 instr/section)
  → Linker.ml (global/builtin/foreign remapping) → linked iqccontainer
  → Omixrun.ml (native VM via ctypes FFI)  OR  Driver.ml → ToAsm.ml → .wasm
```

### Transport Layer

Three transports wrap `Server.ml` (shared protocol handler):

| Transport   | Entry                 | Use case                                                   |
|-------------|-----------------------|------------------------------------------------------------|
| stdio pipes | `Mixc_server_main.ml` | Host spawns `mixc -server`, length-prefixed binary IPC     |
| WebSocket   | `Websocket_server.ml` | Browser/remote, binary framing, SCRAM auth, session tokens |
| Wasm Worker | `Worker_main.ml`      | Browser Web Worker / Node worker_threads                   |

JS bootstrap: `mixc-starter` exports `StartMixCompiler`/`WaitForMixCompiler` — spawns Wasm worker, coordinates via pub/sub on `/org/{org}/ws/{ws}/mixc`. The `mixc-worker/src/worker.js` loads the .wasm
binary, marshals JSON requests into C-string memory, invokes `do_request`, and publishes responses through the Hub.

## Executables

| Binary           | Source                   | Purpose                                                       |
|------------------|--------------------------|---------------------------------------------------------------|
| `mixc`           | `Mixc_main.ml`           | Primary compiler CLI (local, remote, REPL)                    |
| `mixc_server`    | `Mixc_server_main.ml`    | Stdin/stdout compiler server                                  |
| `mixc_websocket` | `Mixc_websocket_main.ml` | WebSocket compiler server (Docker)                            |
| `mix_client`     | `Mix_client_main.ml`     | Interactive REPL (#mark, #load, #push, #pop, #invoke, #debug) |
| `worker`         | `Worker_main.ml`         | Wasm worker registration                                      |
| `mixdoc`         | `Dox_main.ml`            | Doc generator (.mix+.dox → JSON/Markdown)                     |
| Others           |                          | mixc_local, mix_migrate, mixinfo, unittests, cross_check      |

C API: `protoquery_init()` / `protoquery_compile()` — minimal ABI for C/Go embedding (Go wrapper in `extapi/go/`, submodule in `amp`).

## Server Protocol (`Mix_server.Server`)

**Requests:** Create/Destroy_session, Send_libraries, Check_source, Imports, Compile_modules, Compile_phrases, Request_code (incremental, variant: QCODE/WASM), Send_code, Get_stdlib/Get_stdlib_id,
To_json/To_binary, About, Statistics

**Response:** `proto_out` = session + success + messages (info/warning/error) + typed payload

## OCaml Libraries

| Library           | Path                 | Purpose                                                 |
|-------------------|----------------------|---------------------------------------------------------|
| **Mix**           | `src/mix/`           | Compiler core: 36 modules                               |
| **Mixrun**        | `src/mixrun/`        | Native VM bridge (ctypes FFI)                           |
| **Mix_cross**     | `src/mix-cross/`     | Cross-compilation: Wasm (ToAsm), C (ToC), ABI           |
| **Mix_server**    | `src/mix-server/`    | Protocol handler, Worker, ABI, MsgPack server           |
| **Mix_websocket** | `src/mix-websocket/` | WebSocket transport (lwt, SCRAM auth)                   |
| **Amp_client**    | `src/amp-client/`    | AMP client: HTTP/MQTT, REPL modes, Track/Docs/Code APIs |
| **Bigstring**     | `src/bigstring/`     | Bigarray string buffer (C FFI)                          |
| **Crc32**         | `src/crc32/`         | CRC32 hashing (C FFI)                                   |

## Key Compiler Modules (`src/mix/modules/`)

- **Types.ml** — `data` (13 variants: Dundefined…Dbitvec), `value`, `type_term` (Tany/Tbool/Tnumber/Tfun/Tarray/Trecord/Tcases/Tvar/Tuser/Tmodule…), `library_id`, `mod_id`, `RmxID`
- **Syntax.ml** — Lexing + parsing orchestrator; `parse_phrase_string`, `parse_module`, `dependencies`
- **Typing.ml** — Hindley-Milner with extensions for cells, streams, modules
- **Compiler.ml** — `typecheck` → `translate`; `create_standard_universe`; `compile` / `compile_lib`
- **Translate.ml** — AST → QCode; max 1023 instr/section (10-bit valueOrigin)
- **QCode.ml** — Bytecode IR: `qccontainer`, ~25 opcodes, control flow terminators (Qjump/Qreturn/Qapply/Qforeign/Qcoroutine2/Qsuspend/Qcopylib)
- **Linker.ml** — Library linking: `repository`, global/builtin/foreign remapping, LibQCode/LibWasm
- **CUL.ml** — Intermediate representation between AST and QCode
- **Transform.ml** / **Transmod.ml** / **Transopt.ml** / **Transpattern.ml** — Optimization passes

## Mix Standard Library (`src/mix/stdlib/`)

~55 modules (.mix + .dox pairs). Key architectural modules:

- **stdlib.mix** — Prelude: core types (option, result, mutable, lazy), reactive spreadsheet engine (cells, sheets, links with version tracking), module registry ($module_add/expose/get with modSpace:
  screen/agent/component), host environment dispatch (builder/server/web/widget/agent-server), action system (actionTuple, closures, invoke loop)
- **db.mix** — Database layer: 5 scopes (mainDB/appDB/sessionInMemDB/globalInMemDB/remoteDB), dispatcher pattern (rmx_remote vs default), query pipeline (rewritable filter/map/sort → AST nodes or
  stream fallback), live subscriptions via watermark-based change detection, ref-as-case support
- **compiler.mix** — Compiled artifact model (library/executable records), storage abstraction (dbStore/fileStore), libID format (`<name>-<version>@<hash>`), compiler channel resolution (
  remixDev/Beta/Prod), compiler presence via `awaitPresence` with mutex lock
- **agent.mix** — Agent invocation: 3 call paths (subroutineCall for same-session, localAgentCall via pubsub, httpCall via REST), onceKey deduplication, HMAC origin verification, EscJSON serialization

Other stdlib categories: core (bool/number/string/option/result/regex/bitset), collections (array/set/map/gset/uniqArray/circuitarray), I/O (http/file/binary/crypto/wasm/mime), streaming (
stream/xml/websocket), editor/DOM (dom/domUtil/reflect/editorTypes/editorNode/editorScreen), runtime (container/viewstack/debugger/testing/vm), platform (
auth/env/secrets/logging/metrics/co/messaging/loader/calendar/color/random/memo/util)

## Compiler Driver (`src/mix/driver/`)

- **compilerDriver.mix** — Session orchestrator: `createSession`/`destroySession`, `compilePhrases`/`compileModules`, `publishLibrary`/`saveLibrary`, result monad (`res(T)` with andThen/resMap), dual
  transport (MQTT pubsub or websocket)
- **remixFile.mix** — .remix archive builder (zip): content assembly (addBinary/addLibrary/addExecutable/addFile), manifest.json, runtime.json (dbName/appName/settings/styles), appMeta (asset
  type/access/collaborators), staging DB → pack step via $db_zipWrite FFI
- Other: compilerBuild, compilerRule/File/Lib/Exe, compilerREPL, codegenDriver/Util, remixRecipe

## Other Notable Modules

- **Slate editor** (`src/mix/slate/slateDef.mix`) — Visual node editor types: nodeType union (screenParams/screenCard/outputParams/compInstType/constType/forwarderType…), binding (connection points
  with visual type + direction)
- **JWT** (`src/mix/jsonweb/jwt.mix`) — RFC 7519: `t = {claims: map(data)}`, create/verify via JWS, standard claim accessors
- **CSV** (`src/mix/parsing/csv.mix`) — RFC-compliant CSV parser (state-machine tokenizer, handles quoted fields)
- **Special screens** — `_rmx_errorFallback.mix` (error display with restart/ignore/go-home + error report), `_rmx_debugShell.mix` (minimal placeholder)
- **mixdoc** (`src/mixdoc/`) — Doc generator: discovers .mix/.dox pairs → compiles for types → parses .dox markup → cross-checks → emits Markdown/JSON. Types in `Doxtypes.ml`:
  Project/Module/Def/Cell/Case/Type/Example/Heading/FreeText
- **Agents** (`src/mix/agents/`) — _rmx_sessionAgent, _rmx_agentAgent, _rmx_makeAgent → `agents.mixexe` (QCode + Wasm)

## Data Model

**Core values** (`Mix.Types.data`): Dundefined | Dnull | Dbool | Dint(int64) | Dfloat | Dstring | Dbinary | Dobject(value SMap.t) | Darray(value array ref) | Dref(RmxID.t) | Dmutable | Dcasename |
Dcase | Dtag | Dbitvec

**QCode format:** Header (version "qcode-5", ABI, requires, provides/aliases) + Object (builtins/foreigns/sections/globals/imports). Sections: instruction[] + final_instruction + framesize + arity.
MsgPack for protocol; custom binary for files; Wasm uses custom sections.

**Amp Client APIs:** Track (session lifecycle, compile, VM interaction: Load/Push/Pop/Input/Invoke/Poll), Docs (CRUD + app mgmt), Code (Executable/Library objects, stdlib), MQTT (broker, topics:
`/org/{org}/ws/{ws}/...`)

## Key Patterns

- **Module packing:** OCaml `Pack: true` → namespaced (e.g., `Mix.Types`, `Mix_cross.ToAsm`)
- **Session-based compilation:** Stateful sessions with incremental code accumulation
- **Dual serialization:** JSON (Yojson) for messages, MsgPack for binary, custom binary for QCode
- **PPX derivers:** `ppx_deriving_yojson`, `ppx_deriving.show`, `ppx_blob`, `ppx_getenv`
- **Mix stdlib patterns:** `$twelve` for unsafe type-erasure casts, `_inline_` pragmas, descriptor-tagged tuples, `$DEFPRIMITIVES` macro for builtin definitions
- **Test infrastructure:** `T_*.ml` (28+ test modules): inline Mix source snippets → compile → assert output; shell integration tests (`bin/*_test.sh`); rebuild tests vs live environments;
  `testdata/hello/hello.mix` as minimal fixture

## Open Questions

- **Groovebox:** RCM dep (min build 425) — likely the JS/Wasm VM runtime executing QCode
- **mixer/mix-rs:** Rust component — likely native VM runtime that Omixrun FFIs into
- **Go wrapper:** `extapi/go/` used as submodule in `amp` — unexplored
