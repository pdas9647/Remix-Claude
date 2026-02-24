# mix-rs — Exploration Plan

**Repository:** https://github.com/remixlabs/mix-rs
**Branch:** main
**Stack:** Rust (primary), TypeScript/Vite (desktop frontend), C (FFI/wasm2c), Java/JNI, Swift, Kotlin (mobile bindings), Docker
**Structure style:** Cargo workspace monorepo — 7 crates + desktop Tauri app
**Total files:** 708

### Crate Map

| Crate         | Path           | Role                                                                                                                |
|---------------|----------------|---------------------------------------------------------------------------------------------------------------------|
| `remix`       | `remix/`       | Core types: value system, codec (ser/de), builder metadata, intern (qcode, DB, handles), stdlib                     |
| `remix_macro` | `remix_macro/` | Rust proc macros for the workspace                                                                                  |
| `mixcore`     | `mixcore/`     | Mix VM core: FFI bridge, action dispatch, file I/O, HTTP, crypto, DuckDB, Wasm, WebSocket, MQTT, secrets            |
| `mixrun`      | `mixrun/`      | Mix runtime: Wasm execution (wasmtime + wasm2c backends), widget lifecycle, LRU cache, C API                        |
| `mixquery`    | `mixquery/`    | Query engine: AST, parser, query execution (filter/proj/aggr), data storage (compact/batch/index), varcode encoding |
| `mixer`       | `mixer/`       | Server + CLI: HTTP API (v0/v1), agent runtime, MCP, DB, auth, permissions, .remix file handling, SPCS auth          |
| `mixfs`       | `mixfs/`       | Filesystem abstraction: noop/std/tokio backends                                                                     |
| `desktop`     | `desktop/`     | Tauri desktop app (src-tauri + src-shared + src-aux + src-mcp-bridge), TypeScript/Vite frontend                     |

---

## Layer 1 — Skeleton & Overview (must read)

Top-level manifests, config, CI, READMEs — establishes workspace structure and build system.

```
README.md
Cargo.toml                          # workspace manifest (member list, shared deps)
Cargo.lock
Makefile                            # top-level build commands
bacon.toml                          # bacon (Rust watcher) config
rustfmt.toml                        # code style
.gitignore
.circleci/config.yml                # CI pipeline
.circleci/bin/get-remix-token       # CI auth helper
.github/ISSUE_TEMPLATE/tracking-issue.md
```

Per-crate manifests and READMEs:

```
mixcore/Cargo.toml
mixcore/README.md
mixcore/Makefile
mixcore/cbindgen.toml               # C header generation config
mixer/Cargo.toml
mixer/README.md
mixer/Makefile
mixquery/Cargo.toml
mixquery/README.md
mixquery/cbindgen.toml
mixrun/Cargo.toml
mixrun/README.md
mixrun/Makefile
mixfs/Cargo.toml
remix/Cargo.toml
remix_macro/Cargo.toml
desktop/README.md
desktop/Makefile
desktop/package.json
desktop/src-tauri/Cargo.toml
desktop/src-tauri/tauri.conf.json   # Tauri app config (name, windows, plugins)
desktop/src-tauri/tauri.macos.conf.json
desktop/src-tauri/tauri.ios.conf.json
desktop/src-tauri/tauri.windows.conf.json
desktop/src-shared/Cargo.toml
desktop/src-aux/Cargo.toml
desktop/src-mcp-bridge/Cargo.toml
```

## Layer 2 — Entry Points & API Surface (high priority)

Main binaries, lib.rs roots, server API definitions, C API headers.

