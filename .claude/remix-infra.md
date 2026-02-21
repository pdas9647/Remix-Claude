# Remix Infrastructure — Auth, Login, Setup, Engineering Process

> Split from [remix-product.md](./remix-product.md). Parent Notion page: [Remix product](https://www.notion.so/27e1d464528f802291b6d5a093fbc10d)

---

## Running Log — Stable Decisions

> Source: [Running log](https://www.notion.so/27d1d464528f8097a8c1d9fdd1e184e6)
> Previous logs: [Old log (Apr-Jul 2025)](https://www.notion.so/1d71d464528f8059abe4cf056cd0b34c), [Desktop Redux (Aug 2025)](https://www.notion.so/2421d464528f80f7a3baf5844687e419)

Chronological meeting notes — only stable decisions extracted here; re-fetch page for full context.

### Org & Auth Model (Oct 2025)

- A user is associated to one org (customer/account)
- An org has a primary workspace (and hence so does a user) and auth endpoint
- Auth managed at primary workspace level; tokens accepted by any endpoint
- Accessible in the builder/runtime

### Engineering Process (Feb 2026)

- **PM:** Chris = engineering PM, Reza = content PM
- **Sprint cadence:** 2 weeks — planning Monday before standup, retro following Friday
- **Bug reporting:** Slack as triage/discovery → if confirmed bug, create GitHub issue for tracking/prioritization
- **Feature requests:** flow through Global Task List board → GitHub
- **Diagrams:** standardize on draw.io (connected to Google Drive)

### Platform Maturity (Feb 2026)

- Platform is "close to feature complete" — needs bug fixes + incremental improvements, not new releases
- Content is "not at all close" — content PM and content engineering pipeline are critical needs
- Platform components: mobile, extension, desktop, server — deployment is per-customer customizable

### QA & Desktop Studio (Nov 2025)

- Moving off amp builder (remix.remixlabs.com) to desktop studio is a priority
- Service-agent-only apps should be built on desktop in dev channel
- Need beta/prod release channels for desktop app
- Missing feature: "sharable core app" — centralized libraries + build recipes on agent servers (not amp)
- QA vision: standard apps in known location → run through codegen+build pipeline for new versions → install/test across surfaces

---

## Workspace-Based Auth

> Source: [Workspace-based auth](https://www.notion.so/27f1d464528f80228fc2e2c3145aef1f)

### Design Rationale

Moving to decentralized auth because:

- Future: most Remix backend services will be self-hosted with customer's own identity provider
- Services don't need to share a userspace or mutually trust tokens
- Some deployments should exclude Remix-managed dependencies
- Centralized model requires mapping identity providers to specific workspaces anyway

### Architecture

- **Organization (org)** = basic unit of user/auth namespacing, maps to a customer. Unit of billing.
- An org has a **primary workspace** (defined by mixer service endpoint + workspace name) containing identity provider config
- Workspaces belong to exactly one org; an org can have multiple workspaces
- All workspaces in an org share a **userspace** — Remix tokens issued by one workspace are trusted by others in the same org
- Tokens are marked as *issued by* a specific workspace; that workspace exposes a **public key endpoint** for verification

### Client Sign-In Flow

1. User enters **org ID** (can be pre-filled via invite link; cached in browser/client storage after first use)
2. Client redirects to `https://auth.remixlabs.com/a/x/org-auth/{orgID}`
3. Redirect to `{org's agent server}/v1/ws/{ws}/auth/default/login`
4. Identity provider redirect dance → callback → Remix token issued
5. Final redirect to post-signin page with token in URL fragment; token claims include primary workspace base URL

### Org Onboarding

1. Create a workspace to be primary workspace
2. Register with auth server
3. Optionally define identity providers (default = existing central auth server)

### Federated Auth (Future)

- Cross-org resource access (e.g. connecting to catalog in another org)
- Workspaces can opt in to trusting tokens from other orgs
- Access control: not flat userspace but `org+user` namespace
- Not immediate priority — managed with separate sign-ins + local files for now

### Open Questions

- May need two levels of hierarchy (cf. AWS org/account, GCP org/project, Snowflake org/account)

---

## Unified Login

> Source: [Unified Login](https://www.notion.so/2901d464528f8096a29ccd062758e637)
> See
>
also: [Org / System Setup](https://www.notion.so/2871d464528f8079b692edaa589da6b8), [Unified login diagram (draw.io)](https://drive.google.com/file/d/14naLAJarPRn7n0dAvYx0I_VlQJXXqPls/view?usp=sharing)
> Debug notes: [Login debug page](https://www.notion.so/2c41d464528f808ab1e4d162b4a22039)

Full login flow implemented by all surfaces (desktop, mobile, extension, web, amp).

### Screen 1 — Welcome Screen (Workspace Selector)

Each org/customer has a **primary workspace on a mixer server**. Login = login to that workspace.

- Displayed as a webview: `https://remix.app/remix/signin?surface=<surface>`
- Source: `https://remix.remixlabs.com/e/edit/_rmx_signin/home`
- Params:
    - `surface` (string): `desktop|extension|mobile|amp|web`
    - `extra` (string, optional): passed through to stage 2 app (amp uses this for host)
    - `workspace` (string, optional): for token-expired/unlock flow
    - `registered_user` (string, optional): for token-expired/unlock flow

User enters workspace ID → invokes router agent `route` on `agt/_rmx_ws_router` → returns org info object:

| Field             | Type   | Description                                   |
|-------------------|--------|-----------------------------------------------|
| `workspace`       | string | Workspace ID the user entered                 |
| `server`          | url    | Mixer server of the org                       |
| `name`            | string | Org name (display only)                       |
| `remix_remix_url` | url    | Org-specific auth app URL                     |
| `remix_app_url`   | url    | Full remix.app URL for auth app (convenience) |

### Screen 2 — Workspace-Specific Auth

Native surface opens `remix_app_url` in a **real browser tab** (so existing identities/passkeys are available).

- Additional params: `&nonce=<string>`, `&extra=<string>` (passed back at end)
- Auth app is fully customizable with org branding (set up at onboarding)
- Source for Remix default: `https://remix.remixlabs.com/e/edit/_rmx_auth/home`
- After updating: use "Host on GCS" / Upload Package to deploy new version

User picks identity provider (e.g. Google) → OAuth token exchange → Remix/amp token issued → object passed back to native surface:

| Field       | Type   | Description              |
|-------------|--------|--------------------------|
| `token`     | string | Remix token              |
| `workspace` | string | Workspace ID             |
| `server`    | url    | Mixer server             |
| `name`      | string | Org name (display)       |
| `nonce`     | string | Optional, passed through |
| `extra`     | string | Optional, passed through |

Each surface implements its own mechanism for passing data back (deeplink or external action).

### Token Expired / Unlock Flow

Pass `workspace` + `registered_user` to the welcome screen → forces re-login as the same user. Workspace selector is hidden; shows "registered with existing user". If wrong user signs in, loop back.

### Platform Notes

- **iOS:** browser tab auto-closes after sign-in, user returns to app
- **Android:** user must tap a native toast to return (security restriction, no workaround)

### Router Service

Deployed on `agt/remix`, app `_rmx_ws_router`:

| Agent        | Input                                       | Output                                      |
|--------------|---------------------------------------------|---------------------------------------------|
| `route`      | `{workspace}`                               | `{server, workspace, name, auth_remix_url}` |
| `save_route` | `{server, workspace, name, auth_remix_url}` | (none)                                      |

### Key Resources

- `6xX25jCW6I` — Main Remix workspace
- `_rmx_signin` — Sign-in app (welcome + workspace selector): `remix.remixlabs.com/e/edit/_rmx_signin`
- `_rmx_auth` — Auth app for workspace `erzvkynof`: `remix.remixlabs.com/e/edit/_rmx_auth`
- `_rmx_ws_router` — Global router: `remix.remixlabs.com/e/edit/_rmx_ws_router`

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

- Builder app: `remix.remixlabs.com/e/edit/_rmx_prefs`

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
