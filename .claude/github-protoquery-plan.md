# protoquery — Exploration Plan

**Repository:** https://github.com/remixlabs/protoquery
**Branch:** main
**Stack:** OCaml (primary compiler), Mix (target language — `.mix`/`.dox` files), C (FFI), JavaScript/Webpack (worker & starter packages), WebAssembly (cross-compilation target)
**Build tools:** OMake (`OMakefile`/`OMakeroot`), OASIS (`_oasis`), Make, OPAM, npm, CircleCI
**Structure style:** Compiler/runtime monorepo — Mix language compiler (OCaml), standard library (Mix), cross-compilation backends (C/JS/Wasm), VM runtime, REPL client, server, and worker components
**Total files:** 725

---

## Layer 1 — Skeleton & Overview (must read)

Top-level manifests, build config, CI, and deployment:

- `README.md`
- `_oasis` — OASIS project descriptor (defines libraries, executables, dependencies)
- `OMakefile` — Top-level OMake build file
- `OMakeroot` — OMake root config
- `Makefile` — Top-level Make targets
- `protoquery.opam` — OPAM package definition
- `protoquery.opam.locked` — Locked OPAM dependencies
- `setup.ml` — OASIS setup script
- `.circleci/config.yml` — CI pipeline
- `.gitignore`
- `.merlin` — OCaml editor config (Merlin)
- `rcm.config.mjs` — RCM (Remix Config Manager?) config
- `rcm-lock.json` — RCM lock file
- `deploy/websocket-server/Dockerfile` — WebSocket server Docker image
- `deploy/websocket-server/build-and-push-image` — Deploy script
- `data.json` — Top-level data (purpose TBD)
- `package_pub.json` — Published npm package config
- `webpack.config.js` — Top-level Webpack config

## Layer 2 — Entry Points & API Surface (high priority)

### Executable entry points (`progs/`)

- `progs/Mixc_main.ml` — **`mixc` compiler main** (primary entry point)
- `progs/Mixc_local_main.ml` — Local mixc variant
- `progs/Mixc_server_main.ml` — Compiler server main
- `progs/Mixc_websocket_main.ml` — WebSocket server main
- `progs/Mix_client_main.ml` — Mix REPL client main
- `progs/Mix_migrate_main.ml` — Migration tool main
- `progs/Mixinfo_main.ml` — Mix info tool main
- `progs/Worker_main.ml` — Worker process main
- `progs/Unittests.ml` — Unit test runner
- `progs/Unittests_mixrun.ml` — Mixrun unit test runner
- `progs/Cross_check_main.ml` — Cross-compilation checker
- `progs/OMakefile` — Build rules for executables

### CLI entry points (`bin/`)

- `bin/mixc` — mixc compiler CLI wrapper
- `bin/mix_client` — Mix REPL client CLI wrapper

### External API surface (`extapi/`)

- `extapi/C/include/protoquery.h` — **C API header** (public ABI)
- `extapi/C/src/protoquery_api.c` — C API implementation
- `extapi/C/src/protoquery_init.ml` — OCaml init for C API
- `extapi/C/src/minidriver.c` — Minimal C driver
- `extapi/mixc-starter/src/index.js` — **npm `mixc-starter` entry** (JS API)
- `extapi/mixc-starter/src/common.js` — Shared JS utilities
- `extapi/mixc-starter/src/node.js` — Node.js-specific entry

### Top-level src build config

- `src/OMakefile` — Source tree build rules

## Layer 3 — Core Modules (medium-high priority)

### 3A — Compiler Core (`src/mix/modules/`)

The heart of the Mix compiler — lexer, parser, AST, type system, code generation:

