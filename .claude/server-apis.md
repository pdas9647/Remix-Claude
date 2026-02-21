# Server APIs — Auth Integrations, Platform Server, Cloud Agent Server

> Sources:
> - [Auth integrations](https://www.notion.so/11f1d464528f8070b010f945a0b0e7af) — Studio Server > Auth integrations
> - [Legacy Platform Server API](https://www.notion.so/1061d464528f80449377e6c07e72b30b) — Studio Server > Legacy Platform Server API
> - [Cloud Agent Server API](https://www.notion.so/1061d464528f8022aa1cec129096c82b) — Cloud Agent Server > API
> - [Cloud Agent Server v1 API](https://www.notion.so/1bd1d464528f80cb97cde448c9e2a944) — Cloud Agent Server > v1 API
    > Parent: Remix Documentation > User Guides and Tutorials > Remix Studio User Guide

---

## Auth Integrations

### Supported Auth Plugin Types

| Type         | Description                                           | PKCE |
|--------------|-------------------------------------------------------|------|
| `oauth`      | Generic OAuth 2.0                                     | Yes  |
| `oidc`       | OpenID Connect                                        | Yes  |
| `apple`      | Sign In With Apple                                    | N/A  |
| `sfdc_oauth` | Salesforce OAuth (special handling — no OIDC support) | Yes  |

### Configuration Schema

```json
{
  "type": "oauth|oidc|apple|sfdc_oauth",
  "name": "<plugin_name>",
  "client_id": "<client_id>",
  "client_secret": "<client_secret>",
  "auth_url": "<authorization_endpoint>",
  "token_url": "<token_endpoint>",
  "userinfo_url": "<userinfo_endpoint>",
  "endpoint": "<oidc_discovery_endpoint>",
  "scopes": [
    "openid",
    "email",
    ...
  ],
  "default": false,
  "use_for_amp_login": false,
  "store_token": false,
  "offline": false,
  "domain": "<auto_attach_domain>",
  "token_extras": [
    "instance_url"
  ],
  "token_lifetime": 0,
  "force_refresh_after": 0,
  "auth_url_params": {
    "prompt": "consent"
  },
  "headers": {
    "user-agent": [
      "com.remixlabs.runtime/1.0"
    ]
  },
  "callback": "<override_callback_url>"
}
```

### Key Configuration Fields

| Field                     | Effect                                                                                                                                                |
|---------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------|
| `default: true`           | Makes this the default auth flow (`/x/auth` redirects here)                                                                                           |
| `use_for_amp_login: true` | Auth flow generates a Remix token (identity = email from userinfo)                                                                                    |
| `store_token: true`       | Saves 3rd-party token encrypted in DB; accessible via `secrets.getToken(name)` and `/x/auth/{name}/originalToken`                                     |
| `offline: true`           | Requests refresh token; adds `access_type=offline` to auth URL                                                                                        |
| `domain`                  | HTTP requests to this domain auto-attach the stored token if no Authorization header present                                                          |
| `token_extras`            | Fields extracted from OAuth token response and stored alongside (e.g. `instance_url` for Salesforce); accessible via `secrets.tokenExtra(tok, "key")` |
| `token_lifetime`          | Remix token lifetime in seconds (default: 90 days / 7,776,000s)                                                                                       |
| `force_refresh_after`     | Seconds after which to refresh access tokens if no expiration was provided by the IDP                                                                 |

### Callback URLs per Surface

| Surface                | Callback URL Pattern                                   |
|------------------------|--------------------------------------------------------|
| **Web (hosted)**       | `https://<env>.remixlabs.com/a/x/auth/<name>/callback` |
| **Desktop**            | `http://localhost:3434/a/x/auth/<name>/callback`       |
| **Mobile / local dev** | `http://localhost:8000/x/auth/<name>/callback`         |

Some IDPs allow multiple callbacks in one config; others require separate integrations per surface.

### Server API — Auth Config Management

**Public configs** (can be distributed to client apps):

| Method             | Endpoint                      | Access             |
|--------------------|-------------------------------|--------------------|
| `GET`              | `/x/authConfig/public`        | Anyone             |
| `GET`              | `/x/authConfig/public/{name}` | Anyone             |
| `POST`             | `/x/authConfig/public`        | Any signed-in user |
| `PUT/PATCH/DELETE` | `/x/authConfig/public/{name}` | Creator or admin   |

**Confidential configs** (server-only):

| Method                      | Endpoint                 | Access     |
|-----------------------------|--------------------------|------------|
| `GET/POST/PUT/PATCH/DELETE` | `/x/authConfig[/{name}]` | Admin only |

### Auth Flow URLs

| Purpose                 | URL                                                              |
|-------------------------|------------------------------------------------------------------|
| Default sign-in         | `https://<env>.remixlabs.com/a/x/auth?redirect=<url>`            |
| Named provider          | `https://<env>.remixlabs.com/a/x/auth/<provider>?redirect=<url>` |
| Token exchange          | `GET                                                             |POST /x/auth/{handler}/token` (pass `token` or `idToken` param) |
| Get 3rd-party token     | `GET                                                             |POST /x/auth/{handler}/originalToken` (authenticated; refreshes if possible) |
| Delete stored token     | `secrets.deleteToken(name)` in Mix                               |
| Per-app default sign-in | Set `settings.signin-screen` in app metadata                     |

### auth0 Setup (for username/password flows)

1. Create auth0 tenant `rmx-<customer>` → create application
2. Add `oidc` plugin config with client ID/secret from auth0 app
3. Optional: Google sign-in via auth0 Google connection (needs GCP OAuth app)
4. Integrate email provider for auth0 emails
5. Enable "Password" Grant Type in auth0 app Advanced Settings for Mix login FFIs
6. Set tenant "Default Directory" to `Username-Password-Authentication`

---

## Legacy Platform Server API (Amp / Studio Server)

### General Notes

- Parameters accepted as: URL-encoded body > query params > multipart form (first wins)
- All app endpoints: `/<app>/*` (app name ≥ 2 chars, no leading/trailing whitespace)
- `inmem=true` query param → in-memory app (separate namespace, ephemeral)
- Encodings: `application/json` (default), `application/msgpack`, `application/msgpack+base64`
- Text output: `Accept: text/html|text/javascript|text/plain` (for non-object responses)

### Tagged Types (Special JSON Representations)

| Tag                | Format                                                    | Purpose                      |
|--------------------|-----------------------------------------------------------|------------------------------|
| `{tag}bin`         | `{"_rmx_type": "{tag}bin", "data": "<base64>"}`           | Binary data                  |
| `{tag}case:db.ref` | `{"_rmx_type": "{tag}case:db.ref", "_rmx_value": "<id>"}` | DB record reference          |
| `{tag}blob`        | `{"_rmx_type": "{tag}blob", "data": "..."}`               | Nested object → own document |
| `{tag}blob:<mime>` | Blob with MIME type annotation                            | Sets `_rmx_content_type`     |
| `{tag}undefined`   | `{"_rmx_type": "{tag}undefined}"}`                        | Mix `undefined` value        |
| `{str}...`         | Escape prefix                                             | Prevents tag interpretation  |

### Document Metadata (auto-assigned)

`_rmx_id`, `_rmx_created_by`, `_rmx_created_at`, `_rmx_last_modified_by`, `_rmx_last_modified_at`, `_rmx_rev`, `_rmx_row`, `_rmx_deleted`, `_rmx_previous` (if update)

### Document CRUD

| Method   | Endpoint                           | Notes                                                                                            |
|----------|------------------------------------|--------------------------------------------------------------------------------------------------|
| `POST`   | `/<app>/documents`                 | Create (returns ID; array input → newline-separated IDs). Supports batch temp IDs for cross-refs |
| `GET`    | `/<app>/documents/<id>`            | Read by ID                                                                                       |
| `PUT`    | `/<app>/documents/<id>`            | Replace (full overwrite)                                                                         |
| `PATCH`  | `/<app>/documents/<id>`            | Upsert (merge)                                                                                   |
| `DELETE` | `/<app>/documents/<id>`            | Soft delete (204)                                                                                |
| `DELETE` | `/<app>/documents?ids=<json_list>` | Batch delete → returns `{deleted, not_found, error}`                                             |

### Queries

| Method | Endpoint                            | Notes                                                                       |
|--------|-------------------------------------|-----------------------------------------------------------------------------|
| `GET`  | `/<app>/query?query=<query_string>` | Query with optional projections. Params: `includeDeleted`, `includeHistory` |

Full query syntax: [Query syntax](https://www.notion.so/205ca411bf684f5593c3469e39de2af8) (stored in remix-nodes.md)

### App Management

| Method   | Endpoint | Notes                                      |
|----------|----------|--------------------------------------------|
| `POST`   | `/<app>` | Create app (body = metadata JSON)          |
| `GET`    | `/<app>` | Get app metadata                           |
| `PUT`    | `/<app>` | Update app metadata                        |
| `DELETE` | `/<app>` | Delete app                                 |
| `GET`    | `/`      | List all apps (user has builder access to) |

### Multi-App Actions (`GET|POST /x/apps?action=<action>`)

| Action            | Required Params      | Description                                                    |
|-------------------|----------------------|----------------------------------------------------------------|
| `compact`         | `orig`, `dest`       | Copy without history (default: no superseded/deleted)          |
| `copy`            | `orig`, `dest`       | Copy with all history (default)                                |
| `export`          | `orig`               | Download as ZIP (no history by default)                        |
| `fix`             | `orig`, `dest`       | Corruption fix; optional `index` reindex                       |
| `garbage-collect` | `orig`               | In-place prune superseded/deleted; optional `cutoff` timestamp |
| `package`         | `orig`               | Export metadata + executables as Msgpack; optional `bare`      |
| `reindex`         | `orig`               | In-place index rebuild                                         |
| `rename`          | `orig`, `dest`       | Rename app                                                     |
| `remote-export`   | `orig`, `dest` (URL) | Export + upload to URL                                         |
| `remote-package`  | `orig`, `dest` (URL) | Package + upload to URL; optional `gzip`                       |
| `remote-import`   | `orig` (URL), `dest` | Download + import                                              |

### .remix File Endpoints

| Method | Endpoint                 | Description                                                                                                                                                       |
|--------|--------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `POST` | `/x/apps/export`         | Export .remix (manifest body: `apps`, `records`, `metadata`, `setup`, `bare`, `builderAssets`, `databases`, `overrides`); also supports GET with `manifest` param |
| `POST` | `/x/apps/upload`         | Upload .remix → returns `{id}`                                                                                                                                    |
| `POST` | `/x/apps/install`        | Install uploaded .remix by `id`; optional `force` to overwrite newer                                                                                              |
| `POST` | `/x/apps/remote-export`  | Export .remix → upload to `url`                                                                                                                                   |
| `POST` | `/x/apps/remote-fetch`   | Download .remix from `url` → returns `{id}`                                                                                                                       |
| `POST` | `/x/apps/remote-install` | Download + install from `url`                                                                                                                                     |

**Install logic:** Only overwrites if `last_published_at` ≥ existing, unless `force=true`. Records always inserted/passed to `_rmx_setup`.

### Files API

| Method   | Endpoint                                  | Description                                                      |
|----------|-------------------------------------------|------------------------------------------------------------------|
| `GET`    | `/<app>/files/<path>`                     | Serve file (public: no auth; private: signed URL)                |
| `GET`    | `/<app>/files?path=<path>`                | List (if ends with `/`) or read file                             |
| `POST`   | `/<app>/files`                            | Create/copy file (multipart: `path` + `data`, or `src`/`srcApp`) |
| `PUT`    | `/<app>/files`                            | Rename (`src` → `path`)                                          |
| `DELETE` | `/<app>/files?path=<path>`                | Delete file                                                      |
| `PUT`    | `/x/uploadFile?_rmx_upload_token=<token>` | Upload via registered upload URL                                 |

**File Manager** (compatible with Mixer): `/<app>/file-manager/content` (GET/PUT/POST/DELETE), `/<app>/file-manager/props` (GET/PATCH)

### Agents & Webhooks

| Method | Endpoint | Description              |
|--------|----------|--------------------------|
| `GET   | POST`    | `/<app>/webhooks/<id>`   | Invoke pre-configured webhook (module from DB record) |
| `GET   | POST`    | `/<app>/agents/<module>` | Invoke agent by module name; params: `output`, `field`, `async`, `timeout`, `inputEscEnabled`, `outputEscEnabled` |
| `GET   | POST`    | `/<app>/lambda/<module>` | Lightweight agent (only target module loaded) |

### Auth & Token Endpoints

| Method | Endpoint                  | Description                                                               |
|--------|---------------------------|---------------------------------------------------------------------------|
| `GET`  | `/x/auth`                 | Default auth redirect                                                     |
| `GET`  | `/x/auth/<handler>`       | Named provider auth                                                       |
| `GET   | POST`                     | `/x/auth/{handler}/token`                                                 | Exchange IDP token for Remix token |
| `GET   | POST`                     | `/x/auth/{handler}/originalToken`                                         | Get stored 3rd-party token (authenticated) |
| `POST` | `/x/token`                | Refresh/create token; optional `lifetime` (seconds); anonymous if no auth |
| `POST` | `/x/token/revoke`         | Revoke token                                                              |
| `GET`  | `/x/profile`              | User profile (email, name, subscription)                                  |
| `GET`  | `/x/userID?email=<email>` | Get user ID by email (central auth server only)                           |

### Miscellaneous

| Method   | Endpoint                          | Description                                                     |
|----------|-----------------------------------|-----------------------------------------------------------------|
| `POST`   | `/<app>/link?endpoint=<endpoint>` | Create anonymous shareable link to endpoint                     |
| `GET`    | `/<app>/entities`                 | List all `entity` field values in DB                            |
| `POST`   | `/<app>/track`                    | Track messaging API (JSON)                                      |
| `WS`     | `/<app>/track.ws`                 | Track API via WebSockets                                        |
| `GET     | POST`                             | `/<app>/track/repl`                                             | Browser REPL for track debugging |
| `POST`   | `/<app>/resources`                | Upload resource (file, `name`, `public`, `bundled`, `metadata`) |
| `GET`    | `/<app>/resources`                | List resources                                                  |
| `DELETE` | `/<app>/resources/<name>`         | Delete resource                                                 |
| `GET`    | `/<app>/raw/<id>?data=<field>`    | Raw bytes of document field                                     |

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

| Method   | Endpoint                        | Description                                                                                                   |
|----------|---------------------------------|---------------------------------------------------------------------------------------------------------------|
| `POST`   | `/install-remix-file/<ws>`      | Install .remix file (body: octet-stream or JSON `{url, force}`)                                               |
| `POST`   | `/grant-permission/<ws>`        | Grant permission (`subject`, `role`, `resource`)                                                              |
| `POST`   | `/run-agent/<ws>/<app>/<agent>` | Run agent (JSON body = input params)                                                                          |
| `POST`   | `/query/<ws>/<app>`             | Register cloud query (`name`, `ast`, optional `db`, `skip`, `limit`, `include_superseded`, `include_deleted`) |
| `DELETE` | `/query/<ws>/<app>/<name>`      | Delete cloud query                                                                                            |
| `POST`   | `/count-activities/<ws>`        | Count activities; optional filters: `subject`, `activity` (`run_agent                                         |update_db|install_remix_file|create_workspace|grant_permission|remixgen`), `start`, `end` |
| `GET`    | `/subscribe/<topic>`            | Subscribe to cloud topic (SSE: `data:"<JSON>"` per message, `:` heartbeats)                                   |

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

| Method   | Endpoint           | Description                                                             |
|----------|--------------------|-------------------------------------------------------------------------|
| `POST`   | `/v1/signals`      | Create signal → `{url: "/v1/signals/<id>"}`                             |
| `GET`    | `/v1/signals/<id>` | Await signal value (blocking); returns `ok(value)` or `error("canceled" |"timeout")` as Mix tagged JSON |
| `POST`   | `/v1/signals/<id>` | Send signal value (JSON body)                                           |
| `DELETE` | `/v1/signals/<id>` | Cancel signal                                                           |

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