```
# Crate entry points (lib.rs / main.rs)
remix/src/lib.rs                    # core types root
mixcore/src/lib.rs                  # VM core root
mixrun/src/lib.rs                   # runtime root
mixquery/src/lib.rs                 # query engine root
mixfs/src/lib.rs                    # FS abstraction root
mixer/src/lib.rs                    # server lib root
mixer/src/bin/mixer.rs              # server binary entry point
remix_macro/src/lib.rs              # proc macro root
desktop/src-tauri/src/main.rs       # Tauri app entry point
desktop/src-tauri/src/lib.rs        # Tauri lib (commands, setup)
desktop/src-mcp-bridge/src/main.rs  # MCP bridge binary
desktop/src-shared/src/lib.rs       # shared desktop lib
desktop/src-aux/src/bin/builtin-apps.rs
desktop/src-aux/src/bin/desktop-release.rs

# Server API surface
mixer/src/serve.rs                  # HTTP server setup
mixer/src/serve/api_root.rs         # API root router
mixer/src/serve/api_v0.rs           # v0 API routes
mixer/src/serve/api_v1.rs           # v1 API routes (mod)
mixer/src/serve/api_v1/db.rs        # v1 DB endpoints
mixer/src/serve/api_v1/documents.rs # v1 document CRUD
mixer/src/serve/api_v1/query.rs     # v1 query endpoint
mixer/src/serve/api_v1/save.rs      # v1 save endpoint
mixer/src/serve/api_v1/auth_configs.rs
mixer/src/serve/api_v1/auth_spcs.rs # Snowflake SPCS auth
mixer/src/serve/api_v1/permissions.rs
mixer/src/serve/api_v1/info.rs
mixer/src/serve/api_v1/app_metadata.rs
mixer/src/serve/api_v1/ext.rs

# C/FFI API headers
mixcore/include/mixcore.h
mixrun/include/mixrun.h
mixrun/include/mixrun-decls.h
mixcore/src/c_api.rs
mixrun/src/c_api.rs
mixrun/src/c_api/widgets_c.rs
mixrun/src/c_api/widgets_jni.rs
```

## Layer 3 — Core Modules (medium priority)

Domain logic: value system, codec, query engine, VM core, runtime execution, server models.

