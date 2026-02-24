# amp — Exploration Plan

**Repository:** https://github.com/remixlabs/amp
**Branch:** main
**Stack:** Go (primary), Rust (Wasm examples), Node.js (migration scripts), Shell (deploy/bin scripts)
**Structure style:** Go monorepo — platform server (AMP) with embedded VM, MQTT broker, compiler bridge, HTTP API, auth system, plus CLI utilities and Wasm examples
**Total files:** 995 (but ~700+ are testdata binary DB fixtures; ~150 meaningful source files)

## Layer 1 — Skeleton & Overview (must read)

- `README.md`
- `Makefile`
- `go.mod` (root-level, if exists — but actual Go module is in `src/amp/`)
- `src/amp/go.mod`
- `src/amp/main.go`
- `src/amp/doc.json`
- `.circleci/config.yml`
- `deploy/amp/Dockerfile`
- `deploy/amp/startup.sh`
- `deploy/build-and-push-images`
- `rcm.config.mjs`
- `.gitmodules`
- `docs/architecture.md`
- `docs/deployment.md`

## Layer 2 — Entry Points & API Surface (high priority)

### HTTP Server (routes, handlers, API surface)
- `src/amp/server/server.go`
- `src/amp/server/apps.go`
- `src/amp/server/auth.go`
- `src/amp/server/rest.go`
- `src/amp/server/rest_handler.go`
- `src/amp/server/query.go`
- `src/amp/server/batch.go`
- `src/amp/server/health.go`
- `src/amp/server/webhook.go`
- `src/amp/server/track.go`
- `src/amp/server/trackMessaging.go`
- `src/amp/server/invite.go`
- `src/amp/server/profile.go`
- `src/amp/server/proxy.go`
- `src/amp/server/pubkeys.go`
- `src/amp/server/recompile.go`
- `src/amp/server/static_handler.go`
- `src/amp/server/public_rcm_handler.go`
- `src/amp/server/errors.go`

### App Management (core domain)
- `src/amp/apps/apps.go`
- `src/amp/apps/actions.go`
- `src/amp/apps/actions_impl.go`
- `src/amp/apps/actions_remote.go`
- `src/amp/apps/access.go`
- `src/amp/apps/files.go`
- `src/amp/apps/libraries.go`
- `src/amp/apps/deployments.go`
- `src/amp/apps/resources.go`
- `src/amp/apps/remix_package.go`
- `src/amp/apps/remix_package_v1.go`
- `src/amp/apps/remix_package_v2.go`
- `src/amp/apps/runtime_metadata.go`
- `src/amp/apps/secrets.go`
- `src/amp/apps/users.go`
- `src/amp/apps/invite.go`
- `src/amp/apps/link.go`
- `src/amp/apps/vm_pref.go`
- `src/amp/apps/util.go`
- `src/amp/apps/error_context.go`

### Lib entry
- `src/amp/lib/lib.go`
- `src/amp/mobile/mobile.go`

## Layer 3 — Core Modules (medium priority)

### Track / Session Runtime (VM session management, messaging, compilation)
- `src/amp/track/runtime.go`
- `src/amp/track/manager.go`
- `src/amp/track/sessions.go`
- `src/amp/track/messaging.go`
- `src/amp/track/request.go`
- `src/amp/track/requestCompile.go`
- `src/amp/track/response.go`
- `src/amp/track/compiler.go`
- `src/amp/track/compiler_incremental.go`
- `src/amp/track/compilerError.go`
- `src/amp/track/db.go`
- `src/amp/track/agent.go`
- `src/amp/track/broker.go`
- `src/amp/track/cloud.go`
- `src/amp/track/code_cache.go`
- `src/amp/track/dynload.go`
- `src/amp/track/env.go`
- `src/amp/track/defaults.go`
- `src/amp/track/defaults_mobile.go`
- `src/amp/track/recorder.go`
- `src/amp/track/secrets.go`
- `src/amp/track/submachine.go`
- `src/amp/track/README.md`

### Machine / VM (bytecode execution, Wasm backend)
- `src/amp/machine/machine.go`
- `src/amp/machine/linker.go`
- `src/amp/machine/header.go`
- `src/amp/machine/state.go`
- `src/amp/machine/error.go`
- `src/amp/machine/id.go`
- `src/amp/machine/factory/factory.go`
- `src/amp/machine/ffi/ffi.go`
- `src/amp/machine/mnode/codec.go`
- `src/amp/machine/mnode/constant.go`
- `src/amp/machine/mnode/frame.go`
- `src/amp/machine/mnode/mutable.go`
- `src/amp/machine/reader/reader.go`
- `src/amp/machine/reader/qcode_v5.go`
- `src/amp/machine/reader/instruction.go`
- `src/amp/machine/reader/codeptr.go`
- `src/amp/machine/reader/location.go`
- `src/amp/machine/reader/program_groovebox.go`
- `src/amp/machine/reader/execset_groovebox.go`
- `src/amp/machine/session/session.go`
- `src/amp/machine/session/encode.go`
- `src/amp/machine/writer/qcode_v5.go`
- `src/amp/machine/wasm/wasm.go`
- `src/amp/machine/wasm/backend.go`
- `src/amp/machine/wasm/backend_mixrun.go`
- `src/amp/machine/wasm/backend_wasmtime.go`
- `src/amp/machine/wasm/backend_wasmtime_disabled.go`
- `src/amp/machine/wasm/logging.go`

