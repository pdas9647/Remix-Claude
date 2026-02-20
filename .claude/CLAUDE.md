# Remix Labs — Project Memory

This file serves as persistent context for working on Remix Labs tasks. When memory here is insufficient, fetch the linked Notion pages for full details.

---

## Topic Files

- [glossary.md](./glossary.md) — Remix Studio terminology, navigation levels (L0-L3), module types, Mix language, agents, build/deploy concepts, nodegraph mechanics, complete node type catalog (color
  coding, 3 escape hatches)
- [collaboration-assembly.md](./collaboration-assembly.md) — Steel thread: 3-persona pipeline (Developer > Assembler > Consumer), config points, edge personalisation
- [federated-servers.md](./federated-servers.md) — Federated search architecture, external libraries (Amp/Workspace), asset sync (push/pull), L2SourceNode clipboard format, published Remix libraries (
  remix-libraries workspace: Design/HubSpot/Drafts); Catalog and Collections (configurator app pattern, copy/insert button, component metadata); Faceted Search; Catalog V2 (workspace-based, V1 migration)
- [variants.md](./variants.md) — Variants (config assets): base/config/instance model, deep sync, L2SourceNode variant format, creation lifecycle
- [l1-catalog-assets.md](./l1-catalog-assets.md) — L1-level catalog publish/search/sync, Amp vs cloud hosting constraints, JSON unpacking challenges
- [funda.md](./funda.md) — Funda customer project: member directory, workspace sEt2qxPydL, apps (DBP/Admin/Member), LinkedIn clipper, services
- [customer-projects-other.md](./customer-projects-other.md) — Orderly (hotel concierge), 5 Star Music (Twilio IVR), Bomisco (LinkedIn→HubSpot), Tapped In (student events, push messaging), all
  workspace IDs
- [lumber.md](./lumber.md) — Lumber: job costing (lfi_ tables), Workforce Junction (h_ tables), payroll (prl_ 38 tables, 50+ functions), Snowflake migration
- [remix-product.md](./remix-product.md) — Scope, personas, desktop concepts, pricing/SKU, runtime strategy, sub-page index
- [remix-catalog.md](./remix-catalog.md) — Widget catalog, base catalog (Table Component, Charts), HubSpot research, remix_labs library guidance, connectors (MCP tools), AI agents (
  Anthropic/OpenAI/Gemini + embeddings), GTM patterns, T1/T2 categories
- [hubspot-toolkit.md](./hubspot-toolkit.md) — HubSpot toolkit: Connector/Configurator/Library/Example apps, asset versions, OAuth setup (required scopes), installation steps, use case scenarios
- [remix-infra.md](./remix-infra.md) — Auth (workspace-based), Unified Login, Org/System Setup, _rmx_sync (app/group/subscription model, hosting, channel promotion, agent API), _rmx_prefs (3-layer
  cascade), deeplinks, iOS widget debugging, bug reporting, asset submission, MCP tools, deployment methods (10 surfaces), running .remix files (remix.app/embedded), creating iOS/Android widgets,
  Remix Tools (Cloud Workspace Tool, Repository Tool)
- [remix-snowflake.md](./remix-snowflake.md) — Snowflake/Snowpark spike, reference implementation (Cortex AI), Cortex Search catalog (upsert_record/cortex_search agents), Snowflake workspaces,
  connectors, widgets
- [duckdb.md](./duckdb.md) — DuckDB catalog: DuckLake attach + query (Parquet-backed), requires Desktop app
- [oauth.md](./oauth.md) — Third-party OAuth flows for Desktop (popup), Chrome Extension (hosted handler + publish agent), Mobile Widgets (device browser), OAuth Handler library asset
- [remix-smb.md](./remix-smb.md) — SMB catalog: 3 transaction models (ordering/booking services/booking resources), data model (business/venue/catalog/order), order lifecycle (dual status), UI
  assets (_rmx_drafts library), printer integration, Stripe/getaddress.io integrations
