# protoquery

**Repository:** https://github.com/remixlabs/protoquery
**Stack:** OCaml 4.12.1, Mix (.mix/.dox), C (FFI), JS/Webpack (Wasm worker), WebAssembly
**Purpose:** Mix language compiler, standard library, VM runtime bridge, compiler server
**Explored:** L1–L5 (all layers)

## Build & Deploy

- OASIS + OMake + opam; `make deps-install && make` → binaries in `progs/`
- CI: CircleCI — amd64, arm64, macOS; static/dynamic/Wasm/Groovebox/rebuild tests
- Artifacts: `mixc`, `mix_client`, `mix_migrate`, `mixc_server`; `.mixlib`/`.mixexe` + `.wasm`
- Deploy: GCS via `rcm comp publish`; Docker (Debian + Node 22, port 8000); npm (mixc-starter, mixc-worker)
- ABI: Currently 27. ~45 opcodes + typed builtin catalog. Wasm build via wasicaml (OCaml → Wasm).

## Compilation Pipeline

```
Source (.mix) → Syntax.ml (Lexer+Parser) → AST
  → Typing.ml (Hindley-Milner) → checked phrases
  → Translate.ml → QCode (register-based bytecode, max 1023 instr/section)
  → Linker.ml (global/builtin/foreign remapping) → linked iqccontainer
  → Omixrun.ml (native VM via ctypes)  OR  Driver.ml → ToAsm.ml → .wasm
```

**Transports:** stdio pipes (`mixc -server`, length-prefixed binary IPC), WebSocket (SCRAM auth, session tokens), Wasm Worker (browser/Node). JS bootstrap: `mixc-starter` spawns Wasm worker, pub/sub
on `/org/{org}/ws/{ws}/mixc`.

## Executables