- `src/mix/Mix.mli` — **Public interface** for the Mix library
- `src/mix/OMakefile`
- `src/mix/META`
- `src/mix/modules/OMakefile`
- `src/mix/modules/Lexer.mll` — Lexer definition (ocamllex)
- `src/mix/modules/Token.mly` — Token definitions (ocamlyacc)
- `src/mix/modules/Parser.mly` — **Parser** (ocamlyacc)
- `src/mix/modules/Parser_ast.mly` — AST parser variant
- `src/mix/modules/Parser_wrapped.ml` — Parser wrapper
- `src/mix/modules/Lexer_driver.ml` — Lexer driver
- `src/mix/modules/Syntax.ml` — Syntax definitions / AST types
- `src/mix/modules/Parsing_ast.ml` — AST parsing utilities
- `src/mix/modules/Types_ast.ml` — Type AST definitions
- `src/mix/modules/Types.ml` — **Type system**
- `src/mix/modules/Typing.ml` — **Type checker / inference**
- `src/mix/modules/Typutil.ml` — Typing utilities
- `src/mix/modules/Typextract.ml` — Type extraction
- `src/mix/modules/Compiler.ml` — **Compiler** (Mix → QCode)
- `src/mix/modules/QCode.ml` — QCode IR definitions
- `src/mix/modules/QCodeUtil.ml` — QCode utilities
- `src/mix/modules/QCodeEliminate.ml` — QCode dead code elimination
- `src/mix/modules/QCodec.ml` — QCode codec (serialization)
- `src/mix/modules/QCodec_spec.ml` — QCode codec spec
- `src/mix/modules/QCodec_types.ml` — QCode codec types
- `src/mix/modules/QCodec_helper.ml` — QCode codec helpers
- `src/mix/modules/QCodec_generator.ml` — QCode codec generator
- `src/mix/modules/RmxCodec.ml` — Remix codec
- `src/mix/modules/Linker.ml` — **Linker**
- `src/mix/modules/Transform.ml` — AST transforms
- `src/mix/modules/Transform_query.ml` — Query transform
- `src/mix/modules/Ast_transform.ml` — AST transformation pass
- `src/mix/modules/Transmod.ml` — Module transformation
- `src/mix/modules/Transmod_opt.ml` — Module transform optimization
- `src/mix/modules/Transopt.ml` — General transform optimization
- `src/mix/modules/Transpattern.ml` — Pattern matching transform
- `src/mix/modules/Translate.ml` — Translation pass
- `src/mix/modules/Sheet.ml` — Sheet compilation
- `src/mix/modules/Sheet_opt.ml` / `Sheet_opt2.ml` / `Sheet_opt3.ml` — Sheet optimizations
- `src/mix/modules/CUL.ml` — CUL intermediate representation
- `src/mix/modules/CUL_emit.ml` — CUL emission
- `src/mix/modules/CUL_opt.ml` — CUL optimization
- `src/mix/modules/CUL_print.ml` — CUL pretty printer
- `src/mix/modules/Closure.ml` — Closure conversion
- `src/mix/modules/Action.ml` — Action compilation
- `src/mix/modules/Special_forms.ml` — Special form handling
- `src/mix/modules/Lookup.ml` — Name lookup / resolution
- `src/mix/modules/Value.ml` — Value representation
- `src/mix/modules/VMSupport.ml` — VM support / builtins bridge
- `src/mix/modules/Builtin.ml` — Built-in functions
- `src/mix/modules/MixStdlib.ml` — Standard library integration
- `src/mix/modules/Hints.ml` — Compiler hints
- `src/mix/modules/Warnings.ml` — Warning system
- `src/mix/modules/Errors.ml` — Error reporting
- `src/mix/modules/Constants.ml` — Compiler constants
- `src/mix/modules/Universe.ml` — Universe / module system
- `src/mix/modules/Meta.ml` — Meta information
- `src/mix/modules/Match_set.ml` — Match set (pattern matching)
- `src/mix/modules/Map_ext.ml` — Extended map
- `src/mix/modules/Json.ml` — JSON utilities
- `src/mix/modules/MsgPack.ml` — MsgPack codec
- `src/mix/modules/Print.ml` — Pretty printing
- `src/mix/modules/Util.ml` — Compiler utilities
- `src/mix/modules/DomSym_generator.ml` — DOM symbol generator

### 3B — Cross-Compilation Backends (`src/mix-cross/`)

