# Server APIs — Cloud Agent Server API

> Sources:
> - [Cloud Agent Server API](https://www.notion.so/1061d464528f8022aa1cec129096c82b) — Cloud Agent Server > API
> - [Cloud Agent Server v1 API](https://www.notion.so/1bd1d464528f80cb97cde448c9e2a944) — Cloud Agent Server > v1 API
    > Parent: Remix Documentation > User Guides and Tutorials > Remix Studio User Guide

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

### Workspace Import/Export/Clone

| Method | Endpoint                                           | Description                                   |
|--------|----------------------------------------------------|-----------------------------------------------|
| `GET`  | `/ws-export/<ws>`                                  | Export workspace as .remix mastertape         |
| `POST` | `/ws-import?ws=<ws>`                               | Import mastertape (body); optional `ws` name  |
| `POST` | `/ws-clone/<ws1>?ws=<ws2>&url=<url>&token=<token>` | Clone workspace; optionally to a remote mixer |

### Core Operations

| Method   | Endpoint                        | Description                                                                                                                                                          |
|----------|---------------------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `POST`   | `/install-remix-file/<ws>`      | Install .remix file (body: octet-stream or JSON `{url, force}`)                                                                                                      |
| `POST`   | `/grant-permission/<ws>`        | Grant permission (`subject`, `role`, `resource`)                                                                                                                     |
| `POST`   | `/run-agent/<ws>/<app>/<agent>` | Run agent (JSON body = input params)                                                                                                                                 |
| `POST`   | `/query/<ws>/<app>`             | Register cloud query (`name`, `ast`, optional `db`, `skip`, `limit`, `include_superseded`, `include_deleted`)                                                        |
| `DELETE` | `/query/<ws>/<app>/<name>`      | Delete cloud query                                                                                                                                                   |
| `POST`   | `/count-activities/<ws>`        | Count activities; optional filters: `subject`, `activity` (`run_agent\|update_db\|install_remix_file\|create_workspace\|grant_permission\|remixgen`), `start`, `end` |
| `GET`    | `/subscribe/<topic>`            | Subscribe to cloud topic (SSE: `data:"<JSON>"` per message, `:` heartbeats)                                                                                          |

### .remix File Generation

`POST /remixgen/<ws>/<app>` — Generate .remix by injecting query results into an existing .remix template.

Params: `src` or `url` (source .remix), `dest` (output path), `query` (query string), `overrides` (JSON: app → constants overrides).

---

## Cloud Agent Server v1 API

Swagger: `https://agt.remixlabs.com/v1/swagger/`

### Workspace/App/Agent Introspection

| Method | Endpoint                              | Response                                                                       |
|--------|---------------------------------------|--------------------------------------------------------------------------------|
| `GET`  | `/v1/ws`                              | `{workspaces: [{name}]}`                                                       |
| `GET`  | `/v1/ws/<ws>`                         | `{name, apps: [{name}]}`                                                       |
| `GET`  | `/v1/ws/<ws>/app/<app>`               | `{name, agents: [{name}]}`                                                     |
| `GET`  | `/v1/ws/<ws>/app/<app>/agent/<agent>` | `{name, description, inParams: [{type, name, displayName}], outParams: [...]}` |

### Permissions CRUD

Available at workspace, app, and agent levels:

| Method   | Endpoint Pattern               | Description             |
|----------|--------------------------------|-------------------------|
| `GET`    | `/v1/ws/<ws>/permissions`      | List permissions        |
| `GET`    | `/v1/ws/<ws>/permissions/<id>` | Get permission by ID    |
| `DELETE` | `/v1/ws/<ws>/permissions/<id>` | Delete permission (204) |

Same pattern for apps (`/v1/ws/<ws>/app/<app>/permissions[/<id>]`) and agents (`/v1/ws/<ws>/app/<app>/agent/<agent>/permissions[/<id>]`).

Permission object: `{id, subject, role, resource}` where:

- `subject`: `"user/<email>"` or similar
- `role`: `admin`, `editor`, `runner`, etc.
- `resource`: `"workspace"`, `"db/<app>"`, `"agent/<app>/<agent>"`

### Files API (v1)

| Method   | Endpoint                                                                 | Description                                       |
|----------|--------------------------------------------------------------------------|---------------------------------------------------|
| `GET`    | `/v1/ws/<ws>/app/<app>/files/<path>`                                     | Serve file (public: no auth)                      |
| `GET`    | `/v1/ws/<ws>/app/<app>/file-manager/content?path=<path>`                 | Get file or list directory                        |
| `PUT`    | `/v1/ws/<ws>/app/<app>/file-manager/content?path=<path>&force=<bool>`    | Create file                                       |
| `POST`   | `/v1/ws/<ws>/app/<app>/file-manager/content`                             | Create file (multipart: `path`, `data`, `force`)  |
| `DELETE` | `/v1/ws/<ws>/app/<app>/file-manager/content?path=<path>`                 | Delete file                                       |
| `GET`    | `/v1/ws/<ws>/app/<app>/file-manager/props?path=<path>`                   | Get file properties                               |
| `PATCH`  | `/v1/ws/<ws>/app/<app>/file-manager/props?path=<path>&change_path=<new>` | Rename file                                       |
| `POST`   | `/v1/ws/<ws>/app/<app>/file-manager/copy`                                | Copy file (`src`, `src_app`, `path`)              |
| `POST`   | `/v1/ws/<ws>/app/<app>/file-manager/register`                            | Register file (`path`, `reg_url`, `reg_metadata`) |

### Signals

Promise-based coordination between agents and screens.

| Method   | Endpoint           | Description                                                                                             |
|----------|--------------------|---------------------------------------------------------------------------------------------------------|
| `POST`   | `/v1/signals`      | Create signal → `{url: "/v1/signals/<id>"}`                                                             |
| `GET`    | `/v1/signals/<id>` | Await signal value (blocking); returns `ok(value)` or `error("canceled"\|"timeout")` as Mix tagged JSON |
| `POST`   | `/v1/signals/<id>` | Send signal value (JSON body)                                                                           |
| `DELETE` | `/v1/signals/<id>` | Cancel signal                                                                                           |

**Typical flow:** Agent creates signal → passes signal object to screen → agent awaits signal → screen sends value → agent receives it.

All signal endpoints require non-anonymous Remix token.

### Workspace Import/Export (v1)

| Method | Endpoint                | Description                                                |
|--------|-------------------------|------------------------------------------------------------|
| `GET`  | `/v1/ws/<ws>/export`    | Download mastertape                                        |
| `POST` | `/v1/ws-import?id=<ws>` | Upload mastertape (body); optional `id` for workspace name |
| `POST` | `/v1/ws/<ws>/export`    | Export to remote URL (JSON body: `{url, ws?, token?}`)     |
| `POST` | `/v1/ws-import`         | Import from remote URL (JSON body: `{url, ws?, token?}`)   |

**Clone patterns:**

- Push: export to `<remote>/v1/ws-import`
- Pull: import from `<remote>/v1/ws/<ws>/export`