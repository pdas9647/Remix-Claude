# Remix Labs — Project Memory

Load topic files first; fetch linked Notion pages if still needed.

Notion index:
- [notion-index-general.md](./notion-index-general.md) — General teamspace (Product, Connectors, GTM, MCP)
- [notion-index-customer.md](./notion-index-customer.md) — Customer Success (Funda, Lumber, Orderly, etc.)
- [notion-index-doc-studio.md](./notion-index-doc-studio.md) — Docs: Studio & Platform (User Guide, Nodes, Server APIs, Deploying, Tools)
- [notion-index-doc-mix.md](./notion-index-doc-mix.md) — Docs: Mix Language (syntax, type system, screens/agents, build/driver, stdlib)
- [notion-index-doc-catalogs.md](./notion-index-doc-catalogs.md) — Docs: Catalogs & Toolkits (Base, HubSpot, SMB, DBP, AI, DuckDB, OAuth, Snowflake)
- [notion-index-ux-design.md](./notion-index-ux-design.md) — UX & Design (Collaboration, Variants, L1 catalog)
- [notion-index-engineering.md](./notion-index-engineering.md) — Platform Engineering (CI, engineering process)

## Topic Files

### Product & Strategy
- [glossary.md](./glossary.md) — Studio terminology, L0-L3 navigation, module types, node catalog
- [collaboration-assembly.md](./collaboration-assembly.md) — Developer > Assembler > Consumer pipeline, config points
- [remix-product.md](./remix-product.md) — Global Task List, desktop design, pricing/SKU, runtime strategy
- [remix-product-scope.md](./remix-product-scope.md) — First release scope, MoSCoW, 5 personas, self-serve vs CS

### Design & Architecture
- [federated-servers.md](./federated-servers.md) — Federated search, L2SourceNode, Catalog & Collections, Faceted Search, Catalog V2
- [variants.md](./variants.md) — Config assets: base/config/instance, deep sync, L2SourceNode variant format
- [l1-catalog-assets.md](./l1-catalog-assets.md) — L1 catalog publish/search/sync design proposal

### Customer Projects
- [funda.md](./funda.md) — Funda: member directory, sEt2qxPydL, DBP/Admin/Member apps, LinkedIn clipper
- [customer-projects-other.md](./customer-projects-other.md) — Orderly, 5 Star Music (Twilio IVR), Bomisco, Tapped In; workspace IDs
- [lumber.md](./lumber.md) — Lumber: job costing (lfi_), Workforce Junction (h_), payroll (prl_), Snowflake migration

### Library & Catalog Assets
- [remix-catalog.md](./remix-catalog.md) — Widget/base catalog, remix_labs library, connectors, AI agents, GTM, T1/T2
- [hubspot-toolkit.md](./hubspot-toolkit.md) — HubSpot Connector/Configurator/Library/Example apps, OAuth, installation, widget guide
- [connectors.md](./connectors.md) — Notion, Gmail, Google Sheet, Airtable, Confluence, Slack connectors; auth/OAuth/MCP
- [remix-snowflake.md](./remix-snowflake.md) — Snowflake/Snowpark, Cortex AI/Search, connectors, widgets, OAuth + service account
- [duckdb.md](./duckdb.md) — DuckDB: DuckLake attach + query (Parquet-backed)
- [oauth.md](./oauth.md) — Third-party OAuth: Desktop (popup), Chrome Extension, Mobile Widgets flows
- [remix-smb.md](./remix-smb.md) — SMB: 3 transaction models (business/venue/catalog/order), Stripe, printer
- [remix-dbp-media.md](./remix-dbp-media.md) — DBP media: video players, galleries, images, data viz (quote/percentage/bar)
- [remix-dbp-components.md](./remix-dbp-components.md) — DBP components: buttons, overlay modal, text lists, reviews, forms, banners
- [remix-dbp-assembly.md](./remix-dbp-assembly.md) — DBP layout: templates, info tile, footers, admin contact, beacon

