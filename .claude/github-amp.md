# amp

**Repository:** https://github.com/remixlabs/amp
**Stack:** Go 1.24 (primary), CGO (wasmtime/mixrun native), gomobile (iOS/Android), Node.js (migrations)
**Purpose:** Remix platform server — HTTP API, embedded Mix VM (Groovebox), MQTT broker, compiler bridge, auth, mobile library
**Explored:** L1–L3 (~50 files, ~8000 lines)

## Build & Deploy

- `make build` → `bin/amp` (CGO-linked with mixrun/wasmtime via `rcm install-prefix`)
- `make ios` → `Ampmobile.xcframework`; `make android` → `Ampmobile.aar` (gomobile)
- Docker: Debian trixie-slim; ports 8000 (HTTP) + 8002 (MQTT)
- CI: CircleCI — linux/amd64+arm64, darwin/arm64, Android; 5 test suites; Docker → `gcr.io/rmx-prod/amp`
- Key env: `RMX_ORG`, `RMX_WORKSPACE`, `RMX_BASE_URL`, `RMX_ROOT_USER`, `RMX_SESSION_TIMEOUT`
- CLI: `--port` (8000), `--admin-port` (8001), `--broker-port` (8002), `--solo` (desktop), `--no-compiler`, `--database-dir`

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

### Core Packages

| Package                   | Role                                                                                                  |
|---------------------------|-------------------------------------------------------------------------------------------------------|
| `lib`                     | Global config, server lifecycle                                                                       |
| `server`                  | HTTP routes, middleware, REST handlers, WS track                                                      |
| `apps`                    | App CRUD, ACL (read/run/build/create), eleventwo DB wrapper                                           |
| `track`                   | Session manager, request/response protocol, FFI dispatch, compilation                                 |
| `machine`                 | VM interface; `machine/wasm` = Groovebox (wasmtime JIT or mixrun native)                              |
| `machine/reader`          | QCode v5 / Wasm bytecode reader, ExecSet                                                              |
| `auth`                    | JWT (Ed25519/ECDSA), token exchange, middleware; `auth/plugins` = OIDC/OAuth/Apple/Snowflake/WebAuthn |
| `broker` / `brokerclient` | Embedded HMQ MQTT broker + client                                                                     |
| `compiler/messaging`      | Bridge to protoquery compiler over MQTT                                                               |
| `secrets`                 | Envelope encryption, GCP KMS, local key storage                                                       |
| `mobile`                  | gomobile binding: Init/Start/Stop/Terminate; stub compiler, central auth JWKS                         |

## API Surface

### System (`/x/...`)
- **Auth:** `/x/auth/{handler}` (login), `/x/auth/{handler}/callback` (OAuth callback), `/x/token` (exchange/renew), `/x/token/revoke`
- **WebAuthn:** `/x/pubkey/challenge`, `/x/pubkey/authenticate`
- **Admin:** `/x/authConfig` (auth plugin CRUD), `/x/apps` (list/import/export), `/x/profile`, `/x/serverinfo`
- **Infra:** `/x/health/*`, `/x/debug/pprof/*`, `/x/broker.ws` (WS→MQTT proxy), `/x/rcm/` (static files)

### Per-app (`/{app}/...`)
- **Data:** `/{app}` (CRUD), `/documents` (batch CRUD), `/save` (with projection), `/query` (string/AST, ETag)
- **Execution:** `/track` (sync session), `/track.ws` (WS bidirectional), `/webhooks/{id}`, `/agents/{module}`, `/lambda/{module}`
- **Files:** `/files`, `/resources`, `/records`, `/proxy/{remote}/{path}`, `/link`, `/runtime.json`

## Core Modules — Key Details

**Track (session management):** ~30 message types (session lifecycle, view ops, compilation, introspection, VM config). Agent/webhook/lambda pattern (MixTransform): create session → load module with input → read output → destroy or keep. Code caching per (hash, app, codeID). FFI bridge: DB, HTTP, compiler, auth, secrets, crypto, XML, extract, token, resource, logging.

**Machine (VM runtime):** Groovebox engine with wasmtime (Wasm JIT) and mixrun (native C interpreter via CGO) backends. State machine: Init→Running→{Idle, FFIcall, Panic, Exit, Timeout}. Data exchange via `rmxcodec` binary format.

**Auth:** JWT local + prod (central `auth.remixlabs.com` via JWKS). Public plugins (user-owned) vs confidential (admin). Bearer token from header or `?token=` param; anonymous if missing.

**Apps:** `_rmx_admin` (ACL rules), `_rmx_meta` (metadata). Roles: `allowed_read/run/build/create/agents`. Special users: root, anonymous (`anon-*`), `_rmx_all_users`.

## Key Patterns

- eleventwo custom DB with MessagePack/JSON/rmxcodec serialization
- zerolog structured JSON (GCP severity field)
- Solo mode: desktop serves builder (`/e/`), runtime (`/r/`), groovebox (`/g/`) static files
- Dependencies: `eleventwo` (DB), `elaws` (AWS), `siwago` (Apple Sign In), forked `hmq`/`gosnowflake`

## Recent PRs

| PR    | Date   | What                                                                     | Who   |
|-------|--------|--------------------------------------------------------------------------|-------|
| #2790 | Feb 27 | Register `$binary_compress`/`$binary_decompress` in unsupported FFI list | Gerd  |
| #2788 | Feb 15 | Drop rebuild tests in CI (moved to protoquery)                           | Chris |
| #2787 | Feb 11 | Add blob FFIs as unsupported                                             | Gerd  |
| #2786 | Jan 27 | Save/upsert return result type                                           | Fred  |
| #2785 | Jan 27 | eleventwo 1.5.6_pre fixes                                                | Fred  |
| #2784 | Jan 28 | Manifest version 2.1 support                                             | Gerd  |
