# Remix Product — Task List, Desktop Design, Pricing

> Source: [Remix product](https://www.notion.so/27e1d464528f802291b6d5a093fbc10d)
> See also: [remix-product-scope.md](./remix-product-scope.md) (first release scope, personas, key terms), [remix-catalog.md](./remix-catalog.md) (widgets, libraries, connectors, AI agents, GTM),
> [remix-infra-auth.md](./remix-infra-auth.md) (auth, login, engineering process), [remix-infra-setup.md](./remix-infra-setup.md) (org setup, _rmx_sync, _rmx_prefs,
> deeplinks), [remix-snowflake.md](./remix-snowflake.md) (Snowflake integration)

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
| Next release?  | checkbox     | Marks tasks for next release                                              |
| Date           | date         | Optional date/range                                                       |
| Parent item    | relation     | Self-relation (parent task)                                               |
| Sub-item       | relation     | Self-relation (child tasks)                                               |
| issue          | url          | Link to external issue tracker                                            |
| notes          | text         | Free-text notes                                                           |

### Views

| View           | Description                                          |
|----------------|------------------------------------------------------|
| Task List      | All open/parented tasks, grouped by Priority         |
| Current sprint | Filtered to Desktop product + Current sprint checked |
| Platform tasks | Grouped by Priority, non-content area tasks          |
| Lumber tasks   | Grouped by Status, sorted by Priority ascending      |
| Everything     | All tasks, grouped by Status                         |

---

## Product Strategy for 2026

> Source: [Product strategy for 2026](https://www.notion.so/31b1d464528f8099b350ffccbd9892ec)

Platform is essentially **feature complete**; content still needs significant work. Content team generalizes early customer deliveries into repeatable patterns; platform team focuses on stability,
usability, and "content engineering" support.

**Thematic initiatives** — each adds value to an existing platform and becomes a potential channel partner:

1. **HubSpot**
2. **Zendesk**
3. **Snowflake** (relationship well advanced)
4. **(Maybe) Salesforce**

**GTM** — land early customers in these areas, build repeatable sales and content delivery processes.

**Five customer surfaces** (all delivered to at least one customer — no more "v1" milestones):

| Surface                  | Role                                                  |
|--------------------------|-------------------------------------------------------|
| Desktop Studio           | Build/assembly environment                            |
| Self-hosted agent server | Collaboration & integration (incl. Snowflake SPCS)    |
| Chrome extension         | Rapid integration of browser-accessed data (LinkedIn) |
| Web apps                 | Built on Remix web components + optional agent server |
| iOS/Android mobile apps  | Including widgets                                     |

**PM roles:** Chris = PM for engineering team (product→engineering planning); Reza = PM for content.

---

## Running Log (Key Decisions)

> Source: [Running log](https://www.notion.so/27d1d464528f8097a8c1d9fdd1e184e6)

Chronological team decisions (recent first):

- **2026-03-06:** Added product strategy doc (above)
- **2026-02-11:** Sprint cadence — 2-week sprints (planning Monday, retro following Friday). Bug reporting: Slack as triage/discovery → GitHub issue for tracking/prioritization
- **2026-02-10:** Roadmap/planning — platform near feature complete, need focused execution. Chris = engineering PM, Reza = content PM. Global Task List as coordination point
- **2025-11-12:** QA discussion — move off amp builder, desktop beta/prod release channels needed, "sharable core app" via centralized libraries/build recipes
- **2025-10-07:** User→org→workspace decisions (one org per user, primary workspace, auth at workspace level)
- **2025-09-30:** Project planning consolidation at [Remix product](https://www.notion.so/27e1d464528f802291b6d5a093fbc10d); draw.io standardized for diagrams

Previous logs: [Apr–Jul 2025](https://www.notion.so/1d71d464528f8059abe4cf056cd0b34c), [Aug 2025](https://www.notion.so/2421d464528f80f7a3baf5844687e419)

---

## Desktop Design & Redux (Deprecated, Aug 2025)

> Sources: [Old doc](https://www.notion.so/1f11d464528f80a9b544f59e70ad363b) (May 2025, superseded), [Redux](https://www.notion.so/2421d464528f80f7a3baf5844687e419) (merged into Scope doc)

Stable concepts not already in Scope:

- **Asset specialization:** Templates + AI instructions → customized versions without Studio. Pushed to Library. Personas: Contributors (specializers), Assemblers
- **Home App:** IT specifies default app for container apps (task queue, portal, etc.)
- **Edge Analytics (DuckDB + Parquet):** Log events in service agents → Parquet files on object storage → DuckDB edge queries. Limitation: only internal Remix logs today; needs generalization
- **Runtime Strategy:** Mobile=Flutter, Desktop=Tauri, Android widgets=HTML renderer, iOS widgets=SwiftUI (short-term), Windows=in progress

### Pricing / SKU Model (Aug 2025 discussion)

| SKU                     | Price     | Notes                                                |
|-------------------------|-----------|------------------------------------------------------|
| Community workspace     | Free      | Only Remix-sanctioned content                        |
| Widget workspace        | $100/mo   | Unlimited free users, widget collaboration only      |
| Service agent workspace | $500/mo   | $20/user, near-full platform                         |
| Enterprise              | $1,000/mo | $50/user, packs of 100, BYOI, security questionnaire |

Widgets free; everything else paid. Trial on desktop only (no free cloud workspace trial).

---


---

## Products and Milestones

> Sources: [Products and Milestones (original)](https://www.notion.so/32c1d464528f808085dcdf6827b2701e), [Products and Milestones (discussion)](https://www.notion.so/32c1d464528f809c9c5ed929c8709ecd)
> Related: [engineering-ci.md](./engineering-ci.md) — Release Cadence & Branch Strategy

### Core Products

Five core products currently shipping:

1. **Remix Desktop** (Studio)
2. **Remix Copilot** (Chrome extension)
3. **Remix Server** (self-host Mixer)
    - Includes: Remix Server for Snowflake SPCS
4. **Remix Web app** (deployed at remix.app, no official name)
5. **Remix Mobile** (iOS and Android container apps)

### Policy: One Global Milestone at a Time

The team tracks a single active public milestone at any point. Work flows into the current milestone; work deferred goes to the next milestone. Monday standup reserved for milestone board review.

### Milestone 1.0 (In Beta — April 2026)

Goals for v1.0:

- Platform in beta
- CI for Windows and Chrome Extension
- OPFS fixes
- Lumber documentation for self-hosting
- Snowflake Marketplace submission

### Milestone 1.1 (Planned)

Goals for v1.1:

- Desktop usability and stability improvements
- Mixer-based auth
- Stdlib documentation in builder
- Build server

### Review Cadence

- **Monday:** Weekly milestone board review (pre-standup)
- Public milestone = monthly minor version (`1.0`, `1.1`, …); patch per weekly promotion (`1.0.x`)

## Sub-pages Index

- [Global Task List](https://www.notion.so/1d71d464528f80a69c47d21033bc498c) — stored above
- [Product strategy for 2026](https://www.notion.so/31b1d464528f8099b350ffccbd9892ec) — stored above
- [Running log](https://www.notion.so/27d1d464528f8097a8c1d9fdd1e184e6) — stored above
- [Scope of first release, mental model, key terms](https://www.notion.so/26f1d464528f80c68d5ff6d663522db5) — stored in remix-product-scope.md
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
