# amp

**Repository:** https://github.com/remixlabs/amp
**Stack:** Go 1.24 (primary), CGO (wasmtime/mixrun native), gomobile (iOS/Android), Node.js (migration scripts)
**Purpose:** The Remix platform server — HTTP API, embedded Mix VM (Groovebox), MQTT broker, compiler bridge, auth system, and mobile library
**Explored layers:** L1–L3 (skeleton, API surface, core modules — ~50 files, ~8000 lines)

## Build & Deploy

- **Build:** `make build` → `bin/amp` (CGO-linked with `mixrun` / `wasmtime` native libs via `rcm install-prefix`)
- **Mobile:** `make ios` → `Ampmobile.xcframework`; `make android` → `Ampmobile.aar` (via gomobile)
- **Docker:** Debian trixie-slim, installs amp + mixc_server + mix_migrate + Node.js migration; exposes ports 8000 (HTTP) + 8002 (MQTT broker)
- **CI:** CircleCI — builds linux/amd64, linux/arm64, darwin/arm64, Android; 5 test suites (ci-0..ci-6); Docker image pushed to `gcr.io/rmx-prod/amp`; `rcm` tool manages deps/artifacts
- **Key env vars:** `RMX_ORG`, `RMX_WORKSPACE`, `RMX_BASE_URL`, `RMX_ROOT_USER`, `RMX_SESSION_TIMEOUT`, `RMX_CYCLE_TIMEOUT`, `GCP_PROJECT/REGION/KEYRING`
- **CLI flags:** `--port` (8000), `--admin-port` (8001), `--broker-port` (8002), `--solo` (desktop mode), `--no-compiler`, `--database-dir`, `--read-only`, `--dangerous-http-mode`

## Architecture

```
main.go → lib.Server → { server.Server (HTTP), track.SessionManager, broker (MQTT), compiler_messaging }
                             ↓                          ↓
                        gorilla/mux              session → machine.VM (Groovebox/Wasm)
                        REST + WS API                     ↕ FFI bridge
                             ↓                    protoquery.Compiler (OCaml via MQTT)
                        apps.AppManager
                        (eleventwo DB engine)
```

### Core packages

| Package              | Role                                                                    |
|----------------------|-------------------------------------------------------------------------|
| `lib`                | Global config, server lifecycle, init                                   |
| `server`             | HTTP routes, middleware, REST handlers, WS track handler                |
| `apps`               | App CRUD, access control (read/run/build/create), DB engine wrapper     |
| `track`              | Session manager, request/response protocol, FFI dispatch, compilation   |
| `machine`            | VM interface, ExecSet, Program, Factory abstractions                    |
| `machine/wasm`       | Groovebox Wasm engine (wasmtime or mixrun backend)                      |
| `machine/reader`     | QCode v5 / Wasm bytecode reader, header parsing, ExecSet implementation |
| `auth`               | JWT verification, token exchange, auth middleware, plugin dispatch      |
| `auth/plugins`       | OIDC/OAuth/Apple/Snowflake/WebAuthn plugin registry                     |
| `broker`             | Embedded HMQ MQTT broker                                                |
| `brokerclient`       | MQTT client (pub/sub, filtering, multiplexing)                          |
| `compiler/messaging` | Bridge to protoquery OCaml compiler over MQTT                           |
| `protoquery`         | Compiler interface + stub for mobile                                    |
| `secrets`            | Envelope encryption, GCP KMS, local key storage                         |

## API Surface / Entry Points

### System endpoints (`/x/...`)