- `src/mix-cross/Mix_cross.mli` — Cross-compilation public interface
- `src/mix-cross/OMakefile`
- `src/mix-cross/META`
- `src/mix-cross/modules/OMakefile`
- `src/mix-cross/modules/Driver.ml` — **Cross-compilation driver**
- `src/mix-cross/modules/C.ml` — C code generation
- `src/mix-cross/modules/C_types.ml` — C type mapping
- `src/mix-cross/modules/ToC.ml` — Mix → C translation
- `src/mix-cross/modules/Js.ml` — JS code generation
- `src/mix-cross/modules/ToJs.ml` — Mix → JS translation
- `src/mix-cross/modules/ToAsm.ml` — Mix → assembly/Wasm
- `src/mix-cross/modules/MicroCode.ml` — MicroCode IR
- `src/mix-cross/modules/ToMicroCode.ml` — Mix → MicroCode
- `src/mix-cross/modules/ABI.ml` — ABI handling
- `src/mix-cross/modules/Builtins.ml` — Cross-compilation builtins
- `src/mix-cross/modules/Data.ml` — Data layout
- `src/mix-cross/modules/Encode.ml` — Encoding
- `src/mix-cross/modules/Extract_C.ml` — C extraction
- `src/mix-cross/modules/Free.ml` / `Free.mli` — Free variable analysis
- `src/mix-cross/modules/GenBuiltins.ml` — Builtin generation
- `src/mix-cross/modules/LinStack.ml` — Linear stack management
- `src/mix-cross/modules/Mixrt.ml` — Runtime support
- `src/mix-cross/modules/Util.ml` — Cross-compilation utilities

### 3C — VM Runtime (`src/mixrun/`)

- `src/mixrun/Mixrun.mli` — Runtime public interface
- `src/mixrun/OMakefile`
- `src/mixrun/META`
- `src/mixrun/modules/OMakefile`
- `src/mixrun/modules/Omixrun.ml` — **Runtime core**
- `src/mixrun/modules/Omixrunc.ml` — Runtime compiler bridge
- `src/mixrun/modules/Omixrunc_gen.ml` — Generated runtime bridge
- `src/mixrun/mytrap.c` — Signal trap handler (C)

### 3D — Server & Networking

- `src/mix-server/Mix_server.mli`
- `src/mix-server/OMakefile`
- `src/mix-server/META`
- `src/mix-server/modules/Server.ml` — **Mix server**
- `src/mix-server/modules/Worker.ml` — Server worker
- `src/mix-server/modules/ABI.ml` — Server ABI
- `src/mix-server/modules/Msgpack_server.ml` — MsgPack server protocol
- `src/mix-server/modules/OMakefile`
- `src/mix-websocket/modules/Websocket_server.ml` — **WebSocket server**
- `src/mix-websocket/modules/Websocket_auth.ml` — WebSocket auth
- `src/mix-websocket/modules/Websocket_codegen.ml` — WebSocket codegen
- `src/mix-websocket/modules/OMakefile`
- `src/mix-websocket/OMakefile`

### 3E — Amp Client (REPL/MQTT) (`src/amp-client/`)

- `src/amp-client/Amp_client.mli`
- `src/amp-client/OMakefile`
- `src/amp-client/META`
- `src/amp-client/modules/Client.ml` — **Amp client**
- `src/amp-client/modules/Code.ml` — Code handling
- `src/amp-client/modules/CommonREPL.ml` — Common REPL
- `src/amp-client/modules/InternalREPL.ml` — Internal REPL
- `src/amp-client/modules/MixedREPL.ml` — Mixed mode REPL
- `src/amp-client/modules/MQTT.ml` — MQTT protocol
- `src/amp-client/modules/MQTTProto.ml` — MQTT protocol impl
- `src/amp-client/modules/MQTTREPL.ml` — MQTT REPL
- `src/amp-client/modules/Docs.ml` — Documentation
- `src/amp-client/modules/Testdriver.ml` — Test driver
- `src/amp-client/modules/Track.ml` — Tracking
- `src/amp-client/modules/OMakefile`

## Layer 4 — Data & Config (medium priority)

### 4A — Mix Standard Library (`src/mix/stdlib/`)

Each module has a `.mix` (implementation) and `.dox` (documentation):