| Binary           | Purpose                                                       |
|------------------|---------------------------------------------------------------|
| `mixc`           | Primary compiler CLI (local, remote, REPL)                    |
| `mixc_server`    | Stdin/stdout compiler server                                  |
| `mixc_websocket` | WebSocket compiler server (Docker)                            |
| `mix_client`     | Interactive REPL (#mark, #load, #push, #pop, #invoke, #debug) |
| `mixdoc`         | Doc generator (.mix+.dox → JSON/Markdown)                     |

C API: `protoquery_init()`/`protoquery_compile()` for C/Go embedding (Go wrapper in `extapi/go/`, used by `amp`).

## Server Protocol

**Requests:** Create/Destroy_session, Send_libraries, Check_source, Imports, Compile_modules, Compile_phrases, Request_code (QCODE/WASM), Send_code, Get_stdlib/id, To_json/binary, About, Statistics. *
*Response:** session + success + messages + typed payload.

## OCaml Libraries

| Library                            | Purpose                                           |
|------------------------------------|---------------------------------------------------|
| **Mix** (`src/mix/`)               | Compiler core: 36 modules                         |
| **Mixrun** (`src/mixrun/`)         | Native VM bridge (ctypes FFI)                     |
| **Mix_cross** (`src/mix-cross/`)   | Cross-compilation: Wasm (ToAsm), C (ToC), ABI     |
| **Mix_server** (`src/mix-server/`) | Protocol handler, Worker, ABI, MsgPack            |
| **Mix_websocket**                  | WebSocket transport (lwt, SCRAM auth)             |
| **Amp_client**                     | AMP client: HTTP/MQTT, REPL, Track/Docs/Code APIs |

## Key Compiler Modules

- **Types.ml** — `data` (13 variants), `value`, `type_term` (Tany/Tbool/Tnumber/Tfun/Tarray/Trecord/Tcases/Tvar/Tuser/Tmodule…)
- **Syntax.ml** — Lexing + parsing; `parse_phrase_string`, `parse_module`, `dependencies`
- **Typing.ml** — Hindley-Milner with cells, streams, modules
- **Translate.ml** — AST → QCode; max 1023 instr/section (10-bit valueOrigin)
- **QCode.ml** — Bytecode IR: ~25 opcodes, terminators (Qjump/Qreturn/Qapply/Qforeign/Qcoroutine2/Qsuspend/Qcopylib)
- **Linker.ml** — Library linking: global/builtin/foreign remapping, LibQCode/LibWasm

## Mix Standard Library (`src/mix/stdlib/`)

~55 modules (.mix + .dox pairs):

- **stdlib.mix** — Prelude: option/result/mutable/lazy, reactive spreadsheet engine (cells/sheets/links), module registry, host-env dispatch, action system
- **db.mix** — 5 scopes (mainDB/appDB/sessionInMemDB/globalInMemDB/remoteDB), query pipeline, live subscriptions via watermark change detection
- **compiler.mix** — Artifact model (library/executable records), libID format (`<name>-<version>@<hash>`), channel resolution
- **agent.mix** — 3 call paths: subroutineCall (same-session), localAgentCall (pubsub), httpCall (REST); HMAC origin verification
- Other: core, collections, I/O, streaming, editor/DOM, runtime, platform (auth/env/secrets/co/messaging/loader)

## Compiler Driver (`src/mix/driver/`)

- **compilerDriver.mix** — Session orchestrator: create/destroy/compile/publishLibrary; result monad; dual transport (MQTT or websocket)
- **remixFile.mix** — .remix archive builder (zip): manifest.json + runtime.json + appMeta; staging DB → `$db_zipWrite` FFI

## Key Patterns

- Session-based compilation with incremental code accumulation
- Dual serialization: JSON (Yojson) for messages, MsgPack for binary, custom binary for QCode
- PPX derivers: `ppx_deriving_yojson`, `ppx_deriving.show`, `ppx_blob`, `ppx_getenv`
- Mix stdlib: `$twelve` for type-erasure casts, `_inline_` pragmas, `$DEFPRIMITIVES` for builtins
- Tests: 28+ `T_*.ml` modules; shell integration tests; rebuild vs live envs

## PR Archive — Feb 22 – Mar 21, 2026

★ Key: #2226 blob HTTP stdlib; #2263 zlib; #2277 exe-only .remix; #2182 remove domSym.mix; #2265 AST MatchPatterns; #1901 post-parse pseudo primitives; #2075 signals module; #2294 variant:*; #2282 $
loadHook for REPL; #2045 db.asOf; #2270 build server v2.1; #2242 errorInfo runtime.json.

## Recent PRs — Mar 22–31, 2026 (6 merged)

| PR          | Summary                                                                                                                                                                                                                                                   | Author     |
|-------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|------------|
| **#2307** ★ | **Viewstack refactor** (6 files, +314/-99) — splits `currentMainView` from `currentTopView`; adds `topViewName/Id`, `isViewAtTop`, `isScreenView`, `$vs_pendingProps`; `internal` moved to coroutine state; hidden sheets no longer activated by actions. | Gerd       |
| **#2308**   | **`file.upload` stdlib helper** (+94/-5) — PUT upload with content-type, Cache-Control, OAuth token, overwrite guard (412), retry on 409; docs updated for `_rmx_upload_cache_control`.                                                                   | Gerd       |
| **#2309**   | **`messaging.subscribe` in agents** (+3/-2) — subscribe allowed when `env.isAgent == true`.                                                                                                                                                               | Gerd       |
| **#2100** ★ | **Stdlib .dox docs** (24 files, +3211/-92) — adds gset/loader/logging/macro/memo/metrics/mime/option/reflect/regex/util; removes dom/domSym/domUtil; CI ships per-lib JSON to GCS.                                                                        | Gerd       |
| **#2299**   | **Blob get helpers** (+22/-13) — `builder.blobGetBinary/String/Json` over `blobGet`.                                                                                                                                                                      | simonh1000 |
| **#2302**   | **Pin Docker API v1.43** (+2) — CI: `DOCKER_API_VERSION: 1.43`.                                                                                                                                                                                           | Chris      |

## Recent PRs — Apr 18–25, 2026 (3 merged)

| PR          | Summary                                                                                                                                                                                                   | Author        |
|-------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|---------------|
| **#2267** ★ | **repository library — pull support** (28 commits, 16 files, +3964/-6) — New `repository` extra lib: `repoTypes`, `repoID`, `repoEditor`, `repoUtil`, `repoCall`, `repoPull`. Long-lived (Feb 13→Apr 21). | gerdstolpmann |
| **#2314** ★ | **repository library — push support** (10 commits, 9 files, +670/-41) — Follow-up to #2267; adds push capability to the repository library.                                                               | gerdstolpmann |
| **#2304**   | **more documentation** (7 commits, 7 files, +1492/-196) — Stdlib dox additions (long-lived Mar 24→Apr 21).                                                                                                | gerdstolpmann |

## Recent PRs — Apr 26 – Apr 28, 2026 (3 merged)

| PR          | Date   | Summary                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 | Author        |
|-------------|--------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|---------------|
| **#2317** ★ | Apr 28 | **Weak push (rehash-only changes)** — Adds `rehashRepoAsset` to `repoTypes.pushType`; `changeInfo` gains `changeContentHash: option(string)`. `repoUtil.compareHeadWithDatabase` now detects when a head asset's hash matches a `legacyContentHash` on the DB asset and classifies it as `rehashRepoAsset` (no full data re-upload needed). `dbRemote.getArray` refactored into chunked `getArrayChunk` (100-record limit per request) + `slice` helper; `getStream` uses `stream.flatMap` over chunks. | gerdstolpmann |
| **#2316**   | Apr 28 | **CI: decouple test_wasm from update-users** — `test_wasm` no longer a prerequisite for the `update-users` CI job.                                                                                                                                                                                                                                                                                                                                                                                      | gerdstolpmann |
| **#2315**   | Apr 27 | **rcm fixup** — `rcm-lock.json` bump: groovebox 3356→3402, mixrun 11083→11215, mixer 11083→11211.                                                                                                                                                                                                                                                                                                                                                                                                       | gerdstolpmann |