| Endpoint                     | Method   | Purpose                                  |
|------------------------------|----------|------------------------------------------|
| `/x/auth`                    | GET      | Redirect to default auth provider        |
| `/x/auth/{handler}`          | GET      | OAuth/OIDC login flow                    |
| `/x/auth/{handler}/callback` | GET/POST | OAuth callback                           |
| `/x/auth/{handler}/token`    | GET/POST | Exchange for 3rd-party token             |
| `/x/token`                   | POST     | Exchange/renew amp JWT token             |
| `/x/token/revoke`            | POST     | Revoke amp token                         |
| `/x/token/acceptTerms`       | POST     | Record T&C acceptance                    |
| `/x/authConfig`              | GET/POST | Admin: list/create auth plugins          |
| `/x/authConfig/public`       | GET/POST | Public auth plugin CRUD                  |
| `/x/pubkey/challenge`        | POST     | WebAuthn challenge                       |
| `/x/pubkey/authenticate`     | POST     | WebAuthn authentication                  |
| `/x/apps`                    | GET/POST | List/manage apps (import/export/install) |
| `/x/profile`                 | GET      | Current user info                        |
| `/x/serverinfo`              | GET      | Server version + capabilities            |
| `/x/health/*`                | GET      | Disk/memory/CPU health checks            |
| `/x/debug/pprof/*`           | GET      | Go profiling endpoints                   |
| `/x/broker.ws`               | WS       | WebSocket proxy to MQTT broker           |
| `/x/rcm/`                    | GET      | Static builder/runtime files             |

### Per-app endpoints (`/{app}/...`)

| Endpoint                       | Method                | Purpose                                 |
|--------------------------------|-----------------------|-----------------------------------------|
| `/{app}`                       | POST/PUT/GET/DELETE   | App CRUD                                |
| `/{app}/documents`             | POST/PATCH/GET/DELETE | Document CRUD (batch + single)          |
| `/{app}/save`                  | POST/PATCH            | Save with projection                    |
| `/{app}/query`                 | GET/POST              | Query API (string or AST, ETag caching) |
| `/{app}/track`                 | POST                  | Synchronous track session request       |
| `/{app}/track.ws`              | WS                    | WebSocket bidirectional session         |
| `/{app}/webhooks/{id}`         | GET/POST              | Inbound webhook → Mix agent             |
| `/{app}/agents/{module}`       | GET/POST              | Agent endpoint → Mix module             |
| `/{app}/lambda/{module}`       | GET/POST              | Lambda (lite compile) → Mix module      |
| `/{app}/files`                 | GET/POST/PUT/DELETE   | File CRUD                               |
| `/{app}/resources`             | GET/POST/DELETE       | Resource CRUD                           |
| `/{app}/records`               | GET/POST/DELETE       | Low-level record access                 |
| `/{app}/proxy/{remote}/{path}` | *                     | Proxy to external server                |
| `/{app}/link`                  | POST                  | Create shareable magic link             |
| `/{app}/runtime.json`          | GET                   | Runtime metadata (public)               |

## Core Modules

### Track — Session Management (`track/`)

- **SessionManager:** Singleton managing all VM sessions + protoquery compiler + code caches
- **Session types:** Normal (interactive), Temp (REPL/compile), Agent (persistent stateful)
- **~30 message types** organized as request/response JSON protocol:
    - **Session lifecycle:** `msg_session_create`, `msg_session_destroy`, `msg_session_poll`, `msg_session_keepAlive`
    - **View operations:** `msg_view_load`, `msg_view_push`, `msg_view_pop`, `msg_view_input`, `msg_view_invoke`, `msg_view_reset`, `msg_view_reload`, `msg_view_refresh`
    - **Compilation:** `msg_compile_create`, `msg_compile_build`, `msg_compile_modules`, `msg_compile_runPhrases`, `msg_compile_store`
    - **Introspection:** `msg_introspect_spreadsheet`, `msg_introspect_debug`
    - **VM config:** `msg_vm_configure`, `msg_machine_create`
- **Agent/webhook/lambda pattern (MixTransform):** create session → load module with input → read output cell → destroy (stateless) or keep running (stateful/persistent). Supports async, idle actions,
  field projection, timeout.
- **Code caching:** Immutable ExecSet cached per (hash, app, codeID) with mutable copies for sessions
- **FFI bridge:** Session-bound FFIs (DB, HTTP, compiler, auth, secrets, crypto, XML, extract, token, resource, logging) dispatched by name string

### Machine — VM Runtime (`machine/`, `machine/wasm/`)