- [remix-dbp.md](./remix-dbp.md) — Digital Business Profile / About Us catalog: media (video players, galleries, images), data visualization (quote/percentage/bar graph), buttons (simple/complex/appclip
  share/email+phone), overlay modal, text lists, review assets (Yelp/clipper), forms (lead/contact), banners (4 types), layout, basic info data tile, footers (simple/bottom nav bar), admin contact
  details (submission list/personal detail), beacon setup (tracking/fetch data), design library on remix-india-beta; Assemble About Us tutorial (workspace/agent setup, screen assembly steps)
- [platform-topics.md](./platform-topics.md) — Builder/platform mechanics: Web Components (manifest format, hosting, slots), Wasm Functions (manifest, Rust/WASI patterns, mixer wasm build),
  Working with Files from Users (Local File type, upload methods), Cloud Agent Pubsub (server→client push), Component Overriding (overridable/callable pattern), Permissions Management
  (screen-level anonymous access), Public/Private/Ephemeral Files (path-prefix routing), Storing Secrets and Tokens (secrets/tokens API, OAuth originalToken endpoint), Mixed Auth
  (fullscreen vs embedded `_rmx_auth`, fullscreen-embedded rmx-remix pattern), Macros (builder remote control: macro tiles + Mix macro module commands, node/binding/component manipulation,
  ndType/bdType/operator reference, createNodeFromAst), External Actions (runtime→host communication via onEvent map, default/mobile/local-storage/payment action catalog, moving to Client Actions)
- [remix-nodes.md](./remix-nodes.md) — Remix Studio User Guide, Nodes section: In Params/Out Params (reorder bindings, agent error out-params), Cards & Components (style units, text input
  debounce, headless components, triggered components), File Uploading (Local File Controller, accept/multiple props, directory drop), Data Objects/Values/Transforms (Join node), Queries (QB 2.0,
  DB architecture, 3 query methods, Mix API, HTTP endpoint, live query, index-capable ops, history), Query Syntax (full filter/projection/aggregation/calculation reference), Actions (File Register,
  Agent Connect tiles: Service/Local/Amp types, Api node: token/multipart/client-side, Database Save/Delete)
- [server-apis.md](./server-apis.md) — Auth integrations (OAuth/OIDC/Apple/SFDC plugin config, callback URLs, token storage, auth0 setup), Legacy Platform Server API (document CRUD, queries,
  app management, multi-app actions, .remix file endpoints, files API, agents/webhooks/lambda, resources, auth/token endpoints), Cloud Agent Server API (workspace/app management, agent execution,
  cloud queries, permissions, cloud topic subscription, .remix generation), Cloud Agent Server v1 API (workspace/app/agent introspection, permissions CRUD, files, signals, workspace import/export)

---

## Notion Page Index

### Remix Product