- `src/mix/stdlib/stdlib.mix` + `stdlib.dox` — Stdlib root / prelude
- `src/mix/stdlib/stdlib_trailer.mix` — Stdlib trailer
- `src/mix/stdlib/array.mix` + `array.dox`
- `src/mix/stdlib/bool.mix` + `bool.dox`
- `src/mix/stdlib/number.mix` + `number.dox`
- `src/mix/stdlib/string.mix` + `string.dox`
- `src/mix/stdlib/option.mix` + `option.dox`
- `src/mix/stdlib/result.mix` + `result.dox`
- `src/mix/stdlib/map.mix` + `map.dox`
- `src/mix/stdlib/set.mix` + `set.dox`
- `src/mix/stdlib/gset.mix` + `gset.dox`
- `src/mix/stdlib/bitset.mix` + `bitset.dox`
- `src/mix/stdlib/uniqArray.mix` + `uniqArray.dox`
- `src/mix/stdlib/circuitarray.mix` + `circuitarray.dox`
- `src/mix/stdlib/db.mix` + `db.dox`
- `src/mix/stdlib/dbRemote.mix` + `dbRemote.dox`
- `src/mix/stdlib/query.mix` + `query.dox`
- `src/mix/stdlib/http.mix` + `http.dox`
- `src/mix/stdlib/file.mix` + `file.dox`
- `src/mix/stdlib/binary.mix` + `binary.dox`
- `src/mix/stdlib/crypto.mix` + `crypto.dox`
- `src/mix/stdlib/wasm.mix` + `wasm.dox`
- `src/mix/stdlib/mime.mix` + `mime.dox`
- `src/mix/stdlib/stream.mix` + `stream.dox`
- `src/mix/stdlib/xml.mix` (in `src/mix/parsing/`) + `xml.dox`
- `src/mix/stdlib/websocket.mix` + `websocket.dox`
- `src/mix/stdlib/dom.mix` + `dom.dox`
- `src/mix/stdlib/domUtil.mix` + `domUtil.dox`
- `src/mix/stdlib/domSym.dox` (generated, no .mix)
- `src/mix/stdlib/reflect.mix` + `reflect.dox`
- `src/mix/stdlib/editorTypes.mix` + `editorTypes.dox`
- `src/mix/stdlib/editorNode.mix` + `editorNode.dox`
- `src/mix/stdlib/editorScreen.mix` + `editorScreen.dox`
- `src/mix/stdlib/container.mix` + `container.dox`
- `src/mix/stdlib/viewstack.mix` + `viewstack.dox`
- `src/mix/stdlib/debugger.mix` + `debugger.dox`
- `src/mix/stdlib/debuggerBreakpoints.mix` + `debuggerBreakpoints.dox`
- `src/mix/stdlib/testing.mix` + `testing.dox`
- `src/mix/stdlib/compiler.mix` + `compiler.dox`
- `src/mix/stdlib/vm.mix` + `vm.dox`
- `src/mix/stdlib/agent.mix` + `agent.dox`
- `src/mix/stdlib/auth.mix` + `auth.dox`
- `src/mix/stdlib/env.mix` + `env.dox`
- `src/mix/stdlib/secrets.mix` + `secrets.dox`
- `src/mix/stdlib/logging.mix` + `logging.dox`
- `src/mix/stdlib/metrics.mix` + `metrics.dox`
- `src/mix/stdlib/co.mix` + `co.dox`
- `src/mix/stdlib/messaging.mix` + `messaging.dox`
- `src/mix/stdlib/loader.mix` + `loader.dox`
- `src/mix/stdlib/calendar.mix` + `calendar.dox`
- `src/mix/stdlib/color.mix` + `color.dox`
- `src/mix/stdlib/random.mix` + `random.dox`
- `src/mix/stdlib/regex.mix` + `regex.dox`
- `src/mix/stdlib/memo.mix` + `memo.dox`
- `src/mix/stdlib/util.mix` + `util.dox`
- `src/mix/stdlib/action.mix` + `action.dox`
- `src/mix/stdlib/apps.mix` + `apps.dox`
- `src/mix/stdlib/builder.mix` + `builder.dox`
- `src/mix/stdlib/cells.mix` + `cells.dox`
- `src/mix/stdlib/currency.mix` + `currency.dox`
- `src/mix/stdlib/macro.mix` + `macro.dox`
- `src/mix/stdlib/triggeredComponent.mix` + `triggeredComponent.dox`

### 4B — Compiler Driver & Agents (Mix source)