- **machine.VM interface:** GetState, Run, RunSince, Upgrade, Terminate, GetStackTrace, ByteSize
- **machine.ExecSet:** Immutable program container with QCode + Wasm variants, library dependencies, locator
- **Groovebox Engine (`wasm/Engine`):** Embeds mixrt Wasm runtime with two backends:
    - **Wasmtime** (`backend_wasmtime.go`): Full Wasm JIT via `wasmtime-go` — used for Wasm variant
    - **Mixrun** (`backend_mixrun.go`): Native C interpreter via CGO (wasm2c) — used for QCode5 variant
- **State machine:** Init → InitDone → Running → {Idle, FFIcall, Panic, Exit, Timeout} — Run loop dispatches FFI calls back to Go
- **Data exchange:** `rmxcodec` binary format for Go↔Wasm value serialization via IO buffer

### Auth (`auth/`)

- **JWT tokens:** Ed25519/ECDSA signing, local + prod (central auth server) keys, revocation store
- **Plugin system:** OIDC, OAuth2, Apple Sign In, Snowflake, WebAuthn — stored in `_rmx_admin` DB, loaded on startup
- **Public vs confidential plugins:** Public plugins can be created by any user (owns their plugin); confidential plugins require admin access
- **Auth middleware:** Extracts Bearer token from Authorization header or `?token=` query param; creates anonymous token if missing
- **Central auth:** `auth.remixlabs.com` is the prod signing authority; other servers verify via JWKS endpoint

### Apps — Data Layer (`apps/`)

- **AppManager:** Manages disk + in-memory DB engines (via `eleventwo` library)
- **Special apps:** `_rmx_admin` (access control rules as documents), `_rmx_meta` (metadata)
- **Access roles:** `allowed_read`, `allowed_run`, `allowed_builders`, `allowed_agents`, `allowed_create` — stored per-user in admin DB
- **Users:** root user, anonymous users (`anon-*`), `_rmx_all_users`, `_rmx_anon_user`; wildcard `*` grants access to all non-admin apps
- **App actions:** Copy, Export, Import, Package, Fix, GarbageCollect, Reindex, Rename, Reset, RemoteExport/Import — all go through `Execute()` with ACL check

### Mobile (`mobile/`)

- **gomobile binding:** `ampmobile` package with Init/Start/Stop/Terminate/DeleteEverything/StoreToken
- **Config:** Stub compiler, unbuffered engine, no admin port, 24h session timeout, dangerous HTTP mode, central auth JWKS endpoint
- **Lifecycle:** Flutter app calls Start(user, dbDir, tmpDir, versionJSON) → embedded amp server on localhost

## Key Patterns & Conventions

- **Encoding:** eleventwo custom DB format with MessagePack/JSON/rmxcodec serialization; `node.FromJSON`, `node.FromMsgPack`, `node.FromPureData` converters
- **Logging:** zerolog with structured JSON (severity field for GCP compatibility)
- **Config:** Global vars in `lib/` package set by CLI flags + env vars, passed to subsystems via `ContextConfig`/`SessionConfig`
- **Testing:** Tests use `testserver` package for integration tests; testdata has binary DB fixtures
- **Solo mode:** Desktop/Electron mode serves builder (`/e/`), runtime (`/r/`), groovebox (`/g/`) static files, rewrites routes under `/a/`
- **Dependencies:** `eleventwo` (DB engine + data model), `elaws` (AWS plugin), `siwago` (Apple Sign In), forked `hmq` (MQTT broker), forked `gosnowflake`

## Open Questions

- **Detailed FFI implementations:** Each FFI module (db, http, compiler, crypto, xml, etc.) was not deeply explored
- **Compiler messaging protocol:** Exact MQTT topic/message format between amp and protoquery
- **Incremental compilation:** `compiler_incremental.go` logic for partial rebuilds
- **Broker authorization:** How MQTT topic ACLs map to app access rules
- **DynLoad:** Dynamic library loading at runtime (`dynload.go`) details
- **Remote machines:** `msg_machine_create` and remote FFI architecture (security issue noted in code: #2051)