### Auth (authentication, OAuth, OIDC, tokens)
- `src/amp/auth/auth.go`
- `src/amp/auth/handlers.go`
- `src/amp/auth/handlers/handlers.go`
- `src/amp/auth/apple.go`
- `src/amp/auth/cookie.go`
- `src/amp/auth/global.go`
- `src/amp/auth/oauth.go`
- `src/amp/auth/oidc.go`
- `src/amp/auth/oidc_password.go`
- `src/amp/auth/sign.go`
- `src/amp/auth/verify.go`
- `src/amp/auth/revoke.go`
- `src/amp/auth/pubkey.go`
- `src/amp/auth/userid.go`
- `src/amp/auth/same_site_check.go`
- `src/amp/auth/snowflake.go`
- `src/amp/auth/snowflake_stub.go`
- `src/amp/auth/token/token.go`
- `src/amp/auth/tokens/tokens.go`
- `src/amp/auth/prod/prod.go`
- `src/amp/auth/keys/file_cache.go`
- `src/amp/auth/plugins/plugin.go`
- `src/amp/auth/plugins/impl.go`
- `src/amp/auth/plugins/registry.go`
- `src/amp/auth/plugins/central.go`
- `src/amp/auth/users/user.go`
- `src/amp/auth/users/invite.go`
- `src/amp/auth/users/subscription.go`

## Layer 4 — Data, Config & FFI (medium priority)

### Track FFI (foreign function interfaces)
- `src/amp/track/ffi/auth/auth.go`
- `src/amp/track/ffi/compiler/create_exe.go`
- `src/amp/track/ffi/crypto.go`
- `src/amp/track/ffi/db/db.go`
- `src/amp/track/ffi/db/ffi.go`
- `src/amp/track/ffi/db/ffi_buildServer.go`
- `src/amp/track/ffi/db/salesforce.go`
- `src/amp/track/ffi/extract/extract.go`
- `src/amp/track/ffi/http/http.go`
- `src/amp/track/ffi/http/websockets.go`
- `src/amp/track/ffi/info.go`
- `src/amp/track/ffi/resource/resource.go`
- `src/amp/track/ffi/secrets/secrets.go`
- `src/amp/track/ffi/token/token.go`
- `src/amp/track/ffi/xml/xml.go`
- `src/amp/track/ffi/ffi_mobile.go`
- `src/amp/track/wasm_ffis_interp.go`
- `src/amp/track/wasm_ffis_jit.go`
- `src/amp/track/web.json`

### Broker (MQTT)
- `src/amp/broker/broker.go`
- `src/amp/broker/config/config.go`
- `src/amp/brokerclient/mqtt.go`
- `src/amp/brokerclient/filter.go`
- `src/amp/brokerclient/multiplex.go`
- `src/amp/brokerclient/types.go`

### Compiler Messaging (protoquery bridge)
- `src/amp/compiler/messaging/server.go`
- `src/amp/compiler/messaging/context.go`
- `src/amp/compiler/messaging/execute.go`
- `src/amp/compiler/messaging/request.go`
- `src/amp/compiler/messaging/response.go`
- `src/amp/protoquery/api.go`
- `src/amp/protoquery/compiler/compiler.go`
- `src/amp/protoquery/stub/compiler.go`

### Secrets
- `src/amp/secrets/secrets.go`
- `src/amp/secrets/encrypt.go`
- `src/amp/secrets/envelope.go`
- `src/amp/secrets/machine_id.go`
- `src/amp/secrets/platform_keys.go`
- `src/amp/secrets/memstore.go`
- `src/amp/secrets/gcp/gcp.go`
- `src/amp/secrets/gcp/README.md`

### Random
- `src/amp/random/random.go`

### Docs
- `docs/access_control.md`
- `docs/broker.md`
- `docs/builder_devs.md`
- `docs/backup_restore.md`
- `docs/delete_org.md`
- `docs/plugin_auth.md`
- `docs/self_host.md`

## Layer 5 — Supporting (low priority, skip if budget tight)

### CLI Utilities
- `src/ampq/main.go` (query tool)
- `src/amputil/main.go` (utility)
- `src/amputil/tail/main.go`
- `src/json2mp/json2mp.go` (JSON→MessagePack)
- `src/mp2json/mp2json.go` (MessagePack→JSON)
- `src/wl2json/main.go`
- `src/make-apple-secret/main.go`
- `src/mktoken/mktoken.go`
- `src/amp.js/main.go` + `src/amp.js/README.md`

### Wasm Examples (Rust)
- `src/wasm/rust.makefile`
- `src/wasm/example/Cargo.toml`
- `src/wasm/example/src/lib.rs`
- `src/wasm/jsontest/Cargo.toml`
- `src/wasm/jsontest/src/lib.rs`

