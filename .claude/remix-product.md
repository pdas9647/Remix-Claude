# Remix Product — Task List, Desktop Design, Pricing

> Source: [Remix product](https://www.notion.so/27e1d464528f802291b6d5a093fbc10d)
> See also: [remix-product-scope.md](./remix-product-scope.md) (first release scope, personas, key terms), [remix-catalog.md](./remix-catalog.md) (widgets, libraries, connectors, AI agents, GTM),
> [remix-infra-auth.md](./remix-infra-auth.md) (auth, login, engineering process), [remix-infra-setup.md](./remix-infra-setup.md) (org setup, _rmx_sync, _rmx_prefs, deeplinks), [remix-snowflake.md](./remix-snowflake.md) (Snowflake integration)

---

## Global Task List (Database)

> Source: [Global Task List](https://www.notion.so/1d71d464528f80a69c47d21033bc498c)
> Data source ID: `collection://1d71d464-528f-802c-8fd3-000b607122db`

Notion database tracking all engineering/product tasks across Remix.

### Schema

| Property       | Type         | Options / Notes                                                           |
|----------------|--------------|---------------------------------------------------------------------------|
| Task           | title        | Task name                                                                 |
| Status         | status       | New/unprioritized, Not started, In progress, In review, Done              |
| Priority       | select       | high, medium, low, Never                                                  |
| Product        | multi_select | Platform, Server (Snowflake), Mobile apps, Server, Extension, Desktop     |
| Customer       | multi_select | Orderly, Snowflake Marketplace, Funda, Bomisco, Lumber                    |
| Work Stream    | multi_select | Marketing, Infrastructure, Catalog, Desktop                               |
| area           | multi_select | extension, content, platform, desktop, builder, mobile, backend, frontend |
| Assigned To    | person       | —                                                                         |
| Current sprint | checkbox     | Filters the "Current sprint" view                                         |
| Date           | date         | Optional date/range                                                       |
| Parent item    | relation     | Self-relation (parent task)                                               |
| Sub-item       | relation     | Self-relation (child tasks)                                               |
| issue          | url          | Link to external issue tracker                                            |
| notes          | text         | Free-text notes                                                           |

### Views

| View           | Description                                          |
|----------------|------------------------------------------------------|
| Task List      | All tasks, grouped by Status                         |
| Current sprint | Filtered to Desktop product + Current sprint checked |
| Platform tasks | Grouped by Status, sorted by Priority ascending      |
| Lumber tasks   | Grouped by Status, sorted by Priority ascending      |

---

## Remix Desktop Design (Old Doc)

> Source: [Remix Desktop Design - Old doc](https://www.notion.so/1f11d464528f80a9b544f59e70ad363b)

Early design decisions log (May 2025, largely superseded). Key concepts:

- **Menubar:** switch to native (OOS at the time)
- **"My artifacts":** local list of live artifacts created via Claude chat
    - Open design questions around artifact identity: no clear 1-1 between prompt sequence and artifact; need clear asset ID + save semantics; support creating multiple artifacts in same chat;
      resuming work on a previous artifact
- **"My tools":** tools management view (TBD)
- **Share dialog:** critical, needed alignment on process
- **Website project:** `remix.remixlabs.com/e/edit/remixlabs`
- **Invite flow:** follow-up item

---

## Remix Desktop Redux — Stable Concepts (Aug 2025)

> Source: [Remix Desktop Redux](https://www.notion.so/2421d464528f80f7a3baf5844687e419) — **Deprecated**, merged into Scope doc. Key concepts not already in Scope:

### Asset Specialization

When templates and AI instructions are attached to assets, it is possible to create customized versions (specialization) without using Studio. Specialized assets get pushed to the Library and used
like any other asset. Relevant personas: Contributors (specializers), Assemblers.

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

| SKU                     | Price     | Notes                                                |
|-------------------------|-----------|------------------------------------------------------|
| Community workspace     | Free      | Only Remix-sanctioned content                        |
| Widget workspace        | $100/mo   | Unlimited free users, widget collaboration only      |
| Service agent workspace | $500/mo   | $20/user, near-full platform                         |
| Enterprise              | $1,000/mo | $50/user, packs of 100, BYOI, security questionnaire |

Widgets free; everything else paid. Trial on desktop only (no free cloud workspace trial).

---

## Sub-pages Index

- [Global Task List](https://www.notion.so/1d71d464528f80a69c47d21033bc498c) — stored above
- [Scope of first release, mental model, key terms](https://www.notion.so/26f1d464528f80c68d5ff6d663522db5) — stored in remix-product-scope.md
- [Running log](https://www.notion.so/27d1d464528f8097a8c1d9fdd1e184e6) — stored in remix-infra-auth.md
- [Personas](https://www.notion.so/27e1d464528f80ae846cf0e81fcc7f59) — stored in remix-product-scope.md
- [Workspace-based auth](https://www.notion.so/27f1d464528f80228fc2e2c3145aef1f) — stored in remix-infra-auth.md
- [List of widgets](https://www.notion.so/2721d464528f8075a7abfdd7f94a2f68) — stored in remix-catalog.md
- [iOS/Android widgets for HubSpot - Research](https://www.notion.so/25e1d464528f80d4af58e6ef53aa4ddb) — stored in remix-catalog.md
- [Remix in Snowflake/Snowpark](https://www.notion.so/2841d464528f804ba6fbe2d8feb7a963) — stored in remix-snowflake.md
- [Remix Desktop Design - Old doc](https://www.notion.so/1f11d464528f80a9b544f59e70ad363b) — stored above (superseded)
- [Widget framework](https://www.notion.so/2711d464528f801aab2feef57f273d64) - blank
- [Discussion on remix_labs library](https://www.notion.so/2791d464528f8032b76cfb5e3be51176) — stored in remix-catalog.md
- [Remix Desktop Redux](https://www.notion.so/2421d464528f80f7a3baf5844687e419) — **deprecated** (merged into Scope); stable concepts stored above
- [Exported diagrams](https://www.notion.so/27e1d464528f80ea8480d07f4427f13a) — not stored, image-only (draw.io PNGs: conceptual understanding, login flow, user journeys)
- [Remix Desktop - Old log](https://www.notion.so/1d71d464528f8059abe4cf056cd0b34c) — stored in remix-infra-auth.md
- [Unified Login](https://www.notion.so/2901d464528f8096a29ccd062758e637) — stored in remix-infra-auth.md
- [Org / System Setup](https://www.notion.so/2871d464528f8079b692edaa589da6b8) — stored in remix-infra-setup.md
- [Remix Snowflake reference implementation](https://www.notion.so/2931d464528f80cc9da8ede348733125) — stored in remix-snowflake.md
- [Bug reporting](https://www.notion.so/3061d464528f80cdacf7eed2612bad07) — stored in remix-tools.md
- **Miscellaneous** ([page](https://www.notion.so/27e1d464528f80f8aedbcdbb5bad22ed)):
    - [Connectors](https://www.notion.so/1e61d464528f80508ef4c3df63b25a5f) — stored in remix-catalog.md (all sub-pages fetched)
        - [Notion](https://www.notion.so/1ea1d464528f8046bd7feb94aa1137fc), [Gmail](https://www.notion.so/1e41d464528f80bab43bc79f02116d05), [Google Sheet](https://www.notion.so/1eb1d464528f809f90e3f5d0ec35450b), [Airtable](https://www.notion.so/1eb1d464528f80d6a1dacd85a0708175), [Confluence](https://www.notion.so/1eb1d464528f80cd828bd5aea15d505d), [Slack](https://www.notion.so/1ee1d464528f80b2bdefe231f4280888), [Test page](https://www.notion.so/1f21d464528f8033bad3deb0b78547d2), [Snowflake OAuth](https://www.notion.so/20e1d464528f8057b143d055514d6cb5), [Snowflake Service Acct](https://www.notion.so/2a81d464528f80de8febcf0b590be206)
    - [Template format / rehydration process](https://www.notion.so/1fa1d464528f8073bcf9d3a96266a1c9) — not stored, tldraw diagram only, no text
    - [Bootstrapped apps](https://www.notion.so/2031d464528f80b48a64e2ca1a268c5f) — not stored, tldraw diagram only, no text
    - [Claude ↔ builder interaction](https://www.notion.so/1fa1d464528f80d3a680ce795f614e2e) — not stored, tldraw diagram only, no text
    - [Roadmap](https://www.notion.so/27e1d464528f80de876cf89114aac02e) — not stored, early skeleton (3-release cadence Oct/Nov/Dec 2025), minimal content
    - [Notes](https://www.notion.so/27e1d464528f809eb31fc46187a77cb8) — not stored
