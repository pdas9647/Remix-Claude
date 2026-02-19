# Remix Product — Concepts, Personas, Auth & Setup

> Notion: [Remix product](https://www.notion.so/27e1d464528f802291b6d5a093fbc10d)
> Key subpages: Scope/terms, Personas, Workspace-based auth, Unified Login, Org/System Setup, Bug reporting, List of widgets, Discussion on remix_labs library
> Related: [Scope of first release, mental model, key terms](https://www.notion.so/26f1d464528f80c68d5ff6d663522db5) (merged from
> deprecated [Desktop Redux](https://www.notion.so/2421d464528f80f7a3baf5844687e419))

---

## Personas

| Persona          | Tools                                     | Role                                                                                                                              |
|------------------|-------------------------------------------|-----------------------------------------------------------------------------------------------------------------------------------|
| **IT**           | Remix web admin tools                     | Manages Remix installation, workspaces, service agents, user mgmt, security compliance, usage monitoring. Does NOT build content. |
| **Builders**     | Remix Desktop Studio, Web Studio          | Creates net-new assets, publishes to shared libraries. Highly skilled in Remix platform. Usually the Remix Labs services arm.     |
| **Assemblers**   | Desktop Studio or enhanced live artifacts | Business domain experts who assemble apps from library assets. Not expected to be platform experts. Uses AI heavily.              |
| **Contributors** | Desktop Live Artifacts, Mobile, Extension | Contribute assets back to library (e.g., configure widgets, create surveys). Employees of the business.                           |
| **End Users**    | Web, App Clips (not Studio)               | Consume apps/widgets in workflows.                                                                                                |

## Three Remix Clients ("Container Apps")

1. **Remix Desktop** — Tauri-based desktop app (Mac & Windows). Used by Builders/Contributors.
2. **Remix Mobile App** — Flutter-based iOS/Android app. Runs flows and iOS/Android widgets.
3. **Remix Chrome Extension** — Runs flows in browser sidebar, contextual to the current page.

All clients run flows locally on the user's edge device. Content is synced/downloaded from workspaces.

## Key Product Terminology

| Term                     | Definition                                                                                                                                                                                                                                                                          |
|--------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| **Remix Instance**       | Installation of the Remix platform for an organization                                                                                                                                                                                                                              |
| **Workspace**            | Server-side container for service agents, DB records, ACLs, files, secrets. Has a perimeter for IT management.                                                                                                                                                                      |
| **Central Workspace**    | Primary workspace per org with auth server and central service agents                                                                                                                                                                                                               |
| **Library**              | Specialized workspace for storing assets, with built-in sync/management agents                                                                                                                                                                                                      |
| **Widget**               | A configured and published instance of a widget template (iOS/Android native or Chrome Extension)                                                                                                                                                                                   |
| **Widget Template**      | A library asset with a configurator, opened in Remix Desktop                                                                                                                                                                                                                        |
| **Flow**                 | Multi-screen workflow running in a container app                                                                                                                                                                                                                                    |
| **Service Agent**        | Cloud lambda function running in a workspace with ACL. Primary auth/authz mechanism in Remix.                                                                                                                                                                                       |
| **Asset**                | Building blocks: Data Tiles, Components, L1 Assets (Screens/Service Agents), L1 Groups. Can have templates and AI instructions attached for specialization.                                                                                                                         |
| **Asset Specialization** | Attaching templates + AI instructions to assets allows creating customized versions without Studio. Specialized assets are pushed to the Library and used like any other asset.                                                                                                     |
| **Agentic Workflow**     | A business flow requiring a few steps (2–6), one or more of which can be done by an agent                                                                                                                                                                                           |
| **Home Flow**            | Default flow that loads when user opens a container app. Specified by IT per persona (e.g., Task queue for End Users, portal with widgets for others)                                                                                                                               |
| **App Clip**             | Lightweight runnable app from a .remix file                                                                                                                                                                                                                                         |
| **Remix Studio**         | Sophisticated visual IDE for building flows/assets. Access configurable by IT per persona.                                                                                                                                                                                          |
| **MCP**                  | Model Context Protocol — built into Remix Desktop for LLM tool access. Access configurable by IT.                                                                                                                                                                                   |
| **Super App**            | A non-domain-specific layer where non-technical users assemble apps from screens as the unit of assembly — dynamic, personalized, with pluggable AI agents, SoR integration, and innovative edge UX. A domain-specific super app (e.g., for Sales & Marketing) is a specialization. |

## Auth Architecture

### Workspace-based Auth (direction)

> Notion: [Workspace-based auth](https://www.notion.so/27f1d464528f80228fc2e2c3145aef1f)

**Rationale**: Future is self-hosted services using customer's own identity provider. Centralized auth becomes an impediment when mapping many team-specific identity providers to specific workspaces.
Decentralized model removes Remix-managed dependencies from customer deployments.

**Core model:**

- **Org** = customer = unit of billing. A workspace belongs to exactly one org.
- Each org has a designated **primary workspace** on a mixer server
- That workspace contains identity provider config; workspace admins can manage it themselves
- Org can have multiple workspaces; all share a userspace
- Remix tokens are marked as *issued by* a particular workspace
- Tokens issued by one workspace are trusted by others in the same org
- Each workspace exposes a public key endpoint for token verification

**Auth endpoints** (per workspace): `/v1/ws/{ws}/auth/{auth}/login`, `.../callback`, `.../logout`

**Workspace-based sign-in flow:**

1. User enters org ID (pre-filled via invite link or client storage)
2. Client directs browser to `https://auth.remixlabs.com/a/x/org-auth/{orgID}`
3. Redirect to `{org's agent server}/v1/ws/{ws}/auth/default/login` → identity provider → callback
4. Remix token issued; token claims include primary workspace base URL

**Org on-boarding:** a) create workspace as primary, b) register with auth server, c) optionally define identity providers (default: existing central auth)

**Cross-org (federated) access:** Workspaces can opt in to trusting tokens from other orgs. Access control uses org+user space instead of flat userspace. Not immediate priority — most cross-org flows
managed with separate sign-ins and local files.

### Unified Login Flow

> Notion: [Unified Login](https://www.notion.so/2901d464528f8096a29ccd062758e637)

1. **Welcome Screen** (`_rmx_signin` app): User enters org workspace ID (or pre-filled via invite link)
    - Shown as webview to: `https://remix.app/remix/signin?surface=desktop|extension|mobile|amp|web`
    - Source: `https://remix.remixlabs.com/e/edit/_rmx_signin/home`
    - Params: `surface` (required), `extra` (optional, passed through), `workspace` (optional, for refresh), `registered_user` (optional, for refresh)
2. **Router lookup**: Invokes `route` agent on `agt/_rmx_ws_router` → returns `{server, workspace, name, auth_remix_url, remix_app_url}`
3. **Auth screen** (`_rmx_auth` app): Opens in a **real browser tab** (not webview) so existing identities and passkeys are available
    - `_rmx_auth` is fully customizable with org branding (set up at onboarding)
    - Supports Google, Microsoft Office365, Apple
    - Optional `&nonce=<string>` and `&extra=<string>` params (passed back to surface after sign-in)
    - OAuth token exchange → Remix/amp token
    - Each surface implements its own mechanism for passing data back (deeplink or external action)
4. **Token returned**: `{token, workspace, server, name, nonce, extra}` passed back to native surface
    - iOS: browser tab auto-closes, user returns to app automatically
    - Android: user must click a native toast to return (platform security limitation)

### Router Agents (`_rmx_ws_router`)

Deployed on `agt/remix` (workspace `_rmx_ws_router`):

- `route` — input: `{workspace}`, output: `{server, workspace, name, auth_remix_url}`
- `save_route` — input: `{server, workspace, name, auth_remix_url}`, output: none

### Token Expired / Unlock Flow

- Pass `workspace` + `registered_user` to welcome screen → forces same-user re-login
- Workspace selector is hidden; screen shows the app is registered with an existing user
- If user signs in with a different identity, they are returned to the same screen (login fails)
- This flow is completely optional

### Mixed Auth (automatic login prompt)

> Notion: [Unified Login - debugging](https://www.notion.so/2c41d464528f808ab1e4d162b4a22039)

- When a page doesn't permit anonymous access and user has no token, the VM redirects to `_rmx_auth`
- Every app has a default `_rmx_auth` page compiled in by the builder (source: `remix.remixlabs.com/e/edit/_rmx_auth_default/_rmx_auth`)
- The default `_rmx_auth` reads the current workspace from env and loads the signin screen via agent call
- Builder triggers login via `remix/sign-in` client action
- Per-org `_rmx_auth` .remix file hosted at: `agt.files.remix.app/{workspaceId}/_rmx_files/remix/_rmx_auth.remix`
- OAuth login pages must be opened with `<a>` tags so the window is closable after auth completes
- Token callback: `/_rmx_/_rmx_auth?_rmx_upgrade_token=true&_rmx_workspace=&_rmx_server=#token=...`

### Key Auth Resources

- `6xX25jCW6I` — main Remix workspace ID
- `_rmx_signin` — signin app (welcome screen + workspace selector): `remix.remixlabs.com/e/edit/_rmx_signin`
- `_rmx_auth` — auth app (per-org, customizable): `remix.remixlabs.com/e/edit/_rmx_auth`
- `_rmx_ws_router` — global router with `route` agent
- Login flow diagram: [Google Drive](https://drive.google.com/file/d/14naLAJarPRn7n0dAvYx0I_VlQJXXqPls/view)

## Org / System Setup

> Notion: [Org / System Setup](https://www.notion.so/2871d464528f8079b692edaa589da6b8)

### Global Remix Infrastructure

- **Global workspace**: server `agt`, workspace `RYxpTj1Ial`
- Key apps on global workspace:
    - `_rmx_sync` — Global groups for mobile/desktop app syncing. Permissions: `get_apps_for_group` → all auth'ed users
    - `_rmx_files` — Global file hosting ([builder](https://remix.remixlabs.com/e/edit/_rmx_files))
    - `_rmx_ws_router` — Central router mapping workspace IDs to org servers. Permissions: `route` → anon users ([builder](https://remix.remixlabs.com/e/edit/_rmx_ws_router))
- Admin setup for system groups: `remix.remixlabs.com/e/edit/_rmx_sync/setup_admin` (node 58, group 'system groups')

### Remix Catalog

- Server `agt`, workspace `remix_labs`
- App: `_rmx_search` — Global catalog hosting ([builder](https://remix.remixlabs.com/e/edit/_rmx_search))

### Per-Customer/Org Setup

- Each customer gets a random workspace ID
- Workspace created and registered with `_rmx_ws_router` (currently manual via `save_route` agent at `remix.remixlabs.com/e/edit/_rmx_ws_router/save_route`)
- Key apps per org workspace:
    - `_rmx_prefs` — User preferences (catalog defaults, etc.). Permissions: `get_prefs`/`set_prefs` → all auth'ed, `set_default_prefs` → admin only
    - `_rmx_search` — Org-specific catalog. Permissions: all query/get agents → all auth'ed
    - `_rmx_files` — Org file hosting. Permissions: admin only
    - `_rmx_auth` — Org-specific login page (customizable with org branding). **If updated, must also update the router record.**
    - `_rmx_sync` — App syncing for org users. Permissions: `get_apps`/`set_apps`/`set_user_groups` → all auth'ed, `*_admin` variants → admin only

### Sync Groups

- `surface='mobile', group='system'` — Required apps, downloaded on mobile install
- `surface='desktop', group='builtin'` — Packaged with desktop binary
- `surface='desktop', group='system'` — Downloaded on desktop install
- `surface='mobile/desktop', group='default'` — Org-specific default apps (also sets home app for org)
- `surface='mobile', group='<user email>'` — Per-user app list

Setup for sync groups: `remix.remixlabs.com/e/edit/_rmx_sync/setup_admin` — **important: set the proper workspace on action node 12 first**.

## Widget Ecosystem

### Widget Lifecycle

1. **Widget Template Library** — provided by Remix Customer Success, periodically updated. Templates live in `remix_labs` library.
2. **Widget Library** — configured widgets published by Contributors to a workspace library, available to all users in that workspace.
3. **User's Chosen Widgets** — subset the user has selected to show on their device/extension.

### Widget Surfaces

- **iOS/Android** — native home screen widgets (Flutter runtime on mobile)
- **Chrome Extension** — widgets rendered as flows in the browser sidebar
- **macOS/Windows** — inside the Remix Desktop app (Tauri runtime)

### Widget Auth Model

- External system auth uses **OAuth only** with user credentials (HubSpot, Salesforce, ZenDesk, Snowflake)
- No API keys or service accounts for end-user widgets (can be added per customer request)

### Current Widget Templates (Snowflake, Zendesk, HubSpot, Salesforce)

- Snowflake: List, Aggregate, Details, Chart (all use API key + AI/Anthropic)
- Zendesk: Ticket count/list, Users count, Agent leaderboard (all use OAuth + AI/Anthropic)
- HubSpot: informational widgets (OAuth + AI/Anthropic)
- Salesforce: informational widgets (OAuth + AI/Anthropic)
- All widget templates live in `remix_labs` library
- Built/edited at `remix-india.remixlabs.com`

### remix_labs Library Rules

- ALL service agents, local agents, screens, components must be in `remix_labs` library
- No dependencies on other libraries for Remix-published assets
- Use `draft` tag for assets undergoing review

## Remix IT Tools

- Web app for customer IT team
- Capabilities: configure Remix AI services (API key, model version), manage service agent permissions, monitor usage (users, widget templates, widgets, service agent calls including AI)

## Remix Widget Management Tool

- Part of Remix Desktop
- Browse widget template catalog
- Browse, edit, and delete published widgets

## Customer Setup Process

### Initial Setup (Remix Customer Success)

1. Set up customer workspace (Studio tooling)
2. Install initial service agents: AI agents, user mgmt, widget template library, widget mgmt
3. Set up permissions on installed service agents
4. Set up IT self-serve web app and share with IT team
5. Prepare install links for Desktop, Chrome Extension, Mobile app
6. Customize and install widget template library from Remix central library
7. Test and send download/login links to first users

### Initial Setup (Customer IT)

1. Configure systems of record (e.g., HubSpot app permissions)
2. Use Remix IT tools to configure AI services
3. Set up permissions on installed service agents

### Ongoing Self-Serve (Contributors / End Users)

- Contributors: browse template library, configure + publish widgets, manage widget library
- End Users: pick widgets from library (Chrome Extension or mobile), configure widgets on device

## Priority Principle

> Content → Product → Platform. The platform only exists to serve the product. The product only exists to run the content. Content priorities override product priorities. Both override platform
> priorities.

## Bug Reporting Process

> Notion: [Bug reporting](https://www.notion.so/3061d464528f80cdacf7eed2612bad07)

1. **Ask: is this a bug?** — Search Slack/GitHub first, then ask in `#bugbash` or `#dumbquestionsanswered`. If known bug, add context to existing GitHub issue. Content bugs tracked on Slack (no formal
   process yet).
2. **Provide a repro** — Best: minimal `.remix` file or step-by-step from scratch. Also helpful: link to cloud builder screen exhibiting the issue.
    - Always include: product/surface, version, OS, browser (if relevant)
    - If no from-scratch repro: describe what you were doing + exact time (for server log correlation)
3. **Fix it or file it** — Engineer either works immediately (links PR in thread) or creates a GitHub issue. **Initial reporter is responsible** for making sure one of these happens.
4. **Note the resolution** — Mark original Slack post with reaction:
    - ✅ fixed
    - 👍 expected behavior
    - 📝 ticket created/already existed
    - 🚧 content bug, working on it here
