# Remix Labs — Project Memory

This file serves as persistent context for working on Remix Labs tasks. When memory here is insufficient, fetch the linked Notion pages for full details.

---

## Topic Files

- [glossary.md](./glossary.md) — Remix Studio terminology, navigation levels (L0-L3), module types, Mix language, agents, build/deploy concepts, nodegraph mechanics, complete node type catalog (color coding, 3 escape hatches)
- [collaboration-assembly.md](./collaboration-assembly.md) — Steel thread: 3-persona pipeline (Developer > Assembler > Consumer), config points, edge personalisation
- [federated-servers.md](./federated-servers.md) — Federated search architecture, external libraries (Amp/Workspace), asset sync (push/pull), L2SourceNode clipboard format, published Remix libraries (remix-libraries workspace: Design/HubSpot/Drafts)
- [variants.md](./variants.md) — Variants (config assets): base/config/instance model, deep sync, L2SourceNode variant format, creation lifecycle
- [l1-catalog-assets.md](./l1-catalog-assets.md) — L1-level catalog publish/search/sync, Amp vs cloud hosting constraints, JSON unpacking challenges
- [funda.md](./funda.md) — Funda customer project: member directory, workspace sEt2qxPydL, apps (DBP/Admin/Member), LinkedIn clipper, services
- [customer-projects-other.md](./customer-projects-other.md) — Orderly (hotel concierge), 5 Star Music (Twilio IVR), Bomisco (LinkedIn→HubSpot), Tapped In (student events, push messaging), all workspace IDs
- [lumber.md](./lumber.md) — Lumber: job costing (lfi_ tables), Workforce Junction (h_ tables), payroll (prl_ 38 tables, 50+ functions), Snowflake migration
- [remix-product.md](./remix-product.md) — Scope, personas, desktop concepts, pricing/SKU, runtime strategy, sub-page index
- [remix-catalog.md](./remix-catalog.md) — Widget catalog, base catalog (Table Component, Charts), HubSpot research, remix_labs library guidance, connectors (MCP tools), AI agents (Anthropic/OpenAI/Gemini), GTM patterns, T1/T2 categories
- [hubspot-toolkit.md](./hubspot-toolkit.md) — HubSpot toolkit: Connector/Configurator/Library/Example apps, asset versions, OAuth setup (required scopes), installation steps, use case scenarios
- [remix-infra.md](./remix-infra.md) — Auth (workspace-based), Unified Login, Org/System Setup, _rmx_sync (app/group/subscription model, hosting, channel promotion, agent API), _rmx_prefs (3-layer cascade), deeplinks, iOS widget debugging, bug reporting, asset submission, MCP tools
- [remix-snowflake.md](./remix-snowflake.md) — Snowflake/Snowpark spike, reference implementation (Cortex AI), Cortex Search catalog (upsert_record/cortex_search agents), Snowflake workspaces, connectors, widgets
- [duckdb.md](./duckdb.md) — DuckDB catalog: DuckLake attach + query (Parquet-backed), requires Desktop app
- [oauth.md](./oauth.md) — Third-party OAuth flows for Desktop (popup), Chrome Extension (hosted handler + publish agent), Mobile Widgets (device browser), OAuth Handler library asset
- [remix-smb.md](./remix-smb.md) — SMB catalog: 3 transaction models (ordering/booking services/booking resources), data model (business/venue/catalog/order), order lifecycle (dual status), UI assets (_rmx_drafts library), printer integration, Stripe/getaddress.io integrations
- [remix-dbp.md](./remix-dbp.md) — Digital Business Profile / About Us catalog: media (video players, galleries), text lists, review assets (Yelp/clipper), forms (lead/contact), banners (4 types), design library on remix-india-beta

---

## Notion Page Index

### Remix Product

