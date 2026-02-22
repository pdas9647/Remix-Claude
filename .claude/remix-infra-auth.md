# Remix Infrastructure — Auth, Login & Engineering Process

> Split from [remix-product.md](./remix-product.md). Parent Notion page: [Remix product](https://www.notion.so/27e1d464528f802291b6d5a093fbc10d)

**Covers:** Running Log (stable decisions), Workspace-Based Auth, Unified Login

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
> also: [Org / System Setup](https://www.notion.so/2871d464528f8079b692edaa589da6b8), [Unified login diagram (draw.io)](https://drive.google.com/file/d/14naLAJarPRn7n0dAvYx0I_VlQJXXqPls/view?usp=sharing)
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