### Platform & Infrastructure
- [remix-infra-auth.md](./remix-infra-auth.md) — Workspace-Based Auth (org/token arch, sign-in flow), Unified Login
- [remix-infra-setup.md](./remix-infra-setup.md) — _rmx_sync (app syncing, agent API), _rmx_prefs (6-agent cascade), deeplinks
- [remix-deploy.md](./remix-deploy.md) — 10 deployment surfaces, self-hosted server, .remix files, iOS/Android widgets
- [remix-tools.md](./remix-tools.md) — mix_client REPL, bug reporting, MCP tools, Cloud Workspace Tool
- [platform-topics.md](./platform-topics.md) — Web Components, Wasm Functions, files, pubsub, permissions, secrets/tokens
- [platform-builder-runtime.md](./platform-builder-runtime.md) — Mixed Auth (_rmx_auth), External Actions (onEvent map, mobile/storage/payment), Window mgmt
- [platform-builder-macros.md](./platform-builder-macros.md) — Macros (builder remote ctrl, ndType/bdType/operator ref, createNodeFromAst), _rmx_init
- [remix-nodes-basics.md](./remix-nodes-basics.md) — Nodes: In/Out Params, Cards & Components, File Uploading, Data/Transforms, State
- [remix-nodes-queries.md](./remix-nodes-queries.md) — Queries (QB 2.0, DB arch, Mix/HTTP query API, live query), Query Syntax
- [remix-nodes-actions.md](./remix-nodes-actions.md) — Actions: File Register, Agent Connect (Service/Local/Amp), Api node, DB Save/Delete
- [server-apis-auth.md](./server-apis-auth.md) — Auth integrations (OAuth/OIDC/Apple/SFDC configs, callback URLs, token storage)
- [server-apis-legacy.md](./server-apis-legacy.md) — Legacy Platform Server API (doc CRUD, queries, app mgmt, .remix install, files, agents)
- [server-apis-cloud.md](./server-apis-cloud.md) — Cloud Agent Server: arch (RBAC), Mixer API (run-agent, remixgen), v1 API (permissions, files)
- [engineering-ci.md](./engineering-ci.md) — CI: CircleCI setup, contexts (rcm-reader/editor, slack-alerts), Docker image hierarchy

