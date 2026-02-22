# Server APIs — Legacy Platform Server API

> Source: [Legacy Platform Server API](https://www.notion.so/1061d464528f80449377e6c07e72b30b) — Studio Server > Legacy Platform Server API
> Parent: Remix Documentation > User Guides and Tutorials > Remix Studio User Guide

---

## General Notes

- Parameters accepted as: URL-encoded body > query params > multipart form (first wins)
- All app endpoints: `/<app>/*` (app name ≥ 2 chars, no leading/trailing whitespace)
- `inmem=true` query param → in-memory app (separate namespace, ephemeral)
- Encodings: `application/json` (default), `application/msgpack`, `application/msgpack+base64`
- Text output: `Accept: text/html|text/javascript|text/plain` (for non-object responses)

## Tagged Types (Special JSON Representations)

| Tag                | Format                                                    | Purpose                      |
|--------------------|-----------------------------------------------------------|------------------------------|
| `{tag}bin`         | `{"_rmx_type": "{tag}bin", "data": "<base64>"}`           | Binary data                  |
| `{tag}case:db.ref` | `{"_rmx_type": "{tag}case:db.ref", "_rmx_value": "<id>"}` | DB record reference          |
| `{tag}blob`        | `{"_rmx_type": "{tag}blob", "data": "..."}`               | Nested object → own document |
| `{tag}blob:<mime>` | Blob with MIME type annotation                            | Sets `_rmx_content_type`     |
| `{tag}undefined`   | `{"_rmx_type": "{tag}undefined}"}`                        | Mix `undefined` value        |
| `{str}...`         | Escape prefix                                             | Prevents tag interpretation  |

## Document Metadata (auto-assigned)

`_rmx_id`, `_rmx_created_by`, `_rmx_created_at`, `_rmx_last_modified_by`, `_rmx_last_modified_at`, `_rmx_rev`, `_rmx_row`, `_rmx_deleted`, `_rmx_previous` (if update)

## Document CRUD

| Method   | Endpoint                           | Notes                                                                                            |
|----------|------------------------------------|--------------------------------------------------------------------------------------------------|
| `POST`   | `/<app>/documents`                 | Create (returns ID; array input → newline-separated IDs). Supports batch temp IDs for cross-refs |
| `GET`    | `/<app>/documents/<id>`            | Read by ID                                                                                       |
| `PUT`    | `/<app>/documents/<id>`            | Replace (full overwrite)                                                                         |
| `PATCH`  | `/<app>/documents/<id>`            | Upsert (merge)                                                                                   |
| `DELETE` | `/<app>/documents/<id>`            | Soft delete (204)                                                                                |
| `DELETE` | `/<app>/documents?ids=<json_list>` | Batch delete → returns `{deleted, not_found, error}`                                             |

## Queries

| Method | Endpoint                            | Notes                                                                       |
|--------|-------------------------------------|-----------------------------------------------------------------------------|
| `GET`  | `/<app>/query?query=<query_string>` | Query with optional projections. Params: `includeDeleted`, `includeHistory` |

Full query syntax: [Query syntax](https://www.notion.so/205ca411bf684f5593c3469e39de2af8) (stored in remix-nodes-queries.md)

## App Management

| Method   | Endpoint | Notes                                      |
|----------|----------|--------------------------------------------|
| `POST`   | `/<app>` | Create app (body = metadata JSON)          |
| `GET`    | `/<app>` | Get app metadata                           |
| `PUT`    | `/<app>` | Update app metadata                        |
| `DELETE` | `/<app>` | Delete app                                 |
| `GET`    | `/`      | List all apps (user has builder access to) |

## Multi-App Actions (`GET|POST /x/apps?action=<action>`)

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

## .remix File Endpoints

| Method | Endpoint                 | Description                                                                                                                                                       |
|--------|--------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `POST` | `/x/apps/export`         | Export .remix (manifest body: `apps`, `records`, `metadata`, `setup`, `bare`, `builderAssets`, `databases`, `overrides`); also supports GET with `manifest` param |
| `POST` | `/x/apps/upload`         | Upload .remix → returns `{id}`                                                                                                                                    |
| `POST` | `/x/apps/install`        | Install uploaded .remix by `id`; optional `force` to overwrite newer                                                                                              |
| `POST` | `/x/apps/remote-export`  | Export .remix → upload to `url`                                                                                                                                   |
| `POST` | `/x/apps/remote-fetch`   | Download .remix from `url` → returns `{id}`                                                                                                                       |
| `POST` | `/x/apps/remote-install` | Download + install from `url`                                                                                                                                     |

**Install logic:** Only overwrites if `last_published_at` ≥ existing, unless `force=true`. Records always inserted/passed to `_rmx_setup`.

## Files API

| Method   | Endpoint                                  | Description                                                      |
|----------|-------------------------------------------|------------------------------------------------------------------|
| `GET`    | `/<app>/files/<path>`                     | Serve file (public: no auth; private: signed URL)                |
| `GET`    | `/<app>/files?path=<path>`                | List (if ends with `/`) or read file                             |
| `POST`   | `/<app>/files`                            | Create/copy file (multipart: `path` + `data`, or `src`/`srcApp`) |
| `PUT`    | `/<app>/files`                            | Rename (`src` → `path`)                                          |
| `DELETE` | `/<app>/files?path=<path>`                | Delete file                                                      |
| `PUT`    | `/x/uploadFile?_rmx_upload_token=<token>` | Upload via registered upload URL                                 |

**File Manager** (compatible with Mixer): `/<app>/file-manager/content` (GET/PUT/POST/DELETE), `/<app>/file-manager/props` (GET/PATCH)

## Agents & Webhooks

| Method      | Endpoint                 | Description                                                                                                       |
|-------------|--------------------------|-------------------------------------------------------------------------------------------------------------------|
| `GET\|POST` | `/<app>/webhooks/<id>`   | Invoke pre-configured webhook (module from DB record)                                                             |
| `GET\|POST` | `/<app>/agents/<module>` | Invoke agent by module name; params: `output`, `field`, `async`, `timeout`, `inputEscEnabled`, `outputEscEnabled` |
| `GET\|POST` | `/<app>/lambda/<module>` | Lightweight agent (only target module loaded)                                                                     |

## Auth & Token Endpoints

| Method      | Endpoint                          | Description                                                               |
|-------------|-----------------------------------|---------------------------------------------------------------------------|
| `GET`       | `/x/auth`                         | Default auth redirect                                                     |
| `GET`       | `/x/auth/<handler>`               | Named provider auth                                                       |
| `GET\|POST` | `/x/auth/{handler}/token`         | Exchange IDP token for Remix token                                        |
| `GET\|POST` | `/x/auth/{handler}/originalToken` | Get stored 3rd-party token (authenticated)                                |
| `POST`      | `/x/token`                        | Refresh/create token; optional `lifetime` (seconds); anonymous if no auth |
| `POST`      | `/x/token/revoke`                 | Revoke token                                                              |
| `GET`       | `/x/profile`                      | User profile (email, name, subscription)                                  |
| `GET`       | `/x/userID?email=<email>`         | Get user ID by email (central auth server only)                           |

## Miscellaneous

| Method      | Endpoint                          | Description                                                     |
|-------------|-----------------------------------|-----------------------------------------------------------------|
| `POST`      | `/<app>/link?endpoint=<endpoint>` | Create anonymous shareable link to endpoint                     |
| `GET`       | `/<app>/entities`                 | List all `entity` field values in DB                            |
| `POST`      | `/<app>/track`                    | Track messaging API (JSON)                                      |
| `WS`        | `/<app>/track.ws`                 | Track API via WebSockets                                        |
| `GET\|POST` | `/<app>/track/repl`               | Browser REPL for track debugging                                |
| `POST`      | `/<app>/resources`                | Upload resource (file, `name`, `public`, `bundled`, `metadata`) |
| `GET`       | `/<app>/resources`                | List resources                                                  |
| `DELETE`    | `/<app>/resources/<name>`         | Delete resource                                                 |
| `GET`       | `/<app>/raw/<id>?data=<field>`    | Raw bytes of document field                                     |