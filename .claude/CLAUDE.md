# Remix Labs — Project Memory

This file serves as persistent context for working on Remix Labs tasks. When memory here is insufficient, fetch the linked Notion pages for full details.
Full Notion sub-page hierarchy: [NOTION-INDEX.md](./NOTION-INDEX.md)

---

## Topic Files

- [glossary.md](./glossary.md) — Remix Studio terminology, navigation levels (L0-L3), module types, Mix language, agents, build/deploy concepts, nodegraph mechanics, complete node type catalog (color
  coding, 3 escape hatches)
- [collaboration-assembly.md](./collaboration-assembly.md) — Steel thread: 3-persona pipeline (Developer > Assembler > Consumer), config points, edge personalisation
- [federated-servers.md](./federated-servers.md) — Federated search architecture, external libraries (Amp/Workspace), asset sync (push/pull), L2SourceNode clipboard format, published Remix libraries (
  remix-libraries workspace: Design/HubSpot/Drafts); Catalog and Collections (configurator app pattern, copy/insert button, component metadata); Faceted Search; Catalog V2 (workspace-based, V1
  migration)
- [variants.md](./variants.md) — Variants (config assets): base/config/instance model, deep sync, L2SourceNode variant format, creation lifecycle
- [l1-catalog-assets.md](./l1-catalog-assets.md) — L1-level catalog publish/search/sync, Amp vs cloud hosting constraints, JSON unpacking challenges
- [funda.md](./funda.md) — Funda customer project: member directory, workspace sEt2qxPydL, apps (DBP/Admin/Member), LinkedIn clipper, services
- [customer-projects-other.md](./customer-projects-other.md) — Orderly (hotel concierge), 5 Star Music (Twilio IVR), Bomisco (LinkedIn→HubSpot), Tapped In (student events, push messaging), all
  workspace IDs
- [lumber.md](./lumber.md) — Lumber: job costing (lfi_ tables), Workforce Junction (h_ tables), payroll (prl_ 38 tables, 50+ functions), Snowflake migration
- [remix-product.md](./remix-product.md) — Global Task List (schema/views), desktop design concepts, pricing/SKU, runtime strategy, edge analytics, sub-page index
- [remix-product-scope.md](./remix-product-scope.md) — First release scope (alignment, key terms, MoSCoW), personas (IT/Builders/Assemblers/Contributors/End Users), self-serve vs CS responsibilities
- [remix-catalog.md](./remix-catalog.md) — Widget catalog, base catalog (Table Component, Charts), HubSpot research, remix_labs library guidance, connectors (MCP tools), AI agents (
  Anthropic/OpenAI/Gemini + embeddings), GTM patterns, T1/T2 categories
- [hubspot-toolkit.md](./hubspot-toolkit.md) — HubSpot toolkit: Connector/Configurator/Library/Example apps, asset versions, OAuth setup (required scopes), installation steps, use case scenarios
- [remix-infra.md](./remix-infra.md) — Auth (workspace-based), Unified Login, Org/System Setup, _rmx_sync (app/group/subscription model, hosting, channel promotion, agent API), _rmx_prefs (3-layer
  cascade), deeplinks
- [remix-deploy.md](./remix-deploy.md) — Deployment methods (10 surfaces), self-hosted Remix server (Docker/AWS AMI/Snowpark), running .remix files (remix.app/embedded), creating iOS/Android widgets,
  iOS widget debugging, iOS Widget & Shortcut Agents (widget/shortcut agent in/out param spec, _rmx_widgets db, timeline, clickable zones)
- [remix-tools.md](./remix-tools.md) — Bug reporting, asset submission, MCP tools (installation + prompting best practices), Remix Tools (Cloud Workspace Tool, Repository Tool)
- [remix-snowflake.md](./remix-snowflake.md) — Snowflake/Snowpark spike, reference implementation (Cortex AI), Cortex Search catalog (upsert_record/cortex_search agents), Snowflake workspaces,
  connectors, widgets
- [duckdb.md](./duckdb.md) — DuckDB catalog: DuckLake attach + query (Parquet-backed), requires Desktop app
- [oauth.md](./oauth.md) — Third-party OAuth flows for Desktop (popup), Chrome Extension (hosted handler + publish agent), Mobile Widgets (device browser), OAuth Handler library asset
- [remix-smb.md](./remix-smb.md) — SMB catalog: 3 transaction models (ordering/booking services/booking resources), data model (business/venue/catalog/order), order lifecycle (dual status), UI
  assets (_rmx_drafts library), printer integration, Stripe/getaddress.io integrations
- [remix-dbp-media.md](./remix-dbp-media.md) — DBP media & data visualization: video players (Remix/embedded, overlay variants), video lists, image galleries (hero/no-hero/single-row),
  images (full screen/caption), data viz (quote component, percentage visualizer, bar graph); design library on remix-india-beta
- [remix-dbp-components.md](./remix-dbp-components.md) — DBP interactive components: buttons (simple/complex/appclip share/email+phone), overlay modal, text lists, review assets (Yelp/clipper),
  forms (lead/contact), banners (4 types: back/logo+button/profile/home)
- [remix-dbp-assembly.md](./remix-dbp-assembly.md) — DBP layout & admin: layout templates, basic info data tile, footers (simple/bottom nav bar), admin contact details (submission list/personal
  detail), beacon setup (tracking/fetch data), Assemble About Us tutorial (workspace/agent setup, screen assembly steps)
- [platform-topics.md](./platform-topics.md) — Extension points & security: Web Components (manifest format, hosting, slots), Wasm Functions (manifest, Rust/WASI patterns, mixer wasm build),
  Working with Files from Users (Local File type, upload methods), Cloud Agent Pubsub (server→client push), Component Overriding (overridable/callable pattern), Permissions Management
  (screen-level anonymous access), Public/Private/Ephemeral Files (path-prefix routing), Storing Secrets and Tokens (secrets/tokens API, OAuth originalToken endpoint)
- [platform-builder.md](./platform-builder.md) — Builder automation & runtime: Mixed Auth (fullscreen vs embedded `_rmx_auth`, fullscreen-embedded rmx-remix pattern), Macros (builder remote control:
  macro tiles + Mix macro module commands, node/binding/component manipulation, ndType/bdType/operator reference, createNodeFromAst), External Actions (runtime→host communication via onEvent map,
  default/mobile/local-storage/payment action catalog, moving to Client Actions), Edit/Run/Configure modes (3 L2 view modes, URL param, configurator node setup, fullscreen variants)
- [remix-nodes.md](./remix-nodes.md) — Remix Studio User Guide, Nodes section: In Params/Out Params (reorder bindings, agent error out-params), Cards & Components (style units, text input
  debounce, headless components, triggered components), File Uploading (Local File Controller, accept/multiple props, directory drop), Data Objects/Values/Transforms (Join node), Queries (QB 2.0,
  DB architecture, 3 query methods, Mix API, HTTP endpoint, live query, index-capable ops, history), Query Syntax (full filter/projection/aggregation/calculation reference), Actions (File Register,
  Agent Connect tiles: Service/Local/Amp types, Api node: token/multipart/client-side, Database Save/Delete), Service Agent modules ("Edit Cloud Server" navbar button, preview server vs installed workspace)
- [server-apis.md](./server-apis.md) — Auth integrations (OAuth/OIDC/Apple/SFDC plugin config, callback URLs, token storage, auth0 setup), Legacy Platform Server API (document CRUD, queries,
  app management, multi-app actions, .remix file endpoints, files API, agents/webhooks/lambda, resources, auth/token endpoints), Cloud Agent Server API (workspace/app management, agent execution,
  cloud queries, permissions, cloud topic subscription, .remix generation), Cloud Agent Server v1 API (workspace/app/agent introspection, permissions CRUD, files, signals, workspace import/export)
- [mix-std-lib.md](./mix-std-lib.md) — Mix Standard Library master index: all ~60 module names with 1-line descriptions, links to detail files
- [mix-lang-core.md](./mix-lang-core.md) — Mix stdlib scalar types & utilities: bool (Kleenean logic), number (arithmetic/math/format/bitsets), string (immutable Unicode,
  split/concat/JSON/patterns), option (null/some), result (error/ok), bitset (integer sets), passThrough (anonymous cells), regex (compile/find/replace/split/templates/lexer)
- [mix-lang-collections.md](./mix-lang-collections.md) — Mix stdlib collection types: array (map/filter/reduce/sort/join/slice), set (string sets), map (string-keyed maps,
  descend/invert), gset (generic sets), uniqArray (key-based unique arrays)
- [mix-lang-data.md](./mix-lang-data.md) — Mix stdlib database modules: db (CRUD, query pipeline, filter compilability, live queries, references, save options), db-changes (migration notes:
  queries no longer streams, no auto-fallback, no auto-ref-expansion), dbRemote (HTTP protocol variant), minidb (session-only pure-Mix Wasm-compatible DB, transactions), query (AST
  builder: filter/projection nodes, map constructors), registry (session-scoped case value store), resource (static resource URLs, deprecated → use file)
- [mix-lang-io.md](./mix-lang-io.md) — Mix stdlib I/O modules: http (HTTP client: verbs, config map, response shape, URL escaping), file (blob storage: get/read/list/create/register,
  client-side upload flow via remix/upload-file Client Action), binary (byte sequences: slice/concat/index, encode/decode: UTF-8/Base64/Hex/RmxCode), crypto (md5/sha1/sha256/hmac/rsa/aes-gcm),
  wasm (Wasm FFI: load/fun1-9/wasiLoad/wasiCall; prefer mixer wasm CLI), mime (MIME headers, media types, multipart/form-data format/parse)
- [mix-lang-stream.md](./mix-lang-stream.md) — Mix stdlib streaming modules: stream (lazy ops: map/filter/reduce/sort/zip/tee/innerJoin/grouped/lreduce/dedup/permute/enumUnbound),
  xml (parse/format, node types textNode/elementNode, shallow+deep search, id lookup, streaming, entity decoding; server-only), websocket (client: create config, read/write/channels,
  message types, close codes; Wasm limitations)
- [mix-lang-platform.md](./mix-lang-platform.md) — Mix stdlib platform modules: agent (local/remote/persistent descriptors, call options, asUser), auth (emptyToken, signIn/Up, resetPassword via
  auth0), env (runtime identity, hostEnvironment types, base URLs, URL builders, client props), secrets (per-user encrypted storage, OAuth token retrieval, tokenExtra), logging (info/warning/error
  → client console), metrics (timers 0–4, profiling cells/views/actions, tracing spreadsheet/action/pubsub, statistics, Mix event log level, Track API debug commands), co (coroutines
  create/createFG/exit/yield, channels emit/channelFrom/channelFromEH, error handling apply/onError/applyRaw, sleep/applyWithTimeout, mutexes, consume live cells, pub/sub pubsubCreate/Connect/Subscribe
  with $workspace/$wsUser/$app/$user/$session topic scopes), embed (show/hide/toggle by group+instance, onclose callback, placeholderStream, dropdown/tabbed pattern), track (amp session control via
  WebSocket: createRemote, msgSessionCreate/Compile/View message builders, response extractors)

---

## Notion Page Index

> Full sub-page hierarchy with descriptions: [NOTION-INDEX.md](./NOTION-INDEX.md)