### Mix Language
- [mix-syntax-core.md](./mix-syntax-core.md) — modules/imports, cells (links/aliases/observers/live), defs, statements, actions
- [mix-syntax-expressions.md](./mix-syntax-expressions.md) — expressions, literals, lambdas, operators, patterns, types, cases, projections
- [mix-syntax-semantics.md](./mix-syntax-semantics.md) — undefined semantics (Kleene &&/||, bubbling, filters), operator semantics
- [mix-syntax-style.md](./mix-syntax-style.md) — directives (#line/#push/#pop), pragmas (typing/warning), _meta_, style guide
- [mix-type.md](./mix-type.md) — Type system: hierarchy, type vars, case/union, hints vs assertions, labeled tuples
- [mix-build.md](./mix-build.md) — Build system: library/executable records, mixc CLI, dependency rules
- [mix-agent.md](./mix-agent.md) — _rmx_sessionAgent, _rmx_agentAgent, _rmx_makeAgent (build/deploy pipeline)
- [mix-screen.md](./mix-screen.md) — _rmx_auth, _rmx_error/_rmx_errorFallback, _rmx_entry, _rmx_debugShell
- [mix-driver-compiler.md](./mix-driver-compiler.md) — Driver lib: compilerRule/File/Lib/Exe (DB records), compilerDriver (session orch)
- [mix-driver-build.md](./mix-driver-build.md) — Driver lib: compilerBuild, codegenDriver/Util, remixFile (.remix zip), remixRecipe

### Mix Standard Library
- [mix-std-lib.md](./mix-std-lib.md) — ~60 module master index with descriptions
- [mix-lang-core.md](./mix-lang-core.md) — bool, number, string, option, result, bitset, passThrough, regex
- [mix-lang-collections-basic.md](./mix-lang-collections-basic.md) — array, set, map
- [mix-lang-collections-advanced.md](./mix-lang-collections-advanced.md) — gset, uniqArray, circuitarray
- [mix-lang-db.md](./mix-lang-db.md) — db, db-changes, dbRemote, minidb (CRUD, live queries, compilability, references)
- [mix-lang-query.md](./mix-lang-query.md) — query AST builder (filter/projection/aggregation, map constructors), resource
- [mix-lang-io.md](./mix-lang-io.md) — http, file, binary, crypto, wasm, mime
- [mix-lang-stream.md](./mix-lang-stream.md) — stream, xml, websocket
- [mix-lang-editor-dom.md](./mix-lang-editor-dom.md) — dom/domUtil, sync cell protocol, editorTypes, editorNode, editorScreen
- [mix-lang-editor-reflect.md](./mix-lang-editor-reflect.md) — reflect, reflectComponent, slateComponent
- [mix-lang-editor-slate.md](./mix-lang-editor-slate.md) — slateDef, slateDyn, slateGen
- [mix-lang-editor-runtime.md](./mix-lang-editor-runtime.md) — container, viewstack, debugger, testing, compiler, vm
- [mix-lang-advanced.md](./mix-lang-advanced.md) — Recursion, stream mutability, pattern matching, modules advanced
- [mix-lang-advanced-runtime.md](./mix-lang-advanced-runtime.md) — Links (advanced), EscJSON representation, DB query notes, embedding viewstacks
- [mix-lang-platform-runtime.md](./mix-lang-platform-runtime.md) — agent, auth, env, secrets (runtime identity, invocation, OAuth tokens)
- [mix-lang-platform-concurrency.md](./mix-lang-platform-concurrency.md) — logging, metrics, co (coroutines/channels/pubsub), embed, track, memo, util
- [mix-lang-platform-messaging.md](./mix-lang-platform-messaging.md) — calendar, color, messaging (pub/sub topics, cloud/remote), loader (dynamic lib loading)

### Slack Channels
- [slack-general.md](./slack-general.md) — #general: team structure, standups, GTM pipeline, platform incidents
- [slack-announcements.md](./slack-announcements.md) — #announcements: Lumber coordination, desktop readiness, theme migration, WebAuthn design
- [slack-backend.md](./slack-backend.md) — #backend: DB save FFI overhaul, wasm caching, blob URL support, query pipeline, Go/Rust parity
- [slack-builder-runtime.md](./slack-builder-runtime.md) — #builder-runtime: builder features, amp cleanup, catalog items, remote data sources, L0 plugins
- [slack-bugbash.md](./slack-bugbash.md) — #bugbash: open bugs, in-progress fixes, recently resolved
- [slack-design.md](./slack-design.md) — #design: library sync, theme overhaul, TUI grid, L0 desktop org, tailwind fixes, UX bugs
- [slack-desktop.md](./slack-desktop.md) — #desktop: native vs browser debate, app sync, auth bugs, Tauri limitations, release channels
- [slack-dumbquestionsanswered.md](./slack-dumbquestionsanswered.md) — #dumbquestionsanswered: env.runtimeURL deprecation, platform Q&A
- [slack-extension.md](./slack-extension.md) — #extension: Chrome extension release log
- [slack-flutter.md](./slack-flutter.md) — #flutter: mobile app releases, widgets iOS/Android, federated search, deeplinks
- [slack-go-to-market.md](./slack-go-to-market.md) — #go-to-market: GTM strategy, Atomicwork/Alteryx/Simpplr, Snowflake pitch, AI PRD
- [slack-ops.md](./slack-ops.md) — #ops: weekly production promotion workflow, harmony PRs, TF build log
- [slack-rmx-delivery-5starmusic.md](./slack-rmx-delivery-5starmusic.md) — #rmx-delivery-5starmusic: IVR+CRM build, go-live Nov 2025, SMS architecture
- [slack-rmx-delivery-bomisco.md](./slack-rmx-delivery-bomisco.md) — #rmx-delivery-bomisco: LinkedIn→HubSpot clipper, V1/V2 .remix issue, Copilot UX overhaul
- [slack-rmx-delivery-funda.md](./slack-rmx-delivery-funda.md) — #rmx-delivery-funda: member directory, LinkedIn auth, Snowflake Cortex search, go-live prep
- [slack-rmx-delivery-lumber.md](./slack-rmx-delivery-lumber.md) — #rmx-delivery-lumber: analytics reporting (Phase 2), Snowflake ETL, report builder
- [slack-rmx-delivery-snowflake.md](./slack-rmx-delivery-snowflake.md) — #rmx-delivery-snowflake: Snowflake Startup Accelerator, SPCS deployment, OAuth, GTM

### GitHub Repositories
- [github-mix-rs.md](./github-mix-rs.md) — Rust runtime: value system, query engine, FFI bridge, Wasm VM, agent server CLI, Tauri
- [github-protoquery.md](./github-protoquery.md) — Mix compiler (OCaml): Lexer→Parser→AST→QCode→Linker→Wasm, VM bridge, ABI
- [github-turntable.md](./github-turntable.md) — Studio frontend (Elm): node-graph IDE (builder), dual VM (AMP/Wasm), 8 web components
- [github-flutter-runtime.md](./github-flutter-runtime.md) — Mobile runtime (Flutter/Dart): AMP server, WebView, iOS/Android widgets, AppClip, push, IAP
- [github-amp.md](./github-amp.md) — Platform server (Go): HTTP API, Groovebox Wasm VM, MQTT broker, JWT/OAuth, gomobile
- [github-web-components.md](./github-web-components.md) — Standalone web components (TS/JS/Elm/SolidJS): 13 components, Vite builds, manifest-based Studio integration

### Standup Notes
- [standup-notes-jan-2026.md](./standup-notes-jan-2026.md) — Standup notes: decisions, ownership, customer updates, GTM pipeline (Jan 2026)
- [standup-notes-feb-2026.md](./standup-notes-feb-2026.md) — Standup notes Feb 17–27: Lumber CTE facets, delivery process, beta QA plan, workspace-as-environment
- [standup-notes-feb-2026-2.md](./standup-notes-feb-2026-2.md) — Standup notes Feb 2–13: engineering process, desktop bugs, Atomicwork widgets, sprint kickoff