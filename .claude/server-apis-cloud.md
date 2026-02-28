# Server APIs — Cloud Agent Server API

> Sources:
> - [Cloud Agent Server](https://www.notion.so/1061d464528f80f18e46dd27895aeda2) — Architecture hub; last updated 2026-02-26
> - [Cloud Agent Server API](https://www.notion.so/1061d464528f8022aa1cec129096c82b) — Mixer API
> - [Cloud Agent Server v1 API](https://www.notion.so/1bd1d464528f80cb97cde448c9e2a944) — v1 API (Swagger: `https://agt.remixlabs.com/v1/swagger/`)

---

## Architecture Overview

**Workspaces** — database namespace + metering unit. URL form: `https://agt.remixlabs.com/v1/ws/<ws>`.
- **Owner** — immutable, tied to billing; creator of resource. Tier limits apply to owners.
- **Admin** — can grant permissions and create DBs; cannot change ownership.
- Auth server: `auth.remixlabs.com`. One user → many workspaces. Anonymous invocation allowed if app owner permits.

**RBAC** — grants are triples of `(subject, role, resource)`.

Subjects: `user/<email>`, `domain/<host>`, `agent/<server>:<ws>.<app>.<agent>`, `all-users`, `anonymous`

| Role         | Permissions                                                    | Resources                                    |
|--------------|----------------------------------------------------------------|----------------------------------------------|
| `runner`     | run                                                            | `agent/<app>/<agent>`, `db/<app>`, workspace |
| `editor`     | run, export, read, write                                       | `db/<app>`, workspace                        |
| `admin`      | run, export, read, write, grant_permissions, delete, create_db | `db/<app>`, workspace                        |
| `db/creator` | create_db only (cannot read/replace existing)                  | workspace                                    |

- `.remix` install requires `admin` on workspace.
- `anonymous` = no token; `all-users` = any authenticated user.
- Agent-to-agent: cross-app calls allowed if calling user OR calling agent has permission. Same-app calls always allowed.
- Mixer startup: `mixer serve --ws-dir /path/to/workspaces` (add `--tls-certs`/`--tls-key` for HTTPS).

---

## Cloud Agent Server API (Mixer)

### Workspace Management

| Method   | Endpoint         | Description                                      |
|----------|------------------|--------------------------------------------------|
| `GET`    | `/`              | Health check (includes build number)             |
| `GET`    | `/ws/`           | List workspaces (admin only)                     |
| `GET`    | `/ws/<ws>`       | List apps in workspace (admin only)              |
| `POST`   | `/ws`            | Create workspace; optional `name` → `{ok, name}` |
| `DELETE` | `/ws/<ws>`       | Delete workspace                                 |
| `DELETE` | `/ws/<ws>/<app>` | Delete app                                       |

### Workspace Import / Export / Clone

| Method | Endpoint                                           | Description                       |
|--------|----------------------------------------------------|-----------------------------------|
| `GET`  | `/ws-export/<ws>`                                  | Export as .remix mastertape       |
| `POST` | `/ws-import?ws=<ws>`                               | Import mastertape (body)          |
| `POST` | `/ws-clone/<ws1>?ws=<ws2>&url=<url>&token=<token>` | Clone; optionally to remote mixer |

### Core Operations

| Method   | Endpoint                        | Description                                                                                               |
|----------|---------------------------------|-----------------------------------------------------------------------------------------------------------|
| `POST`   | `/install-remix-file/<ws>`      | Install .remix (body: octet-stream or JSON `{url, force}`)                                                |
| `POST`   | `/grant-permission/<ws>`        | Grant permission (`subject`, `role`, `resource`)                                                          |
| `POST`   | `/run-agent/<ws>/<app>/<agent>` | Run agent (JSON body = input params)                                                                      |
| `POST`   | `/query/<ws>/<app>`             | Register cloud query (`name`, `ast`, opt: `db`, `skip`, `limit`, `include_superseded`, `include_deleted`) |
| `DELETE` | `/query/<ws>/<app>/<name>`      | Delete cloud query                                                                                        |
| `POST`   | `/count-activities/<ws>`        | Count activities; filters: `subject`, `activity`, `start`, `end`                                          |
| `GET`    | `/subscribe/<topic>`            | Subscribe to cloud topic (SSE)                                                                            |

`POST /remixgen/<ws>/<app>` — inject query results into a .remix template. Params: `src`/`url`, `dest`, `query`, `overrides` (JSON: app → constants).

---

## Cloud Agent Server v1 API

### Introspection

| `GET` Endpoint                        | Response                                   |
|---------------------------------------|--------------------------------------------|
| `/v1/ws`                              | `{workspaces: [{name}]}`                   |
| `/v1/ws/<ws>`                         | `{name, apps: [{name}]}`                   |
| `/v1/ws/<ws>/app/<app>`               | `{name, agents: [{name}]}`                 |
| `/v1/ws/<ws>/app/<app>/agent/<agent>` | `{name, description, inParams, outParams}` |

### Permissions CRUD

Available at workspace, app, and agent levels (pattern: `.../permissions[/<id>]`).

| Method         | Description             |
|----------------|-------------------------|
| `GET`          | List all permissions    |
| `GET /<id>`    | Get permission by ID    |
| `DELETE /<id>` | Delete permission (204) |

Permission object: `{id, subject, role, resource}`.

### Files API (v1)

Base prefix: `/v1/ws/<ws>/app/<app>/`

| Method   | Path                                    | Description                                       |
|----------|-----------------------------------------|---------------------------------------------------|
| `GET`    | `files/<path>`                          | Serve file (public, no auth)                      |
| `GET`    | `file-manager/content?path=`            | Get file or list directory                        |
| `PUT`    | `file-manager/content?path=&force=`     | Create file                                       |
| `POST`   | `file-manager/content`                  | Create file (multipart: `path`, `data`, `force`)  |
| `DELETE` | `file-manager/content?path=`            | Delete file                                       |
| `GET`    | `file-manager/props?path=`              | Get file properties                               |
| `PATCH`  | `file-manager/props?path=&change_path=` | Rename file                                       |
| `POST`   | `file-manager/copy`                     | Copy file (`src`, `src_app`, `path`)              |
| `POST`   | `file-manager/register`                 | Register file (`path`, `reg_url`, `reg_metadata`) |

### Signals

Promise-based agent↔screen coordination. All endpoints require non-anonymous token.

| Method   | Endpoint           | Description                                               |
|----------|--------------------|-----------------------------------------------------------|
| `POST`   | `/v1/signals`      | Create signal → `{url: "/v1/signals/<id>"}`               |
| `GET`    | `/v1/signals/<id>` | Await value (blocking) → `ok(value)` or `error("canceled" |"timeout")` |
| `POST`   | `/v1/signals/<id>` | Send signal value (JSON body)                             |
| `DELETE` | `/v1/signals/<id>` | Cancel signal                                             |

### Workspace Import / Export (v1)

| Method | Endpoint                | Description                                    |
|--------|-------------------------|------------------------------------------------|
| `GET`  | `/v1/ws/<ws>/export`    | Download mastertape                            |
| `POST` | `/v1/ws-import?id=<ws>` | Upload mastertape                              |
| `POST` | `/v1/ws/<ws>/export`    | Push export to remote (`{url, ws?, token?}`)   |
| `POST` | `/v1/ws-import`         | Pull import from remote (`{url, ws?, token?}`) |

Clone: Push = export to `<remote>/v1/ws-import`; Pull = import from `<remote>/v1/ws/<ws>/export`.