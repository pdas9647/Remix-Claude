# Remix Infrastructure — Org Setup, Sync, Prefs & Deeplinks

> Split from [remix-product.md](./remix-product.md). Parent Notion page: [Remix product](https://www.notion.so/27e1d464528f802291b6d5a093fbc10d)

**Covers:** Org/System Setup, _rmx_sync (.remix file syncing), _rmx_prefs (synced preferences), Mobile/Desktop Deeplinks

---

## Org / System Setup

> Source: [Org / System Setup](https://www.notion.so/2871d464528f8079b692edaa589da6b8)
> Related: [_rmx_sync documentation](https://www.notion.so/25d1d464528f806b8c82e873d275aab7), [_rmx_prefs details](https://www.notion.so/2871d464528f80ae96dfeff9a738ca67)

### 1. Remix Internal (Global Infrastructure)

**Global workspace:** server `agt`, workspace `RYxpTj1Ial`

| App              | Purpose                                                                | Permissions                            |
|------------------|------------------------------------------------------------------------|----------------------------------------|
| `_rmx_sync`      | Global app groups synced to all users (system apps for mobile/desktop) | `get_apps_for_group`: all auth'd users |
| `_rmx_files`     | Global file hosting                                                    | —                                      |
| `_rmx_ws_router` | Central router (workspace → org info)                                  | `route`: anonymous users               |

**`_rmx_sync` groups (global):**

| Surface   | Group     | Type     | Description                                 |
|-----------|-----------|----------|---------------------------------------------|
| `mobile`  | `system`  | required | Downloaded on mobile app install            |
| `desktop` | `builtin` | required | Packaged with desktop binary (compile time) |
| `desktop` | `system`  | required | Downloaded on desktop app install           |

Admin setup: `remix.remixlabs.com/e/edit/_rmx_sync/setup_admin` (node 58, "system groups")

### 2. Remix Catalog

**Workspace:** server `agt`, workspace `remix_labs`

| App           | Purpose                             |
|---------------|-------------------------------------|
| `_rmx_search` | Official catalog (federated search) |

### 3. Per-Customer / Org Setup

Each new customer gets a random workspace ID. A workspace is created and registered with `_rmx_ws_router` via `save_route` agent (currently manual:
`remix.remixlabs.com/e/edit/_rmx_ws_router/save_route`).

**Required apps per org workspace:**

| App           | Purpose                                                  | Key Permissions                                                  |
|---------------|----------------------------------------------------------|------------------------------------------------------------------|
| `_rmx_prefs`  | User preferences (catalog list for builder search, etc.) | `get_prefs`, `set_prefs`: all auth'd; `set_default_prefs`: admin |
| `_rmx_search` | Org catalog (federated search)                           | All query/get agents: all auth'd                                 |
| `_rmx_files`  | Org file hosting                                         | Admin only                                                       |
| `_rmx_auth`   | Org login page + OAuth provider config                   | Update router record after changes                               |
| `_rmx_sync`   | App distribution for org users                           | See below                                                        |

**`_rmx_sync` groups (per org):**

| Surface   | Group          | Type     | Description                                                 |
|-----------|----------------|----------|-------------------------------------------------------------|
| `mobile`  | `default`      | optional | Apps synced to all org users on mobile                      |
| `desktop` | `default`      | optional | Apps synced to all org users on desktop (includes home app) |
| `mobile`  | `<user email>` | optional | User-specific apps on mobile                                |

Permissions: `get_apps`, `set_apps`, `set_user_groups`: all auth'd; `*_admin` variants: admin only.

**Important:** When setting up `_rmx_sync` for an org, set the proper workspace on action node 12 first.

### Key Resources

- Auth example (Google, userspace): `remix.remixlabs.com/e/edit/_rmx_auth`
- Sync admin setup: `remix.remixlabs.com/e/edit/_rmx_sync/setup_admin`
- User sync setup: `remix.remixlabs.com/e/edit/_rmx_sync/setup`

---

## _rmx_sync — .remix File Syncing (Detailed)

> Source: [Mobile/Desktop .remix file syncing](https://www.notion.so/25d1d464528f806b8c82e873d275aab7)

Auto-syncs .remix apps to mobile/desktop. Only apps compatible with the installed platform version are downloaded.

### Data Model

- **App** — A .remix file defined by JSON: `{dbName, appName, type, url?}`
    - `type`: `gcs/versioned` (default, semver + channel), `gcs/channel` (GCS, no versioning), `url` (exact URL)
- **Group** — A list of apps for a surface: `{name, surface, home?, apps[]}`
    - `surface`: `mobile` | `desktop` (or custom)
    - `home`: optional, format `dbName/screenName`
- **Subscriptions** — Per-user, assembled in order (last home wins):
    1. Global **system** group (core apps: `_rmx_home`, `_rmx_widgets`, etc.)
    2. **default** group (per workspace/org)
    3. Named groups (marketing, sales, etc.)
    4. **user** group (identified by email)

### Hosting & Publishing

- **Publish & List** (Tools & Settings plugin) → `gcs/versioned`: uploads .remix + version pointer JSON per semver/channel
    - Path: `rmx-static/packaged/files/<app>/<version>/<app>.remix`
    - Pointer: `rmx-static/packaged/<semver>/<channel>/<app>.json`
- **Host on GCS** (Tools & Settings plugin) → `gcs/channel`: uploads .remix only (no versioning)
    - Path: `rmx-static/<channel>/<app>.remix`
- **Channel promotion**: draft → test → public (via Content Catalog in Tools & Settings). Copies pointer files.
- **Platform version promotion**: when semver bumps (e.g. 1.43→1.44), copy all pointers. Currently manual via shell scripts in `remixlabs/flutter-runtime` repo.

### Agent API (`_rmx_sync` app)

**User agents** (any auth'd user):

| Agent                | Input                               | Output                                        | Purpose                                              |
|----------------------|-------------------------------------|-----------------------------------------------|------------------------------------------------------|
| `get_apps`           | `{channel, version, surface}`       | `{channel, version, surface, user, groups[]}` | Get all synced apps for current user (resolved URLs) |
| `set_app`            | `{surface, home, apps[]}`           | `{success}`                                   | Set apps for current user's personal group           |
| `set_user_groups`    | `{groups[]}`                        | `{success}`                                   | Set named group subscriptions                        |
| `get_apps_for_group` | `{channel, version, surface, name}` | `{...name, apps[]}`                           | Get apps for one specific group                      |

**Admin agents**: `get_apps_admin`, `set_app_admin`, `set_user_groups_admin` — same as above but can target any user/group.

### Mobile Sync Behavior

- **First start** (after auth): calls `get_apps`, downloads all core apps (blocking), sets home screen
- **Subsequent launches** (or foreground): calls `get_apps`, downloads only newer versions (via `if-modified-since` header), installs in background with progress toast

### Persistence

- Records in `_rmx_sync` db: `entity=group` (group definitions), `entity=user-groups` (user subscriptions)
- No UI for management yet — use setup screens or insert records manually

---

## _rmx_prefs — Synced Preferences

> Source: [Synced preferences](https://www.notion.so/2871d464528f80ae96dfeff9a738ca67)

Syncs user preferences across devices/surfaces via a 3-layer cascade.

### Cascade Model (each layer overrides the previous)

1. **System prefs** — Constants at compile time. Default: `search.libs` = `https://agt.remixlabs.com/ws/remix_labs`
2. **Default prefs** — Per-workspace, set by admin (dynamic). Overrides system prefs.
3. **User prefs** — Per-user (dynamic). Overrides default prefs.

### Agent API (`_rmx_prefs` app, deployed on org's primary workspace)

| Agent               | Input     | Output    | Access     | Notes                                                |
|---------------------|-----------|-----------|------------|------------------------------------------------------|
| `get_prefs`         | (none)    | `{prefs}` | All auth'd | Returns merged system + default + user prefs         |
| `set_prefs`         | `{prefs}` | `{prefs}` | All auth'd | Merge semantics; `null` deletes a pref               |
| `set_default_prefs` | `{prefs}` | `{prefs}` | Admin      | Set workspace-wide defaults; `null` resets to system |

- Builder app: `https://remix.remixlabs.com/e/edit/_rmx_prefs`

---

## Mobile/Desktop Deeplinks

> Source: [Mobile/Desktop deeplinks](https://www.notion.so/27a1d464528f803db94ac980a0bd84eb)

URL scheme: `com.remixlabs.runtime://localhost/...`

### Unified (Mobile + Desktop)

| Action                 | URL                                                     | Notes                                         |
|------------------------|---------------------------------------------------------|-----------------------------------------------|
| Download & open .remix | `/run-remix-file?url=<url>&workspace&app&screen&params` | Downloads, shows confirmation, installs, runs |
| Open installed screen  | `/open-screen?workspace&app&screen&params`              | `screen` defaults to `home`                   |

### Desktop Only

| Action        | URL                       | Notes                                                              |
|---------------|---------------------------|--------------------------------------------------------------------|
| Open artifact | `/open-artifact?url&mode` | `mode` default = `configure` (switches to `run` if no config node) |

`workspace` param only meaningful on desktop.