### Remix Product
- [Remix product](https://www.notion.so/27e1d464528f802291b6d5a093fbc10d) → remix-product.md
- [Global Task List](https://www.notion.so/1d71d464528f80a69c47d21033bc498c) → remix-product.md

### Design Zone
- [Glossary](https://www.notion.so/3bea488f743e43f38c4b4c2b44d58000) → glossary.md
- [Collaboration & Assembly](https://www.notion.so/44d11988dfb24d0ea32fb8871637e72a) → collaboration-assembly.md
- [Federated Servers](https://www.notion.so/6496884844e74752af5b94d933afb22a) → federated-servers.md
- [Variants](https://www.notion.so/1361d464528f801db88efc7b8a4bc063) → variants.md
- [Design Proposal - L1 catalog assets](https://www.notion.so/13f1d464528f80f4a2ecceedce5a0051) → l1-catalog-assets.md

### Customer Success Projects
- [Customer Success Projects](https://www.notion.so/26f1d464528f80f3a6fffc1fe4cf294e) — hub page (all workspace IDs)
- [Funda](https://www.notion.so/26f1d464528f807f9782d5bffc5401a5) → funda.md
- [Lumber](https://www.notion.so/28c1d464528f801bac72d6c064d30db2) → lumber.md (sub-pages: Workforce Junction, Schema, Job Costing, Payroll)
- [Orderly](https://www.notion.so/2781d464528f80e6b282fa5ce3050372) → customer-projects-other.md
- [Bomisco](https://www.notion.so/2bc1d464528f806f94c9ce622bddacab) → customer-projects-other.md
- [5 Star Music Kanban](https://www.notion.so/26a1d464528f8095b9b6e072efec37cd) → customer-projects-other.md
- [Tapped In Interactions](https://www.notion.so/10c1d464528f804bb97fd9f061d7fbf2) → customer-projects-other.md

### GTM & Content Operations
- [Remix Next GTM](https://www.notion.so/2001d464528f803293aaebc066f1acb4) → remix-product.md
- [Tasks - Asset Submission Guide](https://www.notion.so/2111d464528f8031a05fd76e687b6379) → remix-infra.md
- [MCP Tools](https://www.notion.so/2141d464528f8070b79bfdece10ff94f) → remix-tools.md (sub-pages: Tool Prompting, Installation Guide)

### Library & Content Assets
- [AI agents](https://www.notion.so/2991d464528f8043b82ff22040962cc2) → remix-catalog.md
- [HubSpot toolkit](https://www.notion.so/1371d464528f809b9f51cf5b4337a137) → hubspot-toolkit.md (sub-pages: assets, OAuth setup, install instructions)
- [DuckDB](https://www.notion.so/2991d464528f80429000f327b27d8845) → duckdb.md
- [OAuth Handler](https://www.notion.so/29a1d464528f807784fcdce13675f665) → oauth.md
- [Snowflake — Cortex Search](https://www.notion.so/2aa1d464528f8016ab74d92bc328935b) → remix-snowflake.md
- [SMB catalog](https://www.notion.so/1051d464528f80a09405d83aec33ffb9) → remix-smb.md (sub-pages: transaction models, data model, assets)
- [Business Profile / About Us Catalog](https://www.notion.so/1051d464528f801e975cc2bf6cd715b9) → remix-dbp-media.md / remix-dbp-components.md / remix-dbp-assembly.md

### Documentation
- [Remix Documentation (root)](https://www.notion.so/fe31096a438b4f29b103c189bc9a7fb8) — not yet fetched
- [Getting Started in Remix Studio](https://www.notion.so/1051d464528f8008b584dd33015fd5b3) → glossary.md
- [User Guides and Tutorials](https://www.notion.so/3e247ff418a94194a8376371117c6bb2) → platform-topics.md / platform-builder.md / federated-servers.md / remix-dbp-assembly.md
- [Deploying Remix applications](https://www.notion.so/11b1d464528f8030a107dc7f5e8d27cc) → remix-infra.md / remix-deploy.md
- [Remix Tools](https://www.notion.so/13b1d464528f80d593a7f1de1e0d6d43) → remix-tools.md
- [Remix Studio User Guide](https://www.notion.so/1051d464528f8007a5f5d40969c49427) — not yet fetched (top-level); sub-pages → remix-nodes.md / platform-builder.md / server-apis.md / remix-infra.md
- [Cloud Agent Server docs](https://www.notion.so/1061d464528f80f18e46dd27895aeda2) → server-apis.md
- [Mix Language docs](https://www.notion.so/259ede6504e34505982dde5dc4b63d10) → mix-std-lib.md + mix-lang-*.md
