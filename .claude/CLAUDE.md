# Remix Labs — Project Memory

This file serves as persistent context for working on Remix Labs tasks.
When memory here is insufficient, fetch the linked Notion pages for full details.

---

## Topic Files

_Detailed notes per domain. Created as pages are reviewed._

- [Platform Architecture](platform-architecture.md) — Core components, repos, runtime stack, deployment, rcm, amp, mixer, databases, terminology
- [Remix Product](remix-product.md) — Personas, key terms, auth architecture, unified login, org/system setup, widget ecosystem, Snowflake integration, MCP connectors, bug reporting
- [Tech Debt](tech-debt.md) — Known architectural limitations: refs vs strings, no AST, incomplete type checker, wasm backend, K8s overkill, crude permissions
- [Engineering Process](engineering-process.md) — CI (CircleCI, Docker images, secrets), deployment channels (dev/beta/release), release cadence, hot fixes, auto-update chain, operational runbooks,
  QA, planning, asset submission/review (Tasks tool), GTM customer categories
- [Lumber](lumber.md) — Customer project: construction labor tracking, job costing report (PostgreSQL→Snowflake), payroll reporting, data model
- [Remix Database](remix-database.md) — Record store, universal indexing, Query Builder 2.0, history/tombstones, performance tiers
- [Remix Docs](remix-docs.md) — Public docs: glossary (L0–L3, Flow, Mix, .remix files, agents), deployment surfaces, catalogs, Mix language overview, .remix file format V2 (manifest, recordsets,
  filesets, executables), .remix syncing, deeplinks, synced preferences
- [Remix Solutions](remix-solutions.md) — Demos, case studies, platform overview videos, and example app builds (DBP, CRUD)
- [Studio Guide](studio-guide.md) — Node types, RBAC & API, state management, external actions, macros, L2 modes
- [Cloud Agent API](cloud-agent-api.md) — Mixer HTTP API: RBAC (subjects/roles/resources), agent execution, .remix install, cloud queries, files API, signals, workspace import/export, local dev

---

## Notion Page Index

_Key parent pages by domain. Fetch these from Notion when deeper context is needed._

### Platform & Engineering

- [The Remix Platform](https://www.notion.so/e29236e1181f4a8e8e25cfd607b021f1) — Internal platform docs hub
- [Remix Database](https://www.notion.so/1061d464528f80248ae6cfe395ee6a9c) — Data model and indexing
- [Queries & Syntax](https://www.notion.so/1051d464528f80dcb464fae23efefcf2) — Query Builder 2.0 and AST reference
- [Cloud Agent Server](https://www.notion.so/1061d464528f80f18e46dd27895aeda2) — Mixer overview, RBAC, API
  docs ([API](https://www.notion.so/1061d464528f8022aa1cec129096c82b), [v1 API](https://www.notion.so/1bd1d464528f80cb97cde448c9e2a944))
- [Remix Files V2](https://www.notion.so/21b1d464528f8043a176cfea87324ba5) — .remix file format spec ([Executable binaries](https://www.notion.so/1e31d464528f80c79470fad0aca57363))
- [Tech debt notes](https://www.notion.so/11f1d464528f80e09035cac4bb91dc0a) — Known architectural issues and planned changes
- [Inventory of prod assets](https://www.notion.so/13c1d464528f8080a220f64048c41ad1) — Full list of prod servers, mobile apps, web assets
- [Engineering process](https://www.notion.so/11f1d464528f80dcaf4ee38f27af640e) — CI, deployment channels, release process, QA, planning, ChrisDB runbooks

### Product

- [Remix product](https://www.notion.so/27e1d464528f802291b6d5a093fbc10d) — Product home page (subpages: Scope/terms, Personas, Workspace-based auth, Unified Login, Org/System Setup, Widgets, Bug
  reporting, Desktop Redux, Miscellaneous/Roadmap)
- [Workspace-based auth](https://www.notion.so/27f1d464528f80228fc2e2c3145aef1f) — Decentralized auth design, org model
- [Unified Login](https://www.notion.so/2901d464528f8096a29ccd062758e637) — Full login flow ([debugging](https://www.notion.so/2c41d464528f808ab1e4d162b4a22039))
- [Org / System Setup](https://www.notion.so/2871d464528f8079b692edaa589da6b8) — Global infra, per-org setup, agent permissions
- [Remix in Snowflake/Snowpark](https://www.notion.so/2841d464528f804ba6fbe2d8feb7a963) — Self-host in Snowpark Container Service, Snowflake auth workspaces
- [Connectors](https://www.notion.so/1e61d464528f80508ef4c3df63b25a5f) — MCP tool connectors (Notion, Airtable, Google Sheets, Slack, Gmail, Confluence, Snowflake)
- [List of widgets](https://www.notion.so/2721d464528f8075a7abfdd7f94a2f68) — Widget template catalog with builder URLs
- [Snowflake reference implementation](https://www.notion.so/2931d464528f80cc9da8ede348733125) — Cortex AI, Unistore research (early stage)

### Documentation

- [Remix Documentation](https://www.notion.so/fe31096a438b4f29b103c189bc9a7fb8) — Public docs hub (subpages: Glossary, Getting Started, Studio user guide, Deploying, Catalogs & Toolkits, .remix
  syncing, Deeplinks, Synced preferences)
- [OAuth Handler](https://www.notion.so/29a1d464528f807784fcdce13675f665) — Third-party OAuth sign-in handling

### Customer Projects

- [Lumber](https://www.notion.so/28c1d464528f801bac72d6c064d30db2) — Construction industry customer (subpages: Workforce Junction, Schema of Lumber Postgres, Job Costing Report, Payroll reporting)