```
# remix — value system & codec
remix/src/value.rs
remix/src/value/bin.rs
remix/src/value/case.rs
remix/src/value/dbref.rs
remix/src/value/de.rs
remix/src/value/number.rs
remix/src/value/proxy.rs
remix/src/value/ser.rs
remix/src/value/stream.rs
remix/src/value/token.rs
remix/src/value/undef.rs
remix/src/value_ops.rs
remix/src/codec.rs
remix/src/codec/ann.rs
remix/src/codec/de.rs
remix/src/codec/ser.rs
remix/src/codec/ser_spe.rs
remix/src/codec/repr.rs
remix/src/codec/rmxtag.rs
remix/src/codec/rmxtype.rs
remix/src/codec/tag.rs
remix/src/codec/error.rs
remix/src/error.rs
remix/src/schema.rs

# remix — intern (qcode, DB, handles)
remix/src/intern.rs
remix/src/intern/qcode.rs
remix/src/intern/qcode/debug.rs
remix/src/intern/db.rs
remix/src/intern/code.rs
remix/src/intern/code_file.rs
remix/src/intern/code_id.rs
remix/src/intern/code_info.rs
remix/src/intern/handle.rs
remix/src/intern/mem.rs
remix/src/intern/on_ptr.rs
remix/src/intern/desc.rs
remix/src/intern/errdec.rs
remix/src/intern/base64.rs
remix/src/intern/misc.rs
remix/src/intern/remix_file.rs

# remix — builder & stdlib
remix/src/builder.rs
remix/src/builder/card.rs
remix/src/builder/type_.rs
remix/src/futures.rs
remix/src/stdlib.rs
remix/src/stdlib/calendar.rs

# mixcore — VM core
mixcore/src/core.rs
mixcore/src/action.rs
mixcore/src/code.rs
mixcore/src/code/db.rs
mixcore/src/code/error.rs
mixcore/src/code/file.rs
mixcore/src/env.rs
mixcore/src/error.rs
mixcore/src/ffi.rs
mixcore/src/ffi/core.rs
mixcore/src/ffi/blob.rs
mixcore/src/ffi_state.rs
mixcore/src/output.rs
mixcore/src/proxy.rs
mixcore/src/mounts.rs
mixcore/src/sys.rs
mixcore/src/remix_file.rs
mixcore/src/remix_file/v1.rs
mixcore/src/remix_file/v2.rs
mixcore/src/request.rs
mixcore/src/request/external.rs
mixcore/src/request/vendored.rs

# mixcore — FFI modules (ffi/ subdirectory)
mixcore/src/ffi/db.rs
mixcore/src/ffi/http.rs
mixcore/src/ffi/files.rs
mixcore/src/ffi/crypto.rs
mixcore/src/ffi/duckdb.rs
mixcore/src/ffi/wasm.rs
mixcore/src/ffi/websocket.rs
mixcore/src/ffi/mqtt.rs
mixcore/src/ffi/secrets.rs
mixcore/src/ffi/token.rs
mixcore/src/ffi/decode_error.rs

# mixcore — file handling
mixcore/src/files.rs
mixcore/src/files/memory.rs
mixcore/src/files/mix_path.rs
mixcore/src/files/regular.rs
mixcore/src/files/s3.rs

# mixrun — runtime execution
mixrun/src/mixrun.rs
mixrun/src/core.rs
mixrun/src/mixrt.rs
mixrun/src/mixrt/backend.rs
mixrun/src/mixrt/error.rs
mixrun/src/mixrt/ptr.rs
mixrun/src/wasmtime.rs
mixrun/src/wasmtime/unit_info.rs
mixrun/src/wasmtime/wasi_shim.rs
mixrun/src/wasm2c.rs
mixrun/src/wasm2c/c_api.rs
mixrun/src/wasm2c/trap.rs
mixrun/src/widgets.rs
mixrun/src/state.rs
mixrun/src/output.rs
mixrun/src/error.rs
mixrun/src/check.rs
mixrun/src/timer.rs
mixrun/src/lru_weighted.rs

# mixquery — query engine
mixquery/src/api.rs
mixquery/src/ast/query_ast.rs
mixquery/src/ast/field.rs
mixquery/src/ast/lit.rs
mixquery/src/ast/op.rs
mixquery/src/parser/expr.rs
mixquery/src/parser/lit.rs
mixquery/src/parser/search.rs
mixquery/src/query/exec.rs
mixquery/src/query/pred.rs
mixquery/src/query/proj.rs
mixquery/src/query/aggr.rs
mixquery/src/query/elem.rs
mixquery/src/query/result.rs
mixquery/src/data/base.rs
mixquery/src/data/batch.rs
mixquery/src/data/brute.rs
mixquery/src/data/compact.rs
mixquery/src/data/header.rs
mixquery/src/data/index.rs
mixquery/src/data/index_1.rs
mixquery/src/data/store.rs
mixquery/src/reader.rs
mixquery/src/data/writelog.rs
mixquery/src/varcode.rs
mixquery/src/tokens.rs
mixquery/src/mp.rs
mixquery/src/db_config.rs
mixquery/src/db_resource.rs
mixquery/src/dir_info.rs
mixquery/src/reg.rs
mixquery/src/result.rs
mixquery/src/timestamp.rs
mixquery/src/data/version.rs
mixquery/src/async_co.rs

# mixer — server core
mixer/src/serve/auth.rs
mixer/src/serve/guard.rs
mixer/src/serve/perm.rs
mixer/src/serve/state.rs
mixer/src/serve/workspace.rs
mixer/src/serve/workspace_context.rs
mixer/src/serve/run_agent.rs
mixer/src/serve/agent_alias.rs
mixer/src/serve/agent_cache.rs
mixer/src/serve/cloud_query.rs
mixer/src/serve/doc.rs
mixer/src/serve/pubsub.rs
mixer/src/serve/signals.rs
mixer/src/serve/import_export.rs
mixer/src/serve/export.rs
mixer/src/serve/s3.rs
mixer/src/serve/secret.rs
mixer/src/serve/origin.rs
mixer/src/serve/overrides.rs
mixer/src/serve/intercept.rs
mixer/src/serve/host_effects.rs
mixer/src/serve/static_files.rs
mixer/src/serve/cache_control.rs
mixer/src/serve/timeouts.rs
mixer/src/serve/logging.rs
mixer/src/serve/error.rs
mixer/src/serve/resp.rs
mixer/src/serve/utils.rs
mixer/src/serve/args.rs

# mixer — data models
mixer/src/serve/model.rs
mixer/src/serve/model/action.rs
mixer/src/serve/model/activity.rs
mixer/src/serve/model/activity_log.rs
mixer/src/serve/model/agent.rs
mixer/src/serve/model/agent_alias.rs
mixer/src/serve/model/api_resource.rs
mixer/src/serve/model/app_metadata.rs
mixer/src/serve/model/auth_config.rs
mixer/src/serve/model/bool.rs
mixer/src/serve/model/cloud_query.rs
mixer/src/serve/model/error.rs
mixer/src/serve/model/form_parse.rs
mixer/src/serve/model/grant.rs
mixer/src/serve/model/ident.rs
mixer/src/serve/model/local_agent.rs
mixer/src/serve/model/origin.rs
mixer/src/serve/model/parquet_log.rs
mixer/src/serve/model/resource.rs
mixer/src/serve/model/role.rs
mixer/src/serve/model/subject.rs
mixer/src/serve/model/token.rs

# mixer — other commands
mixer/src/run.rs
mixer/src/db.rs
mixer/src/codec.rs
mixer/src/file.rs
mixer/src/file/args.rs
mixer/src/remix_file.rs
mixer/src/mcp.rs
mixer/src/error_decode.rs
mixer/src/wasm.rs
mixer/src/wasm/args.rs
mixer/src/wasm/manifest.rs
mixer/src/wasm/rust.rs
mixer/src/wasm/example-lib.rs
mixer/src/webcomp.rs
mixer/src/webcomp/args.rs
mixer/src/shared.rs
mixer/src/shared/err_message.rs
mixer/src/args.rs

# mixfs
mixfs/src/intf.rs
mixfs/src/error.rs
mixfs/src/backends/std.rs
mixfs/src/backends/tokio.rs
mixfs/src/backends/noop.rs
```