- `src/mix/driver/compilerDriver.mix` — Session orchestration
- `src/mix/driver/compilerBuild.mix` — High-level build pipeline
- `src/mix/driver/compilerRule.mix` — Build rules
- `src/mix/driver/compilerFile.mix` — File compilation
- `src/mix/driver/compilerLib.mix` — Library compilation
- `src/mix/driver/compilerExe.mix` — Executable compilation
- `src/mix/driver/compilerREPL.mix` — REPL send-to-VM
- `src/mix/driver/compilerClean.mix` — Clean operations
- `src/mix/driver/compilerExtra.mix` — Extra compiler features
- `src/mix/driver/compilerTarget.mix` — Target selection
- `src/mix/driver/compilerWebsocket.mix` — WebSocket compiler
- `src/mix/driver/codegenDriver.mix` — Code generation driver
- `src/mix/driver/codegenData.mix` — Code generation data
- `src/mix/driver/codegenUtil.mix` — Code generation utilities
- `src/mix/driver/remixFile.mix` — .remix file builder
- `src/mix/driver/remixFile_v2.mix` — .remix file builder v2
- `src/mix/driver/remixFileExport.mix` — .remix export
- `src/mix/driver/remixRecipe.mix` — Declarative recipe types
- `src/mix/driver/OMakefile`
- `src/mix/agents/_rmx_sessionAgent.mix` — Session agent
- `src/mix/agents/_rmx_agentAgent.mix` — Agent agent
- `src/mix/agents/_rmx_makeAgent.mix` — Build/deploy agent
- `src/mix/agents/state.mix` — Agent state
- `src/mix/agents/OMakefile`

### 4C — Slate Editor Components

- `src/mix/slate/slateDef.mix` + `slateDef.dox`
- `src/mix/slate/slateDyn.mix` + `slateDyn.dox`
- `src/mix/slate/slateDynRun.mix` + `slateDynRun.dox`
- `src/mix/slate/slateGen.mix` + `slateGen.dox`
- `src/mix/slate/slateComponent.mix` + `slateComponent.dox`
- `src/mix/slate/slateInstantiate.mix` + `slateInstantiate.dox`
- `src/mix/slate/OMakefile`

### 4D — JSON Web (JWT/JWK/JWS)

- `src/mix/jsonweb/jwt.mix` + `jwt.dox`
- `src/mix/jsonweb/jwk.mix` + `jwk.dox`
- `src/mix/jsonweb/jws.mix` + `jws.dox`
- `src/mix/jsonweb/OMakefile`

### 4E — Parsing

- `src/mix/parsing/csv.mix`
- `src/mix/parsing/xml.mix` + `xml.dox`
- `src/mix/parsing/OMakefile`

### 4F — Special Screens

- `src/mix/stdlib/_rmx_debugShell.mix`
- `src/mix/stdlib/_rmx_errorFallback.mix`
- `src/mix/stdlib/_rmx_extension.mix` + `_rmx_extension.dox`

### 4G — ABI Versions

- `abi/abi-1.json` through `abi/abi-27.json` (27 ABI version snapshots)

### 4H — Utility Libraries

- `src/bigstring/Bigstring.mli`
- `src/bigstring/modules/Buffer.ml`
- `src/bigstring/modules/I.ml`
- `src/bigstring/bigstring_ffi.c`
- `src/crc32/Crc32.ml` + `Crc32.mli`
- `src/crc32/crc32_ffi.c`
- `src/mix-misc/Mix_misc.mli`
- `src/mix-misc/modules/Util.ml`
- `src/infradoc/Infraparse.ml`

### 4I — JS Workers & Starter

- `extapi/mixc-starter/mix_vm_mixc_start`
- `extapi/mixc-starter/Makefile`
- `extapi/mixc-starter/src/package.json`
- `extapi/mixc-starter/example/index.html`
- `extapi/mixc-starter/example/index.js`
- `extapi/mixc-starter/example/package.json`
- `extapi/mixc-starter/example/webpack.config.js`
- `src_cmdline/start.mjs`
- `mixc-worker/src/worker.js`
- `mixc-worker/src/node.js`
- `mixc-worker/src/package.json`
- `mixc-worker/src/webpack.config.js`
- `mixc-worker/Makefile`
- `mixc-worker-c/worker.c`
- `mixc-worker-c/worker.h`

### 4J — Documentation Generator

- `src/mixdoc/Dox_main.ml`
- `src/mixdoc/Dox_out.ml`
- `src/mixdoc/Dox_parse.ml`
- `src/mixdoc/Doxtypes.ml`
- `src/mixdoc/OMakefile`

### 4K — Wasm Build