- [Remix product](https://www.notion.so/27e1d464528f802291b6d5a093fbc10d) — Product hub (stored in remix-product.md)
- [Global Task List](https://www.notion.so/1d71d464528f80a69c47d21033bc498c) — Task tracking database, schema + views (stored in remix-product.md)
- Sub-pages: most fetched and stored across remix-product.md / remix-infra.md / remix-snowflake.md

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
- [Accessing Remix libraries](https://www.notion.so/13c1d464528f8039beb0c08aa8413722) — Published libraries (Design/HubSpot/Drafts) on remix-libraries workspace, setup steps (stored in
  federated-servers.md)
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
        - [images](https://www.notion.so/e1816e3726314bf49950c736b82af61f) — 2 image assets: full screen + with caption (stored in remix-dbp.md)
    - [List of texts](https://www.notion.so/417c0e30d8044b659221751a4337088b) — 3 text list assets: points/steps, paragraph, bullet (stored in remix-dbp.md)
    - [Review assets](https://www.notion.so/7c3dabb3f1644d6c8bd6b5c749ed163f) — Yelp API + clipper review components (stored in remix-dbp.md)
    - [Forms](https://www.notion.so/2a835f37d2874066accbb2d4a02f7cbf) — Container for form assets (stored in remix-dbp.md)
        - [Lead form](https://www.notion.so/f99a82543fd74668928f01fffdd29df6) — Name/email/company/title capture + overlay (stored in remix-dbp.md)
        - [Simple Contact Form](https://www.notion.so/1d071e04ab5b49f48f28a312adb025c2) — Name/email/phone capture + overlay (stored in remix-dbp.md)
    - [Data visualization](https://www.notion.so/34977c23361e4959bb9761a1fc808715) — Container for data viz assets (stored in remix-dbp.md)
        - [Quote Component](https://www.notion.so/8e60cce591b0457baf0a1363290170f5) — Quote display card (stored in remix-dbp.md)
        - [Percentage Visualizer](https://www.notion.so/fd575a58f204424fadcbe26fcee81ecc) — Key stats with dynamic styling (stored in remix-dbp.md)
        - [Bar Graph](https://www.notion.so/fcd1254f1b464ecd9803b8dc8a7f593a) — Configurable horizontal bar graph (stored in remix-dbp.md)
    - [Buttons](https://www.notion.so/a9628c8c35ac4026a77cccd7bfebd3fa) — Container for button assets (stored in remix-dbp.md)
        - [Complex/multi buttons](https://www.notion.so/ad588615b7b64d439d68c54605443975) — 3 multi-element button variants (stored in remix-dbp.md)
        - [Simple buttons](https://www.notion.so/a93a998c46d64a489860af1839f716f0) — 3 single/dual button variants (stored in remix-dbp.md)
        - [Appclip Share](https://www.notion.so/c6eba32b63484d60abd0b373cfdf96da) — Share sheet for appclip URLs (stored in remix-dbp.md)
        - [Email and Phone buttons](https://www.notion.so/608e91e4cd59454286402090df06c4f7) — Contact action buttons (stored in remix-dbp.md)
    - [Overlay Modal](https://www.notion.so/1aa20ff3c50e4569b08f19c74069ea7a) — Generic modal container for overlays (stored in remix-dbp.md)
    - [Banners](https://www.notion.so/cda4a08e28874f42987417dd4f79f2e5) — Container for banner assets (stored in remix-dbp.md)
        - [Banners with back buttons](https://www.notion.so/6336070433614ec389f973ce7bb6fa3f) — Sub-page banners (stored in remix-dbp.md)
        - [Banners with logos and buttons](https://www.notion.so/969074b8bbaa45a78591bed8f6a25ec5) — Home/splash banners with CTA (stored in remix-dbp.md)
        - [Profile page banners](https://www.notion.so/bc6c2faf2fe74ee8be6f3a794e9a744f) — Profile headers, optional LinkedIn (stored in remix-dbp.md)
        - [Banners with home button](https://www.notion.so/3952d0976c014436bb17ee8a3565f1f9) — Home/splash/contact banners (stored in remix-dbp.md)
    - [Layout](https://www.notion.so/ba77f5d722d143298f3b10a28a0acdd8) — Screen layout templates for About Us clips (stored in remix-dbp.md)
    - [Image selector](https://www.notion.so/ffba147ee2a141149dc35ac5bddf05c6) — Blank page (no content yet)
    - [Basic info data tile](https://www.notion.so/0b4f7aec91804465b8228290035bfd44) — Constants data tile for About Us info (stored in remix-dbp.md)
    - [Footers](https://www.notion.so/bcb639d07b4c4f17922c11d077714f09) — Container for footer assets (stored in remix-dbp.md)
        - [Simple Footers](https://www.notion.so/3595525273514f18891ae5bab011c1d6) — 3 footer variants (stored in remix-dbp.md)
        - [Bottom navigation bar](https://www.notion.so/d047130659ba4dab8cf1a759c1b1d7db) — Screen nav bar with tab config (stored in remix-dbp.md)
    - [Admin Contact details](https://www.notion.so/d4209d1c03534a86ae037b5a0bfc2e9d) — Container for admin contact assets (stored in remix-dbp.md)
        - [Contact Submission list](https://www.notion.so/d8c09a0c58174efeb19e1db9c454ba8b) — Admin list of form submissions (stored in remix-dbp.md)
        - [Personal Detail card](https://www.notion.so/115057feedc84691ab67df99c0d9ce52) — Single submission detail view (stored in remix-dbp.md)
    - [Beacon setup](https://www.notion.so/ef2d07acb85140139ac8925087e571ad) — Container for beacon/analytics assets (stored in remix-dbp.md)
        - [Beacon](https://www.notion.so/2cd954d02a8d41f49bcbd64790199fb7) — Appclip stats tracking beacons (stored in remix-dbp.md)
        - [Fetch beacon data](https://www.notion.so/606a224c224342f0bb1c974ad4ecf806) — Admin beacon data visualization (stored in remix-dbp.md)
        - [Banners with back buttons](https://www.notion.so/6336070433614ec389f973ce7bb6fa3f) — Sub-page banners (stored in remix-dbp.md)
        - [Banners with logos and buttons](https://www.notion.so/969074b8bbaa45a78591bed8f6a25ec5) — Home/splash banners with CTA (stored in remix-dbp.md)
        - [Profile page banners](https://www.notion.so/bc6c2faf2fe74ee8be6f3a794e9a744f) — Profile headers, optional LinkedIn (stored in remix-dbp.md)
        - [Banners with home button](https://www.notion.so/3952d0976c014436bb17ee8a3565f1f9) — Home/splash/contact banners (stored in remix-dbp.md)

### Documentation

- [Remix Documentation (root)](https://www.notion.so/fe31096a438b4f29b103c189bc9a7fb8) — not yet fetched
- [Blog and updates](https://www.notion.so/ba9d06c6f70c4d349be54bfcdb8eaed4) — Empty hub (release notes: 18 Sep, 25 Sep 2024). Reveals doc sidebar structure.
- [Getting Started in Remix Studio](https://www.notion.so/1051d464528f8008b584dd33015fd5b3) — 9-step tutorial (all steps read); stable concepts stored in glossary.md: module anatomy, bindings,
  database record format, nodegraph mechanics, complete node catalog
- [Mobile/Desktop .remix file syncing](https://www.notion.so/25d1d464528f806b8c82e873d275aab7) — _rmx_sync architecture: app/group/subscription model, hosting types, channel promotion, agent API (
  stored in remix-infra.md)
- [Mobile/Desktop deeplinks](https://www.notion.so/27a1d464528f803db94ac980a0bd84eb) — URL scheme reference for mobile/desktop (stored in remix-infra.md)
- [Synced preferences](https://www.notion.so/2871d464528f80ae96dfeff9a738ca67) — _rmx_prefs: 3-layer cascade (system→default→user), agent API (stored in remix-infra.md)
- [iOS widget debugging/testing](https://www.notion.so/2091d464528f806bb14ffa0bb037851b) — Dynamic widget fast dev cycle (stored in remix-infra.md)
- [User Guides and Tutorials](https://www.notion.so/3e247ff418a94194a8376371117c6bb2) — not yet fetched
    - [Topics](https://www.notion.so/11b1d464528f801480a8d529a2297527) — Container page; all sub-pages stored across platform-topics.md, federated-servers.md, remix-dbp.md
        - [Web Components](https://www.notion.so/1141d464528f8056a818e8b84c9c10d6) — manifest.json format, Studio setup, slots, publishing/updating (stored in platform-topics.md)
        - [Wasm Functions](https://www.notion.so/1191d464528f803e8538d6d0783b58a1) — manifest format, Rust/WASI, mixer wasm build (stored in platform-topics.md)
        - [Working with files from users](https://www.notion.so/1191d464528f8032b146ed778c80c2e6) — Local File type, drag-and-drop, upload methods (stored in platform-topics.md)
        - [Cloud agent pubsub](https://www.notion.so/98f0325a059c4b31a0003fad4c364169) — Server→client push via cloud topic subscriptions (stored in platform-topics.md)
        - [Component overriding](https://www.notion.so/8d23c22ea59c4117951d9c535c8965b7) — Overridable/callable pattern, no Event bindings constraint (stored in platform-topics.md)
        - [Permissions management](https://www.notion.so/11b1d464528f80ceb963f650386dde34) — Per-screen anonymous access toggle (stored in platform-topics.md)
            - [Make individual screens available to all users](https://www.notion.so/1111d464528f80b8a505c8668f0c726f) — Anonymous access sub-page (stored in platform-topics.md)
        - [Preview .remix file](https://www.notion.so/10c1d464528f8025926ff046b12d5605) — Image-only screenshot (stored in platform-topics.md)
        - [Public, Private and Ephemeral Files](https://www.notion.so/14b1d464528f8074820dcfd0b3e6c72c) — Path-prefix privacy levels (stored in platform-topics.md)
        - [Storing secrets and tokens](https://www.notion.so/1961d464528f80d6883fffbf05c34d56) — secrets/tokens API, OAuth originalToken endpoint (stored in platform-topics.md)
        - [Catalog and Collections](https://www.notion.so/ebc636c1e547468f80944acbbf751d70) — Catalog app pattern, configurator, L2SourceNode, copy/insert button (stored in federated-servers.md)
            - [Faceted Search](https://www.notion.so/10d1d464528f80ad8d5bf181ea499394) — Side-panel filters for L2 search (stored in federated-servers.md)
            - [Example of a catalog page for an Icon Component](https://www.notion.so/f806cab33faf4872810f9fbe04c654d1) — Embed-only example (stored in federated-servers.md)
            - [Catalog V2](https://www.notion.so/1a51d464528f80db82b8d7d4555f117a) — Workspace-based catalog, V1 migration guide (stored in federated-servers.md)
        - [Assemble About Us](https://www.notion.so/b2ad31b63fe8415382958c4e14b08b8e) — DBP assembly tutorial: workspace setup, screen assembly steps (stored in remix-dbp.md)
- [Deploying Remix applications](https://www.notion.so/11b1d464528f8030a107dc7f5e8d27cc) — 10 deployment methods hub (stored in remix-infra.md)
    - [Running .remix files](https://www.notion.so/1971d464528f80e5bed2ec2a380d331f) — remix.app URLs, embedding, drag & drop (stored in remix-infra.md)
    - [Deploying a flow to the Remix mobile app](https://www.notion.so/1051d464528f80b7b98dd77ab6c388bf) — Blank page (no content yet)
    - [Creating iOS and Android widgets in Remix](https://www.notion.so/1191d464528f8025a11be59d14e12ffd) — Widget creation tutorial (stored in remix-infra.md)
- [Remix Tools](https://www.notion.so/13b1d464528f80d593a7f1de1e0d6d43) — Container for builder tools (stored in remix-infra.md)
    - [Cloud Workspace Tool](https://www.notion.so/12c1d464528f809b9f58e6a707b93fe1) — Workspace management: deploy apps, permissions, App Hub (stored in remix-infra.md)
    - [Repository Tool](https://www.notion.so/2e91d464528f800985eee93ec9fd5842) — Version control: create/list/pull/push projects (stored in remix-infra.md)
- [Remix Studio User Guide](https://www.notion.so/1051d464528f8007a5f5d40969c49427) — not yet fetched (top-level)
    - [Self-hosted Remix Server](https://www.notion.so/22a1d464528f8019be54edf3f3b1f576) — Monolithic self-hosted deployment: Docker/AWS AMI/Snowpark, OAuth config, S3-compatible files (stored in remix-infra.md)
    - [Working with State](https://www.notion.so/1431d464528f8060964ecdcbfba99fd7) — 3 state scopes (Local/Screen/App), State/Get State/Set State nodes, static vs dynamic paths (stored in remix-nodes.md)
    - [Enable anonymous access](https://www.notion.so/14c1d464528f8079b1bcd79c5fdbf4c6) — Anonymous screen setup, _rmx_auth transition, remix/user-signin Client Action (stored in platform-topics.md)
    - Studio Server
        - [Auth integrations](https://www.notion.so/11f1d464528f8070b010f945a0b0e7af) — OAuth/OIDC/Apple/SFDC plugin config, callback URLs, token storage, auth0 setup (stored in server-apis.md)
        - [Legacy Platform Server API](https://www.notion.so/1061d464528f80449377e6c07e72b30b) — Full platform server API: documents, queries, apps, files, agents, auth (stored in server-apis.md)
    - [Nodes](https://www.notion.so/13b1d464528f802c90b7c60485b36842) — container page; sub-pages stored in remix-nodes.md
        - [In Params and Out Params](https://www.notion.so/1141d464528f809fbfa6d9983ad074fe) — reorder bindings, agent error out-params (stored in remix-nodes.md)
        - [Cards & Components](https://www.notion.so/10ba63e7a90848f29d5a50f8bd4ac61d) — style units, text debounce, headless/triggered components (stored in remix-nodes.md)
            - [File Uploading](https://www.notion.so/12e1d464528f8088900dcf2f4154cc24) — Local File Controller, accept/multiple, directory drop (stored in remix-nodes.md)
        - [Data objects, values and transforms](https://www.notion.so/1051d464528f8022a535d4ab9342d1b0) — Join node; page was truncated on fetch (stored in remix-nodes.md)
        - [Queries](https://www.notion.so/1051d464528f80dcb464fae23efefcf2) — QB 2.0, DB architecture, Mix/HTTP query API, live query (stored in remix-nodes.md)
            - [Query syntax](https://www.notion.so/205ca411bf684f5593c3469e39de2af8) — full filter/projection/aggregation reference (stored in remix-nodes.md)
        - [Decisions & State Machines](https://www.notion.so/1051d464528f80a48689d2743814b1af) — truncated on fetch; stub stored in remix-nodes.md
        - [Functions & Code Modules](https://www.notion.so/8c02beb038ec40fea16762bbd8aabfe8) — truncated on fetch; stub stored in remix-nodes.md
        - [Actions](https://www.notion.so/1051d464528f80c9b498daa44df9b685) — File Register action; main content truncated (stored in remix-nodes.md)
            - [Agent Connect tiles](https://www.notion.so/12f1d464528f80a4a372e439a6d074f4) — Service/Local/Amp agent types, Refresh, Client Side toggle (stored in remix-nodes.md)
            - [Api](https://www.notion.so/1571d464528f80e9a9fee7fbbf80a0c5) — HTTP action node: token, multipart, form-urlencoded, client-side upload (stored in remix-nodes.md)
            - [Database Actions: Save and Delete](https://www.notion.so/1151d464528f8047b20fd8809e2bb766) — Delete requires DB location + ref; Create/Update placeholder (stored in remix-nodes.md)
            - [Annotations](https://www.notion.so/1051d464528f8018a17fd69389243631) — blank page (stored in remix-nodes.md)
    - [Mixed Auth](https://www.notion.so/1741d464528f808cbb5ff371d571c437) — fullscreen vs embedded `_rmx_auth`, fullscreen-embedded rmx-remix host page pattern (stored in platform-topics.md)
        - [Mixed auth test](https://www.notion.so/1b91d464528f802ebf9efe3417f1ea6f) — embedded test previews (Dev/Beta/Prod) only; no stable content
    - [Macros](https://www.notion.so/2161d464528f801f8ddaf479d816d99f) — builder remote control: macro tiles (Node Search, Screen Search, Make Agent, ArtifactSaveAs/Save, Get Module Signature), Mix macro module commands, ndType/bdType/operator reference, createNodeFromAst (stored in platform-topics.md)
    - [External Actions](https://www.notion.so/2161d464528f8014ac9ce7eda3a134ab) — runtime→host communication via onEvent map; default/mobile/local-storage/payment action catalog; moving to Client Actions (stored in platform-topics.md)
- [Cloud Agent Server docs](https://www.notion.so/1061d464528f80f18e46dd27895aeda2) — not yet fetched (top-level)
    - [Cloud Agent Server API](https://www.notion.so/1061d464528f8022aa1cec129096c82b) — Mixer API: workspace/app management, agent execution, cloud queries, .remix generation (stored in server-apis.md)
    - [Cloud Agent Server v1 API](https://www.notion.so/1bd1d464528f80cb97cde448c9e2a944) — v1 API: introspection, permissions CRUD, files, signals, workspace import/export (stored in server-apis.md)
- [Mix Language docs](https://www.notion.so/259ede6504e34505982dde5dc4b63d10) — not yet fetched