### Shell Scripts
- `bin/backward_compat_check.sh`
- `bin/builder-migrate`
- `bin/builder-rebuild`
- `bin/copy-mobile.sh`
- `bin/copy-user-data`
- `bin/download-db`
- `bin/encode-app-name`
- `bin/get-profile`
- `bin/reset-all-apps`
- `bin/submodule_master_commit_check.sh`
- `deploy/amp/build-and-push-image`
- `deploy/amp/migrate/mix_migrate_export`
- `deploy/amp/migrate/mix_migrate_import`
- `deploy/configure-docker-creds`

### Migration (Node.js)
- `migration/package.json`
- `migration/migrate_amp.js`
- `migration/rebuilder.js`
- `migration/README.md`

### Tests (read selectively to understand patterns)
- `src/amp/server/server_test.go`
- `src/amp/server/apps_test.go`
- `src/amp/server/rest_test.go`
- `src/amp/server/query_test.go`
- `src/amp/server/auth_test.go`
- `src/amp/server/batch_test.go`
- `src/amp/server/track_test.go`
- `src/amp/server/webhook_test.go`
- `src/amp/server/pubkeys_test.go`
- `src/amp/apps/apps_test.go`
- `src/amp/apps/access_test.go`
- `src/amp/apps/actions_test.go`
- `src/amp/apps/actions_remote_test.go`
- `src/amp/apps/files_test.go`
- `src/amp/apps/resources_test.go`
- `src/amp/apps/remix_package_v1_test.go`
- `src/amp/apps/remix_package_v2_test.go`
- `src/amp/apps/runtime_metadata_test.go`
- `src/amp/auth/auth_test.go`
- `src/amp/auth/sign_test.go`
- `src/amp/auth/revoke_test.go`
- `src/amp/auth/pubkey_test.go`
- `src/amp/auth/prod/prod_test.go`
- `src/amp/auth/keys/file_cache_test.go`
- `src/amp/auth/token/token_test.go`
- `src/amp/auth/plugins/registry_test.go`
- `src/amp/auth/users/user_test.go`
- `src/amp/track/db_test.go`
- `src/amp/track/compiler_incremental_test.go`
- `src/amp/track/dynload_test.go`
- `src/amp/track/sessions_test.go`
- `src/amp/track/messaging_test.go`
- `src/amp/track/integration_test.go`
- `src/amp/track/live_query_test.go`
- `src/amp/track/remote_test.go`
- `src/amp/track/request_test.go`
- `src/amp/track/utilCommon_test.go`
- `src/amp/track/utilMobile_test.go`
- `src/amp/track/utilNormal_test.go`
- `src/amp/track/ffi/extract/extract_test.go`
- `src/amp/track/ffi/http/http_test.go`
- `src/amp/track/ffi/crypto_test.go`
- `src/amp/machine/reader/location_test.go`
- `src/amp/machine/session/session_test.go`
- `src/amp/machine/writer/qcode_v5_test.go`
- `src/amp/brokerclient/filter_test.go`
- `src/amp/brokerclient/mqtt_test.go`
- `src/amp/compiler/messaging/` (no tests seen)
- `src/amp/protoquery/compiler/compiler_test.go`
- `src/amp/secrets/envelope_test.go`
- `src/amp/secrets/machine_id_test.go`
- `src/amp/secrets/secrets_test.go`
- `src/amp/secrets/gcp/gcp_test.go`
- `src/amp/testserver/testserver.go`

## Excluded (do not read)

- `src/amp/track/testdata/testdbs/**` — binary database fixtures (~600+ files of .dat/.off/.idx/.json index data)
- `src/amp/track/testdata/LMS/*.mix` — sample Mix app files (read 1-2 only if curious)
- `src/amp/track/testdata/jd_ta/` — test store data
- `src/amp/track/testdata/wasm/` — precompiled .wasm test binaries
- `src/amp/machine/testdata/` — precompiled .qcodev5 and .json test fixtures
- `src/amp/machine/resources/` — just a README
- `src/amp/track/resources/` — just a README
- `go.sum` files everywhere — lock files
- `rcm-lock.json` — lock file
- `bindeps/go.mod`, `bindeps/go.sum` — build dependency lock files
- `Cargo.lock` files in `src/wasm/` — lock files
- `*.wasm` — compiled Wasm binaries
- `.github/ISSUE_TEMPLATE/` — issue template boilerplate

## Reading Budget Estimate

| Layer | Files | Est. Lines |
|-------|-------|-----------|
| L1    | 14    | ~800      |
| L2    | 40    | ~4000     |
| L3    | 55    | ~6000     |
| L4    | 32    | ~3000     |
| L5    | 60+   | ~4000     |
| **Total L1–L3** | **~109** | **~10800** |

Recommended budget: L1–L3 (~109 files, ~10800 lines) for a solid understanding of the AMP server architecture, API surface, VM runtime, and auth system. L4 adds FFI/broker/compiler details. L5 is CLI tools, scripts, and tests.
