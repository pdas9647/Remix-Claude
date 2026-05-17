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

## PR Archive — through May 8, 2026

★ Feb–Mar: #2226 blob HTTP stdlib; #2263 zlib; #2277 exe-only .remix; #2182 remove domSym.mix; #2265 AST MatchPatterns; #1901 post-parse pseudo primitives; #2075 signals; #2294 variant:*; #2282 $loadHook for REPL; #2045 db.asOf; #2270 build server v2.1; #2242 errorInfo runtime.json; #2307 viewstack refactor (split currentMainView/currentTopView); #2308 file.upload; #2309 messaging.subscribe in agents; #2100 stdlib .dox (24 files); #2299 blob get; #2302 pin Docker v1.43.

★ Apr–May 8: #2267 repository pull (long-lived Feb→Apr); #2314 repository push; #2304 stdlib dox (long-lived); #2317 weak push (rehashRepoAsset, content-hash-only changes skip re-upload; `dbRemote.getArray` chunked 100/req); #2316 CI test_wasm decoupled from update-users; #2315 rcm-lock bump (groovebox 3402, mixrun 11215, mixer 11211); #2322 `~expose_internals` required (was optional, defaulted false); #2319 wasicaml 133→148 + libpthread stub.

## Recent PRs — May 11–16, 2026 (8 merged)

★ Key: #2330 final rcm2→rcm rename across .circleci/config.yml + Makefile + OMakefile + bin/*.sh + WrapperCommon.ml + wasm-build/OMakefile (drops `authorize-gcloud` script call); #2321 `locationDetails` metadata.

| PR          | Date   | Summary                                                                                                                                                                                                                                                                                                                                                                                                                                                                | Author        |
|-------------|--------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|---------------|
| **#2330** ★ | May 15 | **rcm2 switch (final)** — Renames `rcm2` binary references back to `rcm` (the new tool now takes over the original name). Removes the transitional `rcm script authorize-gcloud RCM_CI_READER_KEY rmx-artifacts` step. Affects CI config, Makefile, OMakefile, test shell scripts, WrapperCommon.ml `extra_code_from_rcm`, wasm-build/OMakefile. | gerdstolpmann |
| **#2321** ★ | May 13 | **locationDetails metadata** — Screens can declare `_meta_ {modType: "screen", locationDetails: "..."}` and the VM emits `$location` messages with an extra `details` field carrying that string. `output_loc` callback signature gains a 3rd `data` arg; new test055 in T_phrase.ml. Doc added to viewstack.dox. | gerdstolpmann |
| #2325       | May 13 | **rcm2 from image** — Installs rcm2 from container image instead of GCS direct download. Predecessor to #2330. | gerdstolpmann |
| #2320       | May 13 | **test with rcm2** (draft, merged anyway) — Initial rcm2 transition work. Superseded by #2325/#2330. | gerdstolpmann |
| #2326       | May 12 | **`rcm2 install -g` instead of gsutil** (merged into gerd/rcm2-image branch) — Replaces manual gsutil with rcm2's own install. | cvermilion |
| #2328       | May 15 | **Drop physical module name after module resolution** — Follow-up fix to #2327; ensures resolved module name doesn't leak into LoadHook input. | gerdstolpmann |
| #2327       | May 15 | **Fix "LoadHook: bad module name" in some tests** — Module name normalization fix. | gerdstolpmann |
| #2329       | May 15 | **Fix publish script** — npm publish tooling fix. | gerdstolpmann |