- `wasm-build/Makefile`
- `wasm-build/OMakefile`
- `wasm-build/rcm.config.mjs`
- `wasm-build/unittests_mixrun_wasm`

## Layer 5 — Supporting (low priority, skip if budget tight)

### 5A — Test Libraries

- `src/tlib_mix/T_*.ml` (28 test files — AST, typing, exec, driver, stdlib, etc.)
- `src/tlib_mix/OMakefile`
- `src/tlib_tapsim/T_tapsim.ml`
- `src/tlib_tapsim/OMakefile`
- `src/tlib_wrapper_amp/WrapperAmp.ml`
- `src/tlib_wrapper_common/WrapperCommon.ml`
- `src/tlib_wrapper_cross/WrapperCross.ml`
- `src/tlib_wrapper_mixrun/WrapperMixrun.ml`
- `src/mix-cross-test/modules/Testdriver.ml`
- `src/mix-cross-test/modules/Testdriver_proto.ml`
- `src/testing/T_frame.ml`
- `src/tester.ml`

### 5B — Test Data & Fixtures

- `testdata/hello/hello.mix`
- `testdata/foofication/main.mix`
- `testdata/libA/`, `testdata/libB/`, `testdata/libC/`
- `testdata/libfoo/`
- `testdata/loadAB/`, `testdata/loadAB_sameDB/`, `testdata/loadC/`, `testdata/loadC_sameDB/`
- `testdata/twice/`
- `testdata/mixdoc/templates/`
- `src/tlib_tapsim/t001_*` through `t008_*` (test Mix programs + PNGs)

### 5C — Examples & Gists

- `examples/README.md`
- `examples/Makefile`
- `examples/common/ui.mix`
- `examples/counter/counter.mix` + `ui.mix`
- `mix-gist/circuitarray/form.mix` + `form_row.mix`

### 5D — Shell Test Scripts

- `bin/amptest.sh`
- `bin/amp_mixc_test.sh`
- `bin/amp_rebuild_test.sh` + `bin/amp_rebuild_test.d/rebuild.mix`
- `bin/mixrun_test.sh`
- `bin/mixrun_rebuild_test.sh` + `bin/mixrun_rebuild_test.d/rebuild.mix`
- `bin/cross_test.sh`
- `bin/soundcheck_test.sh`

### 5E — CI & Misc Scripts

- `ci/ensure-gsutil-creds-internal`
- `ci/publish-npm-pkg`
- `ci/setup-npm`
- `config-scripts/test_functorpack`
- `progs/mixdoc`
- `progs/mix_testing_get`
- `progs/mix_testing_put`
- `progs/mixc_server_wasm`
- `progs/get-wasicaml`
- `progs/mosquitto.conf` + `mosquitto.sh`
- `progs/Oasistogv.ml`
- `progs/Moduleimport2gv.ml`
- `progs/json_to_qcode_v5.ml`
- `progs/dummy_location.c`

## Excluded (do not read)

- `_oasis_build.om`, `_oasis_hier.om`, `_oasis_install.om` — OASIS-generated build scaffolding (repeated in every directory, ~60+ files)
- `setup.log.om` — Build log
- `abi/abi-*.json` (read only latest `abi-27.json` if ABI understanding needed)
- `*.png` files in `src/tlib_tapsim/` — Test screenshot fixtures
- `extapi/mixc-starter/example/package-lock.json`
- `extapi/mixc-starter/src/package-lock.json`
- `mixc-worker/src/package-lock.json`
- `src/mix-cross/modules/abi_mixrt_exports.txt`, `abi_mixrt_imports.txt` — Generated ABI lists

---

## Reading Strategy

1. **Start with Layer 1** — understand build system, dependencies, project shape
2. **Read `src/mix/Mix.mli`** — the public interface tells you everything the compiler exposes
3. **Read entry points** (`Mixc_main.ml`, `Mix_client_main.ml`) to see how components connect
4. **Dive into compiler pipeline**: Lexer → Parser → Syntax/AST → Types/Typing → Compiler → QCode → Linker
5. **Cross-compilation** if relevant: `src/mix-cross/` (C, JS, Wasm backends)
6. **Standard library** as needed per topic (each `.mix` is self-contained)
7. **Driver/build system** for understanding `.remix` file generation and build orchestration