- [Remix product](https://www.notion.so/27e1d464528f802291b6d5a093fbc10d) — Product hub (stored in remix-product.md)
- [Global Task List](https://www.notion.so/1d71d464528f80a69c47d21033bc498c) — Task tracking database, schema + views (stored in remix-product.md)
- Sub-pages: most fetched and stored across remix-product.md / remix-infra.md / remix-snowflake.md. Remaining unfetched: Widget framework, Template format, Bootstrapped apps, Claude↔builder (tldraw-only), Roadmap

### Design Zone

- [Dashboard](https://www.notion.so/9379f477c4f942b6a4f829903b0b7ba2) — Design team hub, resources, Slack: #design
- [Glossary](https://www.notion.so/3bea488f743e43f38c4b4c2b44d58000) — Remix Studio terminology (stored in glossary.md)
- [Collaboration & Assembly](https://www.notion.so/44d11988dfb24d0ea32fb8871637e72a) — Steel thread (stored in collaboration-assembly.md)
- [Big Ticket Design Backlog & Debt](https://www.notion.so/36e75d4da189492bb64cc71f1b4d7940) — Design backlog: educational content, query builder, card editor, layout assembly, nodegraph, groups
- [Federated Servers](https://www.notion.so/6496884844e74752af5b94d933afb22a) — Federated search setup & sync (stored in federated-servers.md)
- [Design Proposal - Config assets](https://www.notion.so/12a1d464528f8071ad0ce25f9f53320e) — Original design doc (complete), evolved into Variants
- [Variants](https://www.notion.so/1361d464528f801db88efc7b8a4bc063) — Variant lifecycle, deep sync, configurator integration (stored in variants.md)
- [Design Proposal - L1 catalog assets](https://www.notion.so/13f1d464528f80f4a2ecceedce5a0051) — L1 catalog operations design (stored in l1-catalog-assets.md)

### Customer Success Projects

- [Customer Success Projects](https://www.notion.so/26f1d464528f80f3a6fffc1fe4cf294e) — Hub page with all customers, workspace IDs, URLs, scope summaries
- [Funda](https://www.notion.so/26f1d464528f807f9782d5bffc5401a5) — Member directory for founder community (stored in funda.md)
- [FUNDA notes](https://www.notion.so/21f1d464528f804da24df85d82f35d9d) — Detailed screen/service breakdown (stored in funda.md)
- [Orderly](https://www.notion.so/2781d464528f80e6b282fa5ce3050372) — Hotel concierge, WebRez scraper (stored in customer-projects-other.md)
- [Lumber](https://www.notion.so/28c1d464528f801bac72d6c064d30db2) — Workforce/payroll/job costing (stored in lumber.md)
    - [Workforce Junction](https://www.notion.so/2f71d464528f80428a68dd67e47451a0) — Employee benefits MDV schema
    - [Workforce Junction - Schema for MDV](https://www.notion.so/28e1d464528f80238c9cc5eced5161fc) — Full SQL DDL
    - [Schema of Lumber Postgres](https://www.notion.so/2f71d464528f8092b0dce634ef55bfb7) — Job costing lfi_ tables + sample data
    - [Job Costing Report](https://www.notion.so/2fb1d464528f808b987cce2583b7fb8a) — Report columns, data flow, filters
    - [Payroll reporting](https://www.notion.so/3041d464528f80c4bc54c355306a4129) — prl_ schema ER diagram
    - [Payroll tables documentation](https://www.notion.so/3041d464528f8000b005f648dcf2bef6) — 38 table docs
    - [Payroll reports](https://www.notion.so/3041d464528f80ae954cc7968a6373fd) — 50+ function reference
- [Bomisco](https://www.notion.so/2bc1d464528f806f94c9ce622bddacab) — LinkedIn-to-HubSpot clipper, field mappings (stored in customer-projects-other.md)
- [5 Star Music Kanban](https://www.notion.so/26a1d464528f8095b9b6e072efec37cd) — Twilio IVR pilot task board (stored in customer-projects-other.md)
- [Twilio and Remix work steps: IVR 5 Star](https://www.notion.so/29a1d464528f8043a346debe4f572cde) — Twilio setup, call forwarding, architecture (stored in customer-projects-other.md)
- [Tapped In Interactions](https://www.notion.so/10c1d464528f804bb97fd9f061d7fbf2) — Student event platform (U of Alabama), external developer case study (stored in customer-projects-other.md)
    - [Push messaging](https://www.notion.so/19f1d464528f80a781c5cbaa4e2412d3) — Firebase browser push, platform support matrix (stored in customer-projects-other.md)

### GTM & Content Operations

- [Remix Next GTM](https://www.notion.so/2001d464528f803293aaebc066f1acb4) — Warm leads concept, reusable asset patterns, Funda GTM steps (stored in remix-product.md)
- [Tasks - Asset Submission Guide](https://www.notion.so/2111d464528f8031a05fd76e687b6379) — Artifact submission for review/catalog publish (stored in remix-infra.md)
    - [T1/T2 Customers](https://www.notion.so/20d1d464528f80608571fc691f0bf69f) — 4 customer solution categories + prospect list (stored in remix-product.md)
- [MCP Tools](https://www.notion.so/2141d464528f8070b79bfdece10ff94f) — Hub page for MCP tooling
    - [Tool Prompting](https://www.notion.so/2141d464528f80beb14be0ffecf31bf7) — Section-based prompt structure for MCP tool descriptions (stored in remix-infra.md)
    - [MCP Tools Installation Guide](https://www.notion.so/2141d464528f807486ebee2a6b261cfe) — Create service → generate schema → save with mcp_tool type → deploy (stored in remix-infra.md)

### Library & Content Assets

- [AI](https://www.notion.so/2991d464528f8043b82ff22040962cc2) — AI library agents (Anthropic, OpenAI, Gemini) setup + inventory (stored in remix-catalog.md)
- [AI (catalog page)](https://www.notion.so/2991d464528f80178a49fb6079af7441) — Same content with asset URLs, adds openai_embeddings (stored in remix-catalog.md)
- [Accessing Remix libraries](https://www.notion.so/13c1d464528f8039beb0c08aa8413722) — Published libraries (Design/HubSpot/Drafts) on remix-libraries workspace, setup steps (stored in federated-servers.md)
- [Base catalog](https://www.notion.so/1051d464528f80c3952fd7cd198da342) — Hub for published base components (stored in remix-product.md)
    - [Table Component](https://www.notion.so/f66595ea7c7146d895513e296da96275) — Editable/display table widget, cell click events (stored in remix-product.md)
    - [Charts](https://www.notion.so/d1b57c77c9014194b59b6f5a23b7f188) — WebComponent data viz using Chart.js (stored in remix-product.md)
    - [rmx-voice-chat-headless](https://www.notion.so/26c1d464528f80ed87f0cec9d0c31635) — Empty page (no content yet)
- [HubSpot toolkit](https://www.notion.so/1371d464528f809b9f51cf5b4337a137) — Toolkit hub: Connector, Configurator, Library, Example apps (stored in hubspot-toolkit.md)
    - [Example scenarios](https://www.notion.so/1371d464528f80eaa4a9e94ffb518ad5) — Draft/placeholder: planned use case categories (stored in hubspot-toolkit.md)
    - [HubSpot toolkit assets](https://www.notion.so/1371d464528f80c6b609c3589d263c8b) — Asset versions, download URLs, hosted app URLs (stored in hubspot-toolkit.md)
        - [Installation instructions](https://www.notion.so/13b1d464528f80d29831d063eaf7b365) — 3-step install flow, links to sub-steps (stored in hubspot-toolkit.md)
        - [Set up OAuth in HubSpot](https://www.notion.so/1441d464528f8005a611c39fc2f76cc3) — 18-step OAuth guide, required scopes (stored in hubspot-toolkit.md)
        - [HubSpot library](https://www.notion.so/1371d464528f80599defe5ccbc7d1e9c) — Placeholder page, no content yet
- [DuckDB](https://www.notion.so/2991d464528f80429000f327b27d8845) — DuckLake attach + query catalog items (stored in duckdb.md)
- [OAuth Handler](https://www.notion.so/29a1d464528f807784fcdce13675f665) — connect_extension screen, oauthredirect topic (stored in oauth.md)
- [Snowflake — Cortex Search](https://www.notion.so/2aa1d464528f8016ab74d92bc328935b) — Cortex Search service setup, upsert_record/cortex_search agents (stored in remix-snowflake.md)
- [Remix Desktop Third Party OAuth](https://www.notion.so/29c1d464528f80b4aba2f4414a2656ce) — Desktop popup OAuth flow (stored in oauth.md)
- [Remix Extension Third Party OAuth](https://www.notion.so/29c1d464528f80648125fe390838fe0e) — Chrome Extension OAuth flow via hosted handler (stored in oauth.md)
- [Remix Widgets Third Party OAuth](https://www.notion.so/29c1d464528f808f8b86cf7a7d860239) — Mobile widgets OAuth flow via device browser (stored in oauth.md)
- [SMB catalog](https://www.notion.so/1051d464528f80a09405d83aec33ffb9) — SMB catalog hub: transaction models, data model, assets, tools (stored in remix-smb.md)
    - [Transaction models for SMBs](https://www.notion.so/1291d464528f8077b390e89eec31115d) — Ordering, booking services, booking resources (stored in remix-smb.md)
        - [Booking services provided by staff](https://www.notion.so/1291d464528f80429975c270abe1b80c) — Service-based SMB data model (stored in remix-smb.md)
    - [SMB catalog assets](https://www.notion.so/1521d464528f8074aa60c259b498851b) — UI + cloud agent asset inventory (stored in remix-smb.md)
    - [Data model for a small to medium business](https://www.notion.so/10d1d464528f8055b80bc699da804886) — Core entities: business, venue, catalog, order (stored in remix-smb.md)
        - [Data model: Catalog](https://www.notion.so/88f26fb868cb4724ab86f0b0ccf502c6) — Category/Item/Variant/OptionGroup hierarchy (stored in remix-smb.md)
        - [Data model: Order](https://www.notion.so/5cb5cb553449403cad142247d3bb9acf) — Order lifecycle, dual status, attributes (stored in remix-smb.md)
            - [JSON for an Order](https://www.notion.so/11c1d464528f803ea04ff017e590c4c3) — Full JSON example (stored in remix-smb.md)
    - [Ordering and order management](https://www.notion.so/10b1d464528f807e9d03efef673da2cb) — Container page: ordering channels, payments, cart (stored in remix-smb.md)
        - [Printer Integration](https://www.notion.so/e0e420eb2e254a2293cba1f4a337c138) — Customer chit + kitchen chit POS components (stored in remix-smb.md)
- [Business Profile / About Us Catalog](https://www.notion.so/1051d464528f801e975cc2bf6cd715b9) — DBP media/content catalog hub (not yet fetched; sub-pages stored in remix-dbp.md)
    - [Media](https://www.notion.so/500fd31c62d5457ca4044a26782adda5) — Container for media assets (stored in remix-dbp.md)
        - [Video Players](https://www.notion.so/ec18a532129e4dd6a4f37ae9d445387d) — Remix + embedded video players, overlay variants (stored in remix-dbp.md)
        - [List of videos](https://www.notion.so/a96d7f9e95a545b080d788ff5d40d8e2) — Video list components (stored in remix-dbp.md)
        - [image galleries](https://www.notion.so/7c82e8be64384df7b6277edec1f14df7) — 3 gallery types: hero, no-hero, single-row (stored in remix-dbp.md)
        - [List of images with text](https://www.notion.so/d09fa785ff5541df9ab50b74500cc93e) — Image grid with 2-line text per image (stored in remix-dbp.md)
        - [images](https://www.notion.so/e1816e3726314bf49950c736b82af61f) — not yet fetched
    - [List of texts](https://www.notion.so/417c0e30d8044b659221751a4337088b) — 3 text list assets: points/steps, paragraph, bullet (stored in remix-dbp.md)
    - [Review assets](https://www.notion.so/7c3dabb3f1644d6c8bd6b5c749ed163f) — Yelp API + clipper review components (stored in remix-dbp.md)
    - [Forms](https://www.notion.so/2a835f37d2874066accbb2d4a02f7cbf) — Container for form assets (stored in remix-dbp.md)
        - [Lead form](https://www.notion.so/f99a82543fd74668928f01fffdd29df6) — Name/email/company/title capture + overlay (stored in remix-dbp.md)
        - [Simple Contact Form](https://www.notion.so/1d071e04ab5b49f48f28a312adb025c2) — Name/email/phone capture + overlay (stored in remix-dbp.md)
    - [Banners](https://www.notion.so/cda4a08e28874f42987417dd4f79f2e5) — Container for banner assets (stored in remix-dbp.md)
        - [Banners with back buttons](https://www.notion.so/6336070433614ec389f973ce7bb6fa3f) — Sub-page banners (stored in remix-dbp.md)
        - [Banners with logos and buttons](https://www.notion.so/969074b8bbaa45a78591bed8f6a25ec5) — Home/splash banners with CTA (stored in remix-dbp.md)
        - [Profile page banners](https://www.notion.so/bc6c2faf2fe74ee8be6f3a794e9a744f) — Profile headers, optional LinkedIn (stored in remix-dbp.md)
        - [Banners with home button](https://www.notion.so/3952d0976c014436bb17ee8a3565f1f9) — Home/splash/contact banners (stored in remix-dbp.md)

### Documentation

- [Remix Documentation (root)](https://www.notion.so/fe31096a438b4f29b103c189bc9a7fb8) — not yet fetched
- [Blog and updates](https://www.notion.so/ba9d06c6f70c4d349be54bfcdb8eaed4) — Empty hub (release notes: 18 Sep, 25 Sep 2024). Reveals doc sidebar structure.
- [Getting Started in Remix Studio](https://www.notion.so/1051d464528f8008b584dd33015fd5b3) — 9-step tutorial (all steps read); stable concepts stored in glossary.md: module anatomy, bindings, database record format, nodegraph mechanics, complete node catalog
- [Mobile/Desktop .remix file syncing](https://www.notion.so/25d1d464528f806b8c82e873d275aab7) — _rmx_sync architecture: app/group/subscription model, hosting types, channel promotion, agent API (stored in remix-infra.md)
- [Mobile/Desktop deeplinks](https://www.notion.so/27a1d464528f803db94ac980a0bd84eb) — URL scheme reference for mobile/desktop (stored in remix-infra.md)
- [Synced preferences](https://www.notion.so/2871d464528f80ae96dfeff9a738ca67) — _rmx_prefs: 3-layer cascade (system→default→user), agent API (stored in remix-infra.md)
- [iOS widget debugging/testing](https://www.notion.so/2091d464528f806bb14ffa0bb037851b) — Dynamic widget fast dev cycle (stored in remix-infra.md)
- [User Guides and Tutorials](https://www.notion.so/3e247ff418a94194a8376371117c6bb2) — not yet fetched
- [Cloud Agent Server docs](https://www.notion.so/1061d464528f80f18e46dd27895aeda2) — not yet fetched
- [Mix Language docs](https://www.notion.so/259ede6504e34505982dde5dc4b63d10) — not yet fetched
