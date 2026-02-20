# Remix Product — Scope, Personas, Desktop, Pricing

> Source: [Remix product](https://www.notion.so/27e1d464528f802291b6d5a093fbc10d)
> See also: [remix-catalog.md](./remix-catalog.md) (widgets, libraries, connectors, AI agents, GTM), [remix-infra.md](./remix-infra.md) (auth, login, setup, engineering process), [remix-snowflake.md](./remix-snowflake.md) (Snowflake integration)

---

## Global Task List (Database)

> Source: [Global Task List](https://www.notion.so/1d71d464528f80a69c47d21033bc498c)
> Data source ID: `collection://1d71d464-528f-802c-8fd3-000b607122db`

Notion database tracking all engineering/product tasks across Remix.

### Schema

| Property       | Type         | Options / Notes                                                        |
|----------------|--------------|------------------------------------------------------------------------|
| Task           | title        | Task name                                                              |
| Status         | status       | New/unprioritized, Not started, In progress, In review, Done           |
| Priority       | select       | high, medium, low, Never                                               |
| Product        | multi_select | Platform, Server (Snowflake), Mobile apps, Server, Extension, Desktop  |
| Customer       | multi_select | Orderly, Snowflake Marketplace, Funda, Bomisco, Lumber                 |
| Work Stream    | multi_select | Marketing, Infrastructure, Catalog, Desktop                            |
| area           | multi_select | extension, content, platform, desktop, builder, mobile, backend, frontend |
| Assigned To    | person       | —                                                                      |
| Current sprint | checkbox     | Filters the "Current sprint" view                                      |
| Date           | date         | Optional date/range                                                    |
| Parent item    | relation     | Self-relation (parent task)                                            |
| Sub-item       | relation     | Self-relation (child tasks)                                            |
| issue          | url          | Link to external issue tracker                                         |
| notes          | text         | Free-text notes                                                        |

### Views

| View             | Description                                           |
|------------------|-------------------------------------------------------|
| Task List        | All tasks, grouped by Status                          |
| Current sprint   | Filtered to Desktop product + Current sprint checked  |
| Platform tasks   | Grouped by Status, sorted by Priority ascending       |
| Lumber tasks     | Grouped by Status, sorted by Priority ascending       |

---

## First Release — Scope, Mental Model, Key Terms

> Source: [Scope of first release, mental model, key terms](https://www.notion.so/26f1d464528f80c68d5ff6d663522db5)

### Alignment Points

- **Foundational release** — primary purpose is to get something installable at a customer site with daily self-serve value for target personas
- No ambition for full self-serve surface area in first release — initial setup done between Remix Customer Success and customer's IT team
- GTM sales kit sells a vision (operational visibility, actionable insights, agentic workflows); some demo scenarios not supported in first release

### Three Personas (First Release)

**IT:**
- Remix = software vendor with a `Remix instance` on a `Remix server` made of 1+ `workspaces`
- One `central workspace` with `auth server` + central `service agents`
- Uses `IT admin tools` for user management, service agent management, workspace management

**Contributors:**
- Work in `Remix Desktop`, connect to central server/workspace to login
- Access `libraries` with `iOS/Android widget templates` and `Chrome Extension templates`
- `Configure` templates → `publish` widgets/flows back to the library

**End Users:**
- Install `Remix mobile app` and/or `Remix Chrome Extension`
- Login with org's Remix server + central workspace
- `Give access` to org systems (HubSpot etc.) via `OAuth`
- `Configure` widgets/flows locally on device/browser

### Key Terms (Customer-Facing)

| Term | Definition |
|------|-----------|
| Remix instance | The installation of the Remix platform for an organization |
| Widget | A configured and published instance of a widget template (iOS/Android) |
| Flow | A configured Chrome Extension flow (not called "widget" even if informational) |
| Widget template | A library asset with a configurator, openable in Remix Desktop |
| Remix clients | 3 container apps: Remix Desktop, Remix mobile app, Remix Chrome Extension. Run flows/widgets locally on user's edge device. |
| End user surfaces | iOS/Android widgets + Chrome Extension flows (mobile app flows in next release) |
| Workspace | Server-side namespace: databases, service agents, files, secrets. Central workspace runs auth server + central service agents. |
| Library | Specialized workspace for storing assets, with built-in sync/management agents |
| Widget template library | Provided by Remix CS at install, periodically updated |
| Widget library | Set of configured widgets available to the customer |
| User's chosen widgets | Subset the user has selected for their Chrome Extension (iPhone/Android selection not tracked by Remix) |
| Remix IT tools | Web app for: AI service config, service agent permissions, usage monitoring |
| Remix widget mgmt tool | Part of Remix Desktop: browse template catalog, browse/edit/delete widgets |
| Service agent | Cloud lambda function in a workspace with an ACL — primary auth/authz mechanism |
| Agentic workflow | Business flow of 2-6 steps, one or more done by an agent |
| MCP | Model Context Protocol — LLM tool/resource access, built into Remix Desktop, configurable by IT |

### MoSCoW Scope (First Release)

**Must Have:**
- Auth: Google, Office365, Apple (LinkedIn dropped)
- Remix Desktop on Mac + Windows
- Remix mobile app on App Store + Play Store
- Chrome Extension on Chrome Store
- 20-30 informational widget templates for HubSpot, Salesforce, ZenDesk, Snowflake
- Widget templates configurable in Desktop, publishable to workspace library
- Widgets configurable within mobile app (facets only) and Chrome Extension
- Widgets installable/manageable on iOS + Android
- OAuth-only auth for HubSpot/Salesforce/ZenDesk/Snowflake (no API keys/service accounts)
- Workspace-level user restriction (named emails or domain)
- Customer can onboard new users
- Description and tags on widgets

**Won't Have (first release):**
- SSO
- Safari/Edge extensions
- Mac/Windows widgets
- Customer-created/edited widget templates
- Customer deleting templates or self-updating from Remix central library
- Multiple workspaces or user-specific workspaces
- Multiple widget template libraries
- Widget publish approval workflow
- Dev/UAT/prod environments (single env = production)
- Sharing specific widget to specific user list

### Self-Serve vs Remix Customer Success

**Remix CS (initial setup):** workspace setup, service agent install + permissions, IT web app setup, client install links, widget template library customization + install, testing, first-user onboarding

**Customer IT (initial setup):** system-of-record config (e.g. HubSpot app permissions), AI service config (API key, model), service agent permissions

**Sales/Marketing Ops (ongoing):** browse templates, configure + publish widgets, manage widget library, send download instructions, share configured templates, set workspace permissions

**End Users (ongoing):** pick widgets from library in Chrome Extension or mobile, configure widgets in mobile app

---

## Personas

> Source: [Personas](https://www.notion.so/27e1d464528f80ae846cf0e81fcc7f59)

Personas are based on **tasks they do** and **tools they use**. An individual person may perform tasks across personas.

| Persona | Actual Roles | Remix Surface | What They Do (Q4 2025+) |
|---------|-------------|---------------|--------------------------|
| **IT** | Central IT, or dept admin/config personnel | Remix web app (IT tools) | **Outside Remix:** gatekeeper of SoR permissions (HubSpot etc.), mobile device mgmt, Chrome browser policies, SSO admin. **Inside Remix:** workspace setup, service agent install/config/permissions, security compliance, usage monitoring. **Never:** domain expert in end-user workflows, building libraries/assets. |
| **Builders** | IT team or dept personnel willing to learn Studio | Remix Desktop Studio + Web Studio | Net new asset creation + publication to shared library. Highly skilled in Remix platform. Collaborate with Assemblers via shared libraries. Usually don't have deep domain knowledge — provide reference implementations. Maintain quality through rigorous testing. In almost all implementations = Remix Labs services arm. |
| **Assemblers** | — | Desktop Studio or enhanced live artifact | Business domain experts. Assemble apps from library assets. Configure assets for business needs, create specialized assets. Not expected to be Remix platform experts. Contact Builder for missing assets. Combine library assets into screens + service agents, heavily AI-augmented. May create specializable templates for Contributors. |
| **Contributors** | Marketing ops, sales person (creates survey, SOCL query, etc.) | Desktop live artifacts, Mobile live artifacts, Extension variants & data tiles | Contribute assets back to library for IT/Assemblers/automated processes to include in apps. Employees of the business. Runtimes set up for them by IT with training. |
| **End Users** | — | Web, App Clips (NOT live artifacts or Studio) | Participate in workflows and move on. Apps and service agents serve their workflows. |

---

## Remix Desktop Design (Old Doc)

> Source: [Remix Desktop Design - Old doc](https://www.notion.so/1f11d464528f80a9b544f59e70ad363b)

Early design decisions log (May 2025, largely superseded). Key concepts:

- **Menubar:** switch to native (OOS at the time)
- **"My artifacts":** local list of live artifacts created via Claude chat
  - Open design questions around artifact identity: no clear 1-1 between prompt sequence and artifact; need clear asset ID + save semantics; support creating multiple artifacts in same chat; resuming work on a previous artifact
- **"My tools":** tools management view (TBD)
- **Share dialog:** critical, needed alignment on process
- **Website project:** `remix.remixlabs.com/e/edit/remixlabs`
- **Invite flow:** follow-up item

---

## Remix Desktop Redux — Stable Concepts (Aug 2025)

> Source: [Remix Desktop Redux](https://www.notion.so/2421d464528f80f7a3baf5844687e419) — **Deprecated**, merged into Scope doc. Key concepts not already in Scope:

### Asset Specialization

When templates and AI instructions are attached to assets, it is possible to create customized versions (specialization) without using Studio. Specialized assets get pushed to the Library and used like any other asset. Relevant personas: Contributors (specializers), Assemblers.

### Home App

IT specifies the default app that loads when users open any container app. Can be a task queue, a portal with widgets and bookmarks, etc. — varies by persona and implementation.

### Edge Analytics for SMB (DuckDB + Parquet)

Remix's approach to lightweight analytics without cloud compute costs:
- Log events inside service agents
- System generates Parquet files from logs, uploaded to object storage
- Apps run DuckDB queries at the edge to query Parquet files
- Current limitation: `ActivityLog` + parquet export exists for internal Remix logs only; needs generalization
- Next step: unindexed writelog mode → auto-generate Parquet when threshold reached

### Runtime Strategy (Aug 2025 state)

- Mobile: Flutter (stick with for now)
- Desktop: migrating to Tauri
- Android widgets: HTML renderer
- iOS widgets: short-term SwiftUI bugfix + constrain widget types; medium-term port Android renderer
- Windows: in progress (Fred)
- Widget upload/deployment: S3 (investigating ephemeral file expiry)

### Pricing / SKU Model (Aug 2025 discussion)

| SKU | Price | Notes |
|-----|-------|-------|
| Community workspace | Free | Only Remix-sanctioned content |
| Widget workspace | $100/mo | Unlimited free users, widget collaboration only |
| Service agent workspace | $500/mo | $20/user, near-full platform |
| Enterprise | $1,000/mo | $50/user, packs of 100, BYOI, security questionnaire |

Widgets free; everything else paid. Trial on desktop only (no free cloud workspace trial).

---

## Sub-pages Index (not yet fetched unless noted)

- [Global Task List](https://www.notion.so/1d71d464528f80a69c47d21033bc498c) — stored above
- [Scope of first release, mental model, key terms](https://www.notion.so/26f1d464528f80c68d5ff6d663522db5) — stored above
- [Running log](https://www.notion.so/27d1d464528f8097a8c1d9fdd1e184e6) — stored in remix-infra.md
- [Personas](https://www.notion.so/27e1d464528f80ae846cf0e81fcc7f59) — stored above
- [Workspace-based auth](https://www.notion.so/27f1d464528f80228fc2e2c3145aef1f) — stored in remix-infra.md
- [List of widgets](https://www.notion.so/2721d464528f8075a7abfdd7f94a2f68) — stored in remix-catalog.md
- [iOS/Android widgets for HubSpot - Research](https://www.notion.so/25e1d464528f80d4af58e6ef53aa4ddb) — stored in remix-catalog.md
- [Remix in Snowflake/Snowpark](https://www.notion.so/2841d464528f804ba6fbe2d8feb7a963) — stored in remix-snowflake.md
- [Remix Desktop Design - Old doc](https://www.notion.so/1f11d464528f80a9b544f59e70ad363b) — stored above (superseded)
- [Widget framework](https://www.notion.so/2711d464528f801aab2feef57f273d64)
- [Discussion on remix_labs library](https://www.notion.so/2791d464528f8032b76cfb5e3be51176) — stored in remix-catalog.md
- [Remix Desktop Redux](https://www.notion.so/2421d464528f80f7a3baf5844687e419) — **deprecated** (merged into Scope); stable concepts stored above
- [Exported diagrams](https://www.notion.so/27e1d464528f80ea8480d07f4427f13a) — image-only (draw.io PNGs: conceptual understanding, login flow, user journeys)
- [Remix Desktop - Old log](https://www.notion.so/1d71d464528f8059abe4cf056cd0b34c) — skipped (too large; superseded)
- [Unified Login](https://www.notion.so/2901d464528f8096a29ccd062758e637) — stored in remix-infra.md
- [Org / System Setup](https://www.notion.so/2871d464528f8079b692edaa589da6b8) — stored in remix-infra.md
- [Remix Snowflake reference implementation](https://www.notion.so/2931d464528f80cc9da8ede348733125) — stored in remix-snowflake.md
- [Bug reporting](https://www.notion.so/3061d464528f80cdacf7eed2612bad07) — stored in remix-infra.md
- **Miscellaneous** ([page](https://www.notion.so/27e1d464528f80f8aedbcdbb5bad22ed)):
    - [Connectors](https://www.notion.so/1e61d464528f80508ef4c3df63b25a5f) — stored in remix-catalog.md (all sub-pages fetched)
        - [Notion](https://www.notion.so/1ea1d464528f8046bd7feb94aa1137fc), [Gmail](https://www.notion.so/1e41d464528f80bab43bc79f02116d05), [Google Sheet](https://www.notion.so/1eb1d464528f809f90e3f5d0ec35450b), [Airtable](https://www.notion.so/1eb1d464528f80d6a1dacd85a0708175), [Confluence](https://www.notion.so/1eb1d464528f80cd828bd5aea15d505d), [Slack](https://www.notion.so/1ee1d464528f80b2bdefe231f4280888), [Test page](https://www.notion.so/1f21d464528f8033bad3deb0b78547d2), [Snowflake OAuth](https://www.notion.so/20e1d464528f8057b143d055514d6cb5), [Snowflake Service Acct](https://www.notion.so/2a81d464528f80de8febcf0b590be206)
    - [Template format / rehydration process](https://www.notion.so/1fa1d464528f8073bcf9d3a96266a1c9) — tldraw diagram only, no text
    - [Bootstrapped apps](https://www.notion.so/2031d464528f80b48a64e2ca1a268c5f) — tldraw diagram only, no text
    - [Claude ↔ builder interaction](https://www.notion.so/1fa1d464528f80d3a680ce795f614e2e) — tldraw diagram only, no text
    - [Roadmap](https://www.notion.so/27e1d464528f80de876cf89114aac02e) — early skeleton (3-release cadence Oct/Nov/Dec 2025), minimal content
    - [Notes](https://www.notion.so/27e1d464528f809eb31fc46187a77cb8) — agentic workflow brainstorm: (1) LinkedIn job listings → HubSpot clipper, (2) Gmail → HubSpot context in Chrome ext
