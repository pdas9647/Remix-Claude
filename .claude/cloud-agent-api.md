# Cloud Agent Server (Mixer) — API Reference

> Notion: [Cloud Agent Server](https://www.notion.so/1061d464528f80f18e46dd27895aeda2)
> Subpages: [Cloud Agent Server API](https://www.notion.so/1061d464528f8022aa1cec129096c82b), [Cloud Agent Server v1 API](https://www.notion.so/1bd1d464528f80cb97cde448c9e2a944)
> Swagger: `https://agt.remixlabs.com/v1/swagger/`

---

## Overview

- Lambda-like runtime for agent execution — no HTTP DB API, no persistent sessions
- Drop-in replacement for Studio Server **for apps that only contain agents**
- Agents are the same as elsewhere in the platform; can be invoked locally or deployed to cloud
- Workspaces = database namespace + metering unit

## Auth & Input Conventions

- All requests (except anonymous agents) require: `Authorization: Bearer $TOKEN`
- Input params via URL-encoded form body or JSON POST body (not query params, unless noted)
- Exception: `file` param in `/install-remix-file/<ws>` must be a form parameter

### Error Responses

- Agent execution failure: `{"kind": "run_error", "run_error": <JSON>}`
- Other failure: `{"ok": false, "kind": "message", "message": <STRING>}`

## Access Control (RBAC)

Permission grants are triples of `{subject, role, resource}`:

### Subjects

| Format                             | Description                                                   |
|------------------------------------|---------------------------------------------------------------|
| `user/USER_EMAIL`                  | Specific user                                                 |
| `domain/HOST`                      | All users with email at domain (e.g., `domain/remixlabs.com`) |
| `agent/SERVER:WORKSPACE.APP.AGENT` | Agent-to-agent permission                                     |
| `all-users`                        | Any authenticated user                                        |
| `anonymous`                        | No authentication required                                    |

### Roles & Permissions

| Role         | Permissions                                                    | Applicable Resources                   |
|--------------|----------------------------------------------------------------|----------------------------------------|
| `runner`     | run                                                            | `agent/DB/AGENT`, `db/DB`, `workspace` |
| `editor`     | run, export, read, write                                       | `db/DB`, `workspace`                   |
| `admin`      | run, export, read, write, grant_permissions, delete, create_db | `db/DB`, `workspace`                   |
| `db/creator` | create_db                                                      | `workspace`                            |

### Key Rules

- **Owner vs Admin**: Ownership is immutable, tied to billing. Owner = creator. Admins can grant permissions but cannot change ownership.
- **Workspace-level** `runner` permission implies all DBs; **DB-level** `runner` implies all agents in that DB.
- **No intra-app checks**: Agent X calling agent Y within the same app always succeeds if X was invokable. Internal calls are "transparent" to permissions.
- **Agent chaining**: If user A invokes agent X which calls agent Y (different app), allowed if A or X has permission on Y.
- **Installing .remix files** requires `admin` role on the workspace.

---

## Core API Endpoints

### Health & Workspace Management

| Method                  | Endpoint                                 | Description |
|-------------------------|------------------------------------------|-------------|
| `GET /`                 | Health check (includes build number)     |
| `GET /ws/`              | List workspaces + permissions (admin)    |
| `GET /ws/<ws>`          | List apps in workspace (admin)           |
| `POST /ws`              | Create workspace (optional `name` param) |
| `DELETE /ws/<ws>`       | Delete workspace                         |
| `DELETE /ws/<ws>/<app>` | Delete app                               |

### Agent Execution

| Method                               | Endpoint                          | Description |
|--------------------------------------|-----------------------------------|-------------|
| `POST /run-agent/<ws>/<app>/<agent>` | Run agent with JSON body as input |

### Permissions

| Method                        | Endpoint                                                | Description |
|-------------------------------|---------------------------------------------------------|-------------|
| `POST /grant-permission/<ws>` | Grant permission (`subject`, `role`, `resource` params) |

### .remix File Operations

| Method                          | Endpoint                                                           | Description |
|---------------------------------|--------------------------------------------------------------------|-------------|
| `POST /install-remix-file/<ws>` | Install .remix file (octet-stream or `{"url": "..."}`)             |
| `POST /remixgen/<ws>/<app>`     | Generate .remix file by injecting query results into existing file |

`remixgen` params: `src`/`url` (source .remix), `dest` (output path), `query` (filter string), `overrides` (JSON: constants per app).

### Cloud Queries

| Method                            | Endpoint                                                                                                      | Description |
|-----------------------------------|---------------------------------------------------------------------------------------------------------------|-------------|
| `POST /query/<ws>/<app>`          | Register cloud query (`name`, `ast`, optional `db`, `skip`, `limit`, `include_superseded`, `include_deleted`) |
| `DELETE /query/<ws>/<app>/<name>` | Delete cloud query                                                                                            |

Cloud queries are served at `POST /run-agent/<ws>/<app>/<name>` with parameter values as input. Same permission checks as agents.

### Workspace Import/Export

| Method                                                  | Endpoint                                    | Description |
|---------------------------------------------------------|---------------------------------------------|-------------|
| `GET /ws-export/<ws>`                                   | Export workspace as .remix mastertape       |
| `POST /ws-import?ws=<ws>`                               | Import mastertape (optional workspace name) |
| `POST /ws-clone/<ws1>?ws=<ws2>&url=<url>&token=<token>` | Clone workspace (optional remote target)    |

### Activity Logging

| Method                        | Endpoint                                                                   | Description |
|-------------------------------|----------------------------------------------------------------------------|-------------|
| `POST /count-activities/<ws>` | Count activities (optional filters: `subject`, `activity`, `start`, `end`) |

Activity types: `run_agent`, `update_db`, `install_remix_file`, `create_workspace`, `grant_permission`, `remixgen`.

### Cloud Topics (SSE)

| Method                   | Endpoint                                                       | Description |
|--------------------------|----------------------------------------------------------------|-------------|
| `GET /subscribe/<topic>` | Subscribe to cloud topic (SSE: `data:"<JSON>"`, heartbeat `:`) |

---

## v1 API Endpoints

Newer REST-style endpoints under `/v1` prefix.

### Introspection

| Method                                    | Endpoint                               | Description |
|-------------------------------------------|----------------------------------------|-------------|
| `GET /v1/ws`                              | List workspaces                        |
| `GET /v1/ws/<ws>`                         | Workspace info + app list              |
| `GET /v1/ws/<ws>/app/<app>`               | App info + agent list                  |
| `GET /v1/ws/<ws>/app/<app>/agent/<agent>` | Agent info (name, inParams, outParams) |

### Permissions (v1)

| Method                                                        | Endpoint                   | Description |
|---------------------------------------------------------------|----------------------------|-------------|
| `GET /v1/ws/<ws>/permissions`                                 | List workspace permissions |
| `GET /v1/ws/<ws>/permissions/<id>`                            | Get permission by ID       |
| `DELETE /v1/ws/<ws>/permissions/<id>`                         | Delete permission by ID    |
| `GET /v1/ws/<ws>/app/<app>/permissions`                       | List app permissions       |
| `DELETE /v1/ws/<ws>/app/<app>/permissions/<id>`               | Delete app permission      |
| `GET /v1/ws/<ws>/app/<app>/agent/<agent>/permissions`         | List agent permissions     |
| `DELETE /v1/ws/<ws>/app/<app>/agent/<agent>/permissions/<id>` | Delete agent permission    |

### Workspace Import/Export (v1)

| Method                    | Endpoint                                                                              | Description |
|---------------------------|---------------------------------------------------------------------------------------|-------------|
| `GET /v1/ws/<ws>/export`  | Export workspace (download mastertape)                                                |
| `POST /v1/ws-import`      | Import workspace (upload mastertape or `{"url": "...", "ws": "...", "token": "..."}`) |
| `POST /v1/ws/<ws>/export` | Export to remote URL (push-clone: `{"url": "<server>/v1/ws-import", ...}`)            |

### Files API

**`file` data type**: `{app, path, kind, metadata, url}`

- `kind`: `persisted` (server-stored) or `client` (client-only)
- `path`: UNIX-style absolute paths

| Method                                              | Endpoint                                                          | Description |
|-----------------------------------------------------|-------------------------------------------------------------------|-------------|
| `GET /v1/ws/<ws>/app/<app>/files/<path>`            | Serve public files (Content-Type by suffix)                       |
| `GET /v1/ws/<ws>/app/<app>/file-manager/content`    | Get file contents or list directory (`path` param)                |
| `PUT /v1/ws/<ws>/app/<app>/file-manager/content`    | Create file (`path`, optional `force`)                            |
| `POST /v1/ws/<ws>/app/<app>/file-manager/content`   | Create file (multipart: `path`, `data`, `force`)                  |
| `DELETE /v1/ws/<ws>/app/<app>/file-manager/content` | Delete file (`path`)                                              |
| `GET /v1/ws/<ws>/app/<app>/file-manager/props`      | Get file properties                                               |
| `PATCH /v1/ws/<ws>/app/<app>/file-manager/props`    | Rename file (`path`, `change_path`)                               |
| `POST /v1/ws/<ws>/app/<app>/file-manager/copy`      | Copy file (`src`, optional `src_app`, `path`)                     |
| `POST /v1/ws/<ws>/app/<app>/file-manager/register`  | Register external URL as file (`path`, `reg_url`, `reg_metadata`) |

### Signals API

Signals are promises — create, then await or send a value (either order, but only once).

| Method                    | Endpoint                                                          | Description |
|---------------------------|-------------------------------------------------------------------|-------------|
| `POST /v1/signals`        | Create signal → `{"url": "/v1/signals/<id>"}`                     |
| `GET /v1/signals/<id>`    | Await signal value → `ok(value)` or `error("canceled"/"timeout")` |
| `POST /v1/signals/<id>`   | Send signal value (JSON body)                                     |
| `DELETE /v1/signals/<id>` | Cancel signal                                                     |

**Typical pattern**: Agent creates signal → passes signal URL to a screen → awaits value → screen sends value back → agent receives it.

Requires non-anonymous Remix token. Caller must handle HTTP-layer timeouts on GET.

---

## Local Development

```bash
# Start mixer
mixer serve --ws-dir /path/to/workspaces

# With TLS
mkcert 127.0.0.1
mixer serve --ws-dir /path/to/workspaces --tls-certs 127.0.0.1.pem --tls-key 127.0.0.1-key.pem

# Auth setup
AUTH="Authorization: Bearer $TOKEN"
URL="http://127.0.0.1:8000"

# Create workspace
curl -H "$AUTH" "$URL/ws" -F name=ws1

# Install .remix file
curl -H "$AUTH" "$URL/install-remix-file/ws1" --data-binary @/path/to/myapp.remix -H 'Content-type: application/octet-stream'

# Grant anonymous access to an agent
curl -H "$AUTH" "$URL/grant-permission/ws1" -F subject=anonymous -F resource=agent/app1/agent1 -F role=runner

# Run an agent
curl -H "$AUTH" "$URL/run-agent/ws1/myapp/myagent" -d '{"my": "arg"}'
```