## Layer 4 — Data & Config (medium priority)

Build scripts, Tauri configs, deployment configs, rcm configs.

```
# Build scripts
mixcore/build.rs
mixrun/build.rs
mixer/build.rs
remix/tests/codec_tests.rs
desktop/src-tauri/build.rs
desktop/src-shared/build.rs

# Desktop Tauri configs
desktop/src-tauri/tauri.conf.json
desktop/src-tauri/tauri.macos.conf.json
desktop/src-tauri/tauri.ios.conf.json
desktop/src-tauri/tauri.windows.conf.json
desktop/src-tauri/Entitlements.plist
desktop/src-tauri/capabilities/desktop.json
desktop/src-tauri/capabilities/mobile.json
desktop/src-tauri/builtin-apps/group.json
desktop/src-tauri/vite.config.ts
desktop/src-tauri/tsconfig.json
desktop/.prettierrc

# rcm configs (Remix package manager)
desktop/rcm.config.mjs
desktop/rcm-lock.json
mixcore/rcm.config.mjs
mixer/rcm.config.mjs
mixrun/rcm.config.mjs
mixrun/rcm-lock.json

# Server deployment
mixer/server/Dockerfile
mixer/server/start-mixer.sh
mixer/server/configure-docker-creds
mixer/server-snowflake/Dockerfile
mixer/server-snowflake/mixer.yml
mixer/server-snowflake/start-mixer.sh
mixer/server-snowflake/rcm.config.mjs
mixer/server-snowflake/rcm-lock.json
mixer/server-snowflake/README.md
mixer/server-snowflake/prefetch-css-imports.sh
mixer/server-snowflake/app/builder.html
mixer/server-snowflake/app/runtime.html
mixer/resources/cargo-amendment.toml
mixer/resources/example-lib.rs
```

## Layer 5 — Supporting (low priority, skip if budget tight)

Desktop app modules, TypeScript frontend, bindings, tests, perf data.

