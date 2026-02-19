# Remix Product — Concepts, Personas, Auth & Setup

> Notion: [Remix product](https://www.notion.so/27e1d464528f802291b6d5a093fbc10d)
> Key subpages: Scope/terms, Personas, Workspace-based auth, Unified Login, Org/System Setup, Bug reporting, List of widgets, Discussion on remix_labs library

---

## Personas

| Persona | Tools | Role |
|---------|-------|------|
| **IT** | Remix web admin tools | Manages Remix installation, workspaces, service agents, user mgmt, security compliance, usage monitoring. Does NOT build content. |
| **Builders** | Remix Desktop Studio, Web Studio | Creates net-new assets, publishes to shared libraries. Highly skilled in Remix platform. Usually the Remix Labs services arm. |
| **Assemblers** | Desktop Studio or enhanced live artifacts | Business domain experts who assemble apps from library assets. Not expected to be platform experts. Uses AI heavily. |
| **Contributors** | Desktop Live Artifacts, Mobile, Extension | Contribute assets back to library (e.g., configure widgets, create surveys). Employees of the business. |
| **End Users** | Web, App Clips (not Studio) | Consume apps/widgets in workflows. |

## Three Remix Clients ("Container Apps")

1. **Remix Desktop** — Tauri-based desktop app (Mac & Windows). Used by Builders/Contributors.
2. **Remix Mobile App** — Flutter-based iOS/Android app. Runs flows and iOS/Android widgets.
3. **Remix Chrome Extension** — Runs flows in browser sidebar, contextual to the current page.

All clients run flows locally on the user's edge device. Content is synced/downloaded from workspaces.

## Key Product Terminology

| Term | Definition |
|------|-----------|
| **Remix Instance** | Installation of the Remix platform for an organization |
| **Workspace** | Server-side container for service agents, DB records, ACLs, files, secrets. Has a perimeter for IT management. |
| **Central Workspace** | Primary workspace per org with auth server and central service agents |
| **Library** | Specialized workspace for storing assets, with built-in sync/management agents |
| **Widget** | A configured and published instance of a widget template (iOS/Android native or Chrome Extension) |
| **Widget Template** | A library asset with a configurator, opened in Remix Desktop |
| **Flow** | Multi-screen workflow running in a container app |
| **Service Agent** | Cloud lambda function running in a workspace with ACL. Primary auth/authz mechanism in Remix. |
| **Asset** | Building blocks: Data Tiles, Components, L1 Assets (Screens/Service Agents), L1 Groups |
| **App Clip** | Lightweight runnable app from a .remix file |
| **Remix Studio** | Sophisticated visual IDE for building flows/assets |
| **MCP** | Model Context Protocol — built into Remix Desktop for LLM tool access |
| **Super App** | A non-domain-specific layer where non-technical users assemble apps from screens as the unit of assembly — dynamic, personalized, with pluggable AI agents, SoR integration, and innovative edge UX. A domain-specific super app (e.g., for Sales & Marketing) is a specialization. |

## Auth Architecture

### Workspace-based Auth (direction)
- Moving from centralized to decentralized auth model
- Each **org** has a designated **primary workspace** on a mixer server
- That workspace contains identity provider config
- Org can have multiple workspaces; all share a userspace
- Remix tokens issued by one workspace are trusted by others in the same org
- Each workspace exposes a public key endpoint for token verification
- Cross-org access: workspaces can opt in to trusting tokens from other orgs

### Unified Login Flow
1. **Welcome Screen**: User enters org workspace ID (or pre-filled via invite link)
   - Shown as webview to: `https://remix.app/remix/signin?surface=desktop|extension|mobile|amp|web`
   - Source: `https://remix.remixlabs.com/e/edit/_rmx_signin/home`
2. **Router lookup**: Invokes `route` agent on `agt/_rmx_ws_router` → returns `{server, workspace, name, auth_remix_url}`
3. **Auth screen**: Opens org-specific auth page in browser tab (uses existing browser identities/passkeys)
   - Supports Google, Microsoft Office365, Apple
   - OAuth token exchange → Remix/amp token
4. **Token returned**: `{token, workspace, server, name, nonce, extra}` passed back to native surface via deeplink/external action

### Token Refresh
- Pass `workspace` + `registered_user` to welcome screen → forces same-user re-login

## Org / System Setup

### Global Remix Infrastructure
- **Global workspace**: server `agt`, workspace `RYxpTj1Ial`
- Key apps on global workspace:
  - `_rmx_sync` — Global groups for mobile/desktop app syncing
  - `_rmx_files` — Global file hosting
  - `_rmx_ws_router` — Central router mapping workspace IDs to org servers

### Remix Catalog
- Server `agt`, workspace `remix_labs`
- App: `_rmx_search` — Global catalog hosting

### Per-Customer/Org Setup
- Each customer gets a random workspace ID
- Workspace created and registered with `_rmx_ws_router` (currently manual via `save_route` agent)
- Key apps per org workspace:
  - `_rmx_prefs` — User preferences (catalog defaults, etc.)
  - `_rmx_search` — Org-specific catalog
  - `_rmx_files` — Org file hosting
  - `_rmx_auth` — Org-specific login page (customizable with org branding)
  - `_rmx_sync` — App syncing for org users (mobile/desktop groups)

### Sync Groups
- `surface='mobile', group='system'` — Required apps, downloaded on mobile install
- `surface='desktop', group='builtin'` — Packaged with desktop binary
- `surface='desktop', group='system'` — Downloaded on desktop install
- `surface='mobile/desktop', group='default'` — Org-specific default apps
- `surface='mobile', group='<user email>'` — Per-user app list

## Widget Ecosystem

### Current Widget Templates (Snowflake + Zendesk)
- Snowflake: List, Aggregate, Details, Chart (all use API key + AI/Anthropic)
- Zendesk: Ticket count/list, Users count, Agent leaderboard (all use OAuth + AI/Anthropic)
- All widget templates live in `remix_labs` library
- Built/edited at `remix-india.remixlabs.com`

### remix_labs Library Rules
- ALL service agents, local agents, screens, components must be in `remix_labs` library
- No dependencies on other libraries for Remix-published assets
- Use `draft` tag for assets undergoing review

## Bug Reporting Process

> Notion: [Bug reporting](https://www.notion.so/3061d464528f80cdacf7eed2612bad07)

1. **Ask: is this a bug?** — Search Slack/GitHub first, then ask in `#bugbash` or `#dumbquestionsanswered`. If known bug, add context to existing GitHub issue. Content bugs tracked on Slack (no formal process yet).
2. **Provide a repro** — Best: minimal `.remix` file or step-by-step from scratch. Also helpful: link to cloud builder screen exhibiting the issue.
   - Always include: product/surface, version, OS, browser (if relevant)
   - If no from-scratch repro: describe what you were doing + exact time (for server log correlation)
3. **Fix it or file it** — Engineer either works immediately (links PR in thread) or creates a GitHub issue. **Initial reporter is responsible** for making sure one of these happens.
4. **Note the resolution** — Mark original Slack post with reaction:
   - ✅ fixed
   - 👍 expected behavior
   - 📝 ticket created/already existed
   - 🚧 content bug, working on it here
