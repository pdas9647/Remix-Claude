# Remix Platform Architecture

> Notion: [The Remix Platform](https://www.notion.so/e29236e1181f4a8e8e25cfd607b021f1)
> Subpages: Components of Remix, Implementations of the Remix runtime stack, mixcore, mixrt, rcm, amp, Cloud Agent Server (mixer), Components of the "prod environment"

---

## Core Components & Repos

| Component           | Repo                        | Language   | Description                                                                                                                                 |
|---------------------|-----------------------------|------------|---------------------------------------------------------------------------------------------------------------------------------------------|
| **rcm**             | `remixlabs/rcm`             | Bash/TS    | Package manager for Remix components (like npm for internal deps)                                                                           |
| **protoquery**      | `remixlabs/protoquery`      | —          | Compiler for the **Mix** programming language. Produces `mixc`, `mixc_server`, `mix_client`                                                 |
| **eleventwo**       | `remixlabs/eleventwo`       | Go         | Core database — history-preserving, append-only, unstructured, fully indexed. Used by amp server and mobile app                             |
| **amp**             | `remixlabs/amp`             | Go         | Combined backend server: HTTP API to DBs + WebSocket/MQTT for builder/runtime sessions. The "studio server" / "platform server"             |
| **turntable**       | `remixlabs/turntable`       | JS/Elm     | Frontend assets for builder and runtime + web components (`rmx-runtime`, `rmx-editor`, etc.). Also home of Chrome extension and desktop app |
| **remix.app**       | `remixlabs/remix.app`       | JS         | Web app clip runtime + CloudFlare worker for .remix file server                                                                             |
| **groovebox**       | `remixlabs/groovebox`       | C          | Mix VM — executes Mix bytecode (QCode or Wasm). Compiled to `mixrt.wasm` and native `mixrt.{h,c}`                                           |
| **mix-rs**          | `remixlabs/mix-rs`          | Rust       | Collection of Rust crates: `remix`, `mixquery`, `mixcore`, `mixrun`, `mixer`                                                                |
| **flutter-runtime** | `remixlabs/flutter-runtime` | Dart/Swift | iOS and Android mobile apps                                                                                                                 |
| **M**               | `remixlabs/M`               | —          | Cloud infrastructure for deploying mixer                                                                                                    |
| **harmony**         | `remixlabs/harmony`         | —          | Self-contained Docker image ("monoimage") combining amp + mixc + turntable + nginx                                                          |

## Key Rust Crates (mix-rs)

- **mixquery**: Database engine (Rust). Operates on same file hierarchy as eleventwo (Go DB). Nearly feature-equivalent.
- **mixcore**: FFI calls/dispatch + action handling + builder endpoints (`/a` and `/mixcore/a`). Builds to `mixcore.wasm` (browser) and `mixcore.a` (native)
- **mixrun**: Combines mixcore + mixrt.wasm (compiled back to C via wasm2c). Produces `libmixrun.a` for iOS/Android/Linux/WASI
- **mixer**: Server packaging of mixrun — the cloud agent server binary. Also CLI utilities.

## Runtime Stack Layers

1. **Bytecode**: Compiled Mix code in Wasm or QCode format
2. **VM**: `mixrt.wasm` (browser) or `mixrt.c` (native, via wasm2c)
3. **Bindings**: `wasmtime` (Go, for amp) or `mixrun` (Rust/C)
4. **Driver/Run Loop**: Event-driven state machine managing FFI dispatch + user input
    - Go driver: `track` (in amp, mobile app)
    - Rust driver: `mixrun` (in mixer, iOS widgets, agents)
    - JS driver: `groovebox/machine-wasm` (in web components, app clips, desktop)
5. **Session Manager**: Wires driver to client sessions
6. **Renderer/Client**: DOM rendering + user interaction (Elm frontend, Flutter, SwiftUI, or headless for agents)
7. **Database**: `eleventwo` (Go, in amp) or `mixquery` (Rust, in mixer/app clips/widgets)

## Two Database Engines

- **eleventwo** (Go): Used by amp server, mobile app, desktop app. Embedded in the amp binary.
- **mixquery** (Rust): Used by mixer (agent server), app clips, widgets, web components. Nearly feature-equivalent to eleventwo. Can operate on the same file hierarchy in parallel (but avoid
  concurrent access).

## Deployment Architecture

### Production Servers

- **Studio/Platform Server (amp)**: e.g., `nevista.remixlabs.com/e`, `remix.remixlabs.com/e`
    - 3 parts: static assets (GCS) + amp pod (GKE) + nginx pod (GKE)
    - Auth server: `auth.remixlabs.com`
    - Dev: `remix-dev.remixlabs.com`
- **Agent Server (mixer)**: e.g., `agt.remixlabs.com`, `agt-dev.remixlabs.com`, `agt-uk.remixlabs.com`
    - GCP project: `rmx-cloudagent-dev`
    - Docker container in GKE, persistent disk
- **Web App Clip Runtime**: `remix.app` — CloudFlare Workers + static assets
- **Mobile App**: iOS App Store + Play Store

### GCP Layout & URL Routing

- Each amp **org = 1 GCP project = 1 customer** — data segregated by project
- Each org has a single K8s cluster with: 1 nginx pod (TLS termination, routing) + 1 amp pod per workspace
- Each amp pod mounts a persistent disk for its database
- Only ports 80/443 open externally (80 redirects to 443), all via nginx
- URL scheme: `{org}[-dev|-test|-staging].remixlabs.com`
    - `/e` — builder/editor
    - `/{app}/{screen}` — runtime
    - `/a/{path}` — amp API (proxied by nginx to amp pod)
    - Static assets served from GCS: `gs://rmx-static/assets/{build}/...`
- Default workspace per org is `prod` (can be omitted from URL)
- Deployment config repo: `remixlabs/roadie`

### Access Control

- Roles stored in `_rmx_admin` app per workspace
- Two roles per app: **editor** (superset) and **user**
- Auth uses asymmetric JWT: only `auth` org has private signing key, all orgs have public verification key
- User identified by email from JWT claims

### CI/CD

- **CircleCI** for builds and deployments
- Artifacts stored in GCS bucket `remix-dev` at `gs://remix-dev/artifacts/{component}/{build}/...`
- Auto-update chain: merging to `main` triggers `update-users.mjs` which cascades updates to dependent components

## rcm (Remix Component Manager)

- Self-updating bash script + JS bundle
- Dependency specs in `rcm.config.mjs`, lock in `rcm-lock.json`
- Dependency types: `build` (immutable tag), `version` (semver), `branch`, `sha`, `local` (with fallback)
- Artifacts installed to `$INSTALL_PREFIX/{bin,lib,assets,scripts}`
- Key commands: `rcm install`, `rcm update`, `rcm comp publish`, `rcm dep install NAME`
- Requires `gcloud` auth + access to GCS bucket `remix-dev`

## amp Server Key Details

- Default ports: 8000 (authenticated), 8001 (admin, unauthenticated)
- Data dir: `server_db/` (configurable via `--database-dir`)
- Root user: set via `RMX_ROOT_USER` env var
- Token generation: `mktoken` utility
- Debug flags: `--flags N` (1=memory, 2=scheduler, 4=state, 8=stack, 16=loop, 32=memory-old, 64=GC, 128=instruction trace)
- Key internal endpoints: `/x/build`, `/x/sha1`, `/x/version`, `/x/health/*`, `/x/debug/pprof/*`
- Profiling: `?trace=` for timing logs, `?profile=name` for CPU profiles
- Build: `rcm install && make setup amp`

## mixer (Cloud Agent Server) Key Details

- Binary: `mixer serve`
- Activity logging in `_rmx_logging` db per workspace
- Metering: provisioned users + anonymous invocations/month
- S3-compatible files backend via env vars: `MIXER_S3_ENDPOINT`, `MIXER_S3_REGION`, `MIXER_S3_PUBLIC_DOMAIN`, `MIXER_S3_ACCESS_KEY`, `MIXER_S3_SECRET_KEY`, `MIXER_S3_BUCKET_NAME`
- Local dev: use ngrok for HTTPS, or amp with `-dangerous-http-mode`

## Prod Environment — Named Servers & Roles

> Notion: [Components of the "prod environment"](https://www.notion.so/4d9def6449d84f72ba523d6b46001d25)
> Also: [Inventory of prod assets](https://www.notion.so/13c1d464528f8080a220f64048c41ad1)

Two main parts of any prod deployment: **platform** and **content**.

### Known Studio Server (amp) Instances

| Host                          | Role                                                                  |
|-------------------------------|-----------------------------------------------------------------------|
| `remix.remixlabs.com`         | Remix Labs internal content building                                  |
| `remix-india.remixlabs.com`   | Widget template building                                              |
| `community.remixlabs.com`     | Auto-update flow for container mobile app; some public-facing things  |
| `poc.remixlabs.com`           | Early partner/trial customers ("proof of concept")                    |
| `nevista.remixlabs.com`       | Nevista customer prod (restaurant apps)                               |
| `xcaliber.remixlabs.com`      | Xcaliber customer                                                     |
| `gainsolutions.remixlabs.com` | Gain Solutions / Joey et al.                                          |
| `auth.remixlabs.com`          | Centralized auth server (technically amp, updated on its own cadence) |
| `remix-dev.remixlabs.com`     | Dev environment                                                       |

### Known Agent Server (mixer) Instances

| Host                    | Role                                                            |
|-------------------------|-----------------------------------------------------------------|
| `agt.remixlabs.com`     | Internal, global                                                |
| `agt-uk.remixlabs.com`  | Nevista customer prod (UK); platform agents + restaurant agents |
| `agt-poc.remixlabs.com` | Same users as `poc.remixlabs.com`                               |
| `agt-dev.remixlabs.com` | Dev environment                                                 |

### Known Customer Workspace IDs

> Notion: [Customer Success Projects](https://www.notion.so/26f1d464528f80f3a6fffc1fe4cf294e)

| Customer         | Workspace ID | Agent Server        |
|------------------|--------------|---------------------|
| Remix (internal) | `6xX25jCW6I` | `agt.remixlabs.com` |
| Funda            | `sEt2qxPydL` | `agt.remixlabs.com` |
| Orderly          | `G4dLBeuE4V` | `agt.remixlabs.com` |
| Bomisco          | `0ry4DirCMQ` | `agt.remixlabs.com` |
| 5 Star Music     | `Hafoe2ekMN` | `agt.remixlabs.com` |
| Lumber           | `iaEj4QYboi` | `agt.remixlabs.com` |

- Google Shared Drive for customer success projects: `https://drive.google.com/drive/folders/0AABUg1qaj6gfUk9PVA`

### Customer Use Case Notes

> Notion: [Customer Success Projects](https://www.notion.so/26f1d464528f80f3a6fffc1fe4cf294e)

- **Funda** (`sEt2qxPydL`) — WhatsApp community of ~500 Indian founders ([funda.club](https://www.funda.club/)). Phase 1: member directory — admins (2 named users) maintain profiles via LinkedIn
  clipping, members (~600 anonymous) browse/search directory and contact members. Phase 2: network summary reports. Reusable services: Snowflake Cortex semantic search, general person+company data
  model (underpins CRM/directory/sales), modular LinkedIn clipper configs.
- **Orderly** (`G4dLBeuE4V`) — Early-stage hotel concierge startup, 1 hotel in Mountain View. Phase 1: Chrome extension scraping WebRez PMS → Orderly API + front desk flow for triggering
  checkin/checkout concierge. Remix workspace holds no customer data (only logs). Future: widgets, AI agentic flows for guests.
- **Bomisco** (`0ry4DirCMQ`) — LinkedIn → HubSpot contact posting. Chrome extension for clipping LinkedIn profiles, editing, and pushing to HubSpot as contacts. Old workspace corrupted (`ET8fjwrW2h`).
- **5 Star Music** (`Hafoe2ekMN`) — Music school (50–60 students, 2–3 teachers). Twilio IVR → SMS with Remix digital business experience. ~10 inbound calls/day. DBP: `remix.app/about/5starmusic`. IVR
  contact form: `remix.app/about/5starmusic/ivr_contact_form`. Staff review inbound requests via Remix mobile app + web app.
- **Lumber** (`iaEj4QYboi`) — Construction labor tracking, job costing, payroll. Snowflake agents at `remix-dev.remixlabs.com/e/edit/snowflake`. Reporting at
  `remix-india.remixlabs.com/e/edit/lumber_reporting`. Catalog: `remix.app/run?_rmx_url=https://agt.files.remix.app/iaEj4QYboi/_rmx_files/apps/catalog.remix`. See [lumber.md](lumber.md).
- **Nevista** — Restaurant apps (UK). Dedicated amp (`nevista.remixlabs.com`) + agent server (`agt-uk.remixlabs.com`).
- **Petavue** — Ball in their court. India team getting up to speed on Arvind's implementation pattern (reusable UI delivery model).

### Mobile Apps

- **Primary Remix app** (iOS/Android) — managed by `flutter-runtime` repo
    - Includes all server components except the Mix compiler
    - App clip code uses `mixrun` runtime
    - iOS native app clip relies on App Store version of the mobile app
    - Content sync tied to platform version (compatibility managed this way)
    - App updates not required; users may run old versions — but app clips always use the latest
- **TappedIn** — built from `remix-swiftui-template` repo

### Desktop App

- **Remix Desktop** — Tauri-based, Mac & Windows (managed by `turntable` repo)
- Local cache: `~/Library/Application Support/com.remixlabs.desktop` (clear to reset)
- Desktop releases URL: `https://remix.app/remix/desktop-releases?channel=dev|beta|release`
- Slack channel: `#desktop`

### Web App Clip & Web Assets

- `remix.app` — CloudFlare Workers + static assets (managed by `remix.app` repo)
    - Worker is stateless; updates deploy with zero downtime
    - Wrapper around turntable web component → mixcore + groovebox
- Static files in CloudFlare R2 and GCS (documentation incomplete)
- `remixlabs.com` web assets including `run.html` and `app.html` (managed in `website` repo)
- **Chrome extension** — documentation incomplete
- **Push Messaging** — Browser-based push notifications via Google Firebase (foreground + background). Works on Safari (iOS, must be "added to desktop"), Chrome/Firefox (Android, Windows, macOS,
  Linux). Docs: `turntable` repo `webapp/` README.

## Prod Update Cadence (Weekly)

| Step | Component                                   | Downtime                                                            |
|------|---------------------------------------------|---------------------------------------------------------------------|
| 1    | Mobile app → App Store / Play Store         | None (store rollout)                                                |
| 2    | Web runtime assets → CloudFlare             | None (stateless worker)                                             |
| 3    | Studio server frontend (nginx update)       | Few seconds                                                         |
| 4    | Studio server backend (new container image) | ~15–20 sec (single pod; only one pod can attach to DB disk at once) |
| 5    | Agent server (new build, pod restart)       | ~5 sec (single-node; LB returns 502s during transition)             |

- Auth server is technically a studio server but updated on its own cadence (only when auth-relevant features change)
- Content release does NOT need to be coordinated with mobile app release — content can be released in advance to test beta builds

## GitHub Project Boards

- Platform: `github.com/orgs/remixlabs/projects/2/views/1`
- Content, Flows & Feedback: `github.com/orgs/remixlabs/projects/3`

## Key Terminology

- **Mix**: The programming language used in Remix
- **App Clip**: Lightweight runnable app from a .remix file (iOS native or web)
- **Flow**: A Mix program / visual code built in Remix Studio
- **Builder**: The visual IDE (Remix Studio) for creating flows
- **Workspace**: Runtime namespace for apps, agents, and data
- **Org**: Billing account (not part of workspace structure)
- **Agent**: A headless Mix program that runs on the cloud agent server (mixer)
- **Track**: Go FFI library in amp for Mix compiler/runtime API