```
# Desktop Tauri modules (src-tauri/src/)
desktop/src-tauri/src/app.rs
desktop/src-tauri/src/appclip.rs
desktop/src-tauri/src/apps.rs
desktop/src-tauri/src/auth.rs
desktop/src-tauri/src/builder.rs
desktop/src-tauri/src/claude.rs
desktop/src-tauri/src/cmd_error.rs
desktop/src-tauri/src/deep_link.rs
desktop/src-tauri/src/dialog.rs
desktop/src-tauri/src/env.rs
desktop/src-tauri/src/home.rs
desktop/src-tauri/src/http_utils.rs
desktop/src-tauri/src/instant.rs
desktop/src-tauri/src/logging.rs
desktop/src-tauri/src/menu.rs
desktop/src-tauri/src/mixcore.rs
desktop/src-tauri/src/mixer.rs
desktop/src-tauri/src/open_remix.rs
desktop/src-tauri/src/protocol.rs
desktop/src-tauri/src/rmx_auth.rs
desktop/src-tauri/src/rmx_signin.rs
desktop/src-tauri/src/routes.rs
desktop/src-tauri/src/runtime.rs
desktop/src-tauri/src/setup_handle.rs
desktop/src-tauri/src/state.rs
desktop/src-tauri/src/states.rs
desktop/src-tauri/src/tray.rs
desktop/src-tauri/src/updater.rs
desktop/src-tauri/src/user_info.rs
desktop/src-tauri/src/watch.rs
desktop/src-tauri/src/window.rs
desktop/src-tauri/src/workspace.rs

# Desktop shared library
desktop/src-shared/src/config.rs
desktop/src-shared/src/platform_version.rs
desktop/src-shared/src/release_info.rs
desktop/src-shared/src/sync.rs

# Desktop TypeScript frontend
desktop/src-tauri/src/builder/index.html
desktop/src-tauri/src/builder/main.ts
desktop/src-tauri/src/runtime/index.html
desktop/src-tauri/src/runtime/main.ts
desktop/src-tauri/src/login/index.html
desktop/src-tauri/src/login/main.ts
desktop/src-tauri/src/instant/index.html
desktop/src-tauri/src/instant/main.ts
desktop/src-tauri/src/error/index.html
desktop/src-tauri/src/error/main.ts
desktop/src-tauri/src/error/style.css
desktop/src-tauri/src/desktop.ts
desktop/src-tauri/src/event-handlers.ts

# JavaScript bindings (mixcore)
mixcore/bindings/javascript/src/mixcore.ts
mixcore/bindings/javascript/src/mixcore-base.ts
mixcore/bindings/javascript/src/mixcore-tauri.ts
mixcore/bindings/javascript/src/mixcore-wasm.ts
mixcore/bindings/javascript/src/mixcore-wasm-api.js
mixcore/bindings/javascript/src/mixcore-webkit.ts
mixcore/bindings/javascript/src/ffi-filter.ts
mixcore/bindings/javascript/package.json
mixcore/bindings/javascript/tsconfig.json
mixcore/bindings/javascript/webpack.config.js

# Swift / Java / Kotlin bindings
mixcore/bindings/swift/MixCore.swift
mixcore/bindings/java/com/remixlabs/runtime/MixCore.java
mixcore/bindings/java/com_remixlabs_runtime_MixCore.h
mixcore/bindings/java/Makefile
mixrun/bindings/swift/MixRun.swift
mixrun/bindings/kotlin/com/remixlabs/runtime/MixRun.kt

# C runtime support (mixrun)
mixrun/src/mixrun.c
mixrun/src/wasi.c
mixrun/groovebox-assets/wasm2c/include/mixrt.h
mixrun/groovebox-assets/wasm2c/include/wasm-rt.h
mixrun/groovebox-assets/wasm2c/include/wasm-rt-impl.h
mixrun/groovebox-assets/wasm2c/src/mixrt.c
mixrun/groovebox-assets/wasm2c/src/wasm-rt-impl.c
mixrun/groovebox-assets/machine-wasm/include/control.h
mixrun/groovebox-assets/machine-wasm/include/errors.h
mixrun/groovebox-assets/machine-wasm/include/logic.h
mixrun/groovebox-assets/machine-wasm/include/value.h
mixrun/groovebox-assets/Makefile
mixrun/groovebox-assets/README.md
mixrun/examples/swift_test.c

# Tests
mixcore/src/tests/ffis.rs
mixcore/src/tests/proxies.rs
mixer/tests/test_auth_configs.rs
mixer/tests/test_cloud_query.rs
mixer/tests/test_files.rs
mixer/tests/test_serve.rs
mixer/tests/test_util.rs
mixquery/tests/query.rs
mixer/src/serve/model/test.rs
mixrun/bin/which-ref-type.mjs

# Perf evaluation
mixcore/eval-wasmer-wasm2c/perf_data.html
mixcore/eval-wasmer-wasm2c/perf_data.ipynb
mixcore/eval-wasmer-wasm2c/perf_data/*.csv
```

## Excluded (do not read)

```
# Generated Xcode / iOS project files
desktop/src-tauri/gen/apple/**

# Test database fixtures (binary data)
mixquery/tests/resources/db/**

# Lock files (auto-generated)
Cargo.lock
desktop/package-lock.json
mixcore/bindings/javascript/package-lock.json

# Icon assets
desktop/src-tauri/icons/*

# Snowflake server vendored dep placeholders
mixer/server-snowflake/deps/**

# Mobile provisioning profiles
desktop/src-tauri/gen/Remix_Mobile_App_Store.mobileprovision
```