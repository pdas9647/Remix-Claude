# Remix Labs — Project Memory

Persistent context for Remix Labs tasks. When memory is insufficient, read the relevant topic file below, then fetch linked Notion pages if still needed.

Notion page index (split by teamspace, load only what's needed):
- [notion-index-general.md](./notion-index-general.md) — General teamspace (Product, Connectors, GTM, MCP)
- [notion-index-customer.md](./notion-index-customer.md) — Customer Success (Funda, Lumber, Orderly, etc.)
- [notion-index-documentation.md](./notion-index-documentation.md) — Documentation (Studio User Guide, Mix Language, Catalogs)
- [notion-index-ux-design.md](./notion-index-ux-design.md) — UX & Design (Collaboration, Variants, L1 catalog)

---

## Topic Files

### Product & Strategy

- [glossary.md](./glossary.md) — Remix Studio terminology, L0-L3 navigation, module types, node catalog (colors, 3 escape hatches)
- [collaboration-assembly.md](./collaboration-assembly.md) — Steel thread: Developer > Assembler > Consumer pipeline, config points
- [remix-product.md](./remix-product.md) — Global Task List, desktop design, pricing/SKU, runtime strategy
- [remix-product-scope.md](./remix-product-scope.md) — First release scope, MoSCoW, 5 personas, self-serve vs CS

### Design & Architecture

- [federated-servers.md](./federated-servers.md) — Federated search, external libraries (Amp/Workspace), asset sync, L2SourceNode, Catalog & Collections, Faceted Search, Catalog V2
- [variants.md](./variants.md) — Config assets: base/config/instance, deep sync, L2SourceNode variant format
- [l1-catalog-assets.md](./l1-catalog-assets.md) — L1 catalog publish/search/sync design proposal

### Customer Projects

- [funda.md](./funda.md) — Funda: member directory, workspace sEt2qxPydL, DBP/Admin/Member apps, LinkedIn clipper
- [customer-projects-other.md](./customer-projects-other.md) — Orderly, 5 Star Music (Twilio IVR), Bomisco, Tapped In; all workspace IDs
- [lumber.md](./lumber.md) — Lumber: job costing (lfi_), Workforce Junction (h_), payroll (prl_ 38 tables), Snowflake migration

### Library & Catalog Assets

- [remix-catalog.md](./remix-catalog.md) — Widget/base catalog, HubSpot research, remix_labs library, connectors, AI agents, GTM, T1/T2
- [hubspot-toolkit.md](./hubspot-toolkit.md) — HubSpot Connector/Configurator/Library/Example apps, OAuth setup, installation
- [connectors.md](./connectors.md) — Third-party connector setup: Notion, Gmail, Google Sheet, Airtable, Confluence, Slack (auth flows, OAuth config, MCP agents)
- [remix-snowflake.md](./remix-snowflake.md) — Snowflake/Snowpark, Cortex AI/Search, connectors, widgets, OAuth + service account setup
- [duckdb.md](./duckdb.md) — DuckDB: DuckLake attach + query (Parquet-backed)
- [oauth.md](./oauth.md) — Third-party OAuth: Desktop (popup), Chrome Extension, Mobile Widgets flows
- [remix-smb.md](./remix-smb.md) — SMB: 3 transaction models, data model (business/venue/catalog/order), Stripe, printer
- [remix-dbp-media.md](./remix-dbp-media.md) — DBP media: video players, galleries, images, data viz (quote/percentage/bar)
- [remix-dbp-components.md](./remix-dbp-components.md) — DBP components: buttons, overlay modal, text lists, reviews, forms, banners
- [remix-dbp-assembly.md](./remix-dbp-assembly.md) — DBP layout: templates, basic info tile, footers, admin contact, beacon, assembly tutorial

### Platform & Infrastructure

- [remix-infra.md](./remix-infra.md) — Auth (workspace-based), Unified Login, Org/System Setup, _rmx_sync, _rmx_prefs, deeplinks
- [remix-deploy.md](./remix-deploy.md) — 10 deployment surfaces, self-hosted server, .remix files, iOS/Android widgets, widget debugging
- [remix-tools.md](./remix-tools.md) — mix_client REPL, bug reporting, asset submission, MCP tools, Cloud Workspace Tool, Repository Tool
- [platform-topics.md](./platform-topics.md) — Web Components, Wasm Functions, files, pubsub, component overriding, permissions, secrets/tokens
- [platform-builder.md](./platform-builder.md) — Mixed Auth, Macros, External Actions, Edit/Run/Configure modes, _rmx_init
- [remix-nodes-basics.md](./remix-nodes-basics.md) — Nodes: In/Out Params, Cards & Components, File Uploading, Data Objects/Transforms, Working with State
- [remix-nodes-queries.md](./remix-nodes-queries.md) — Queries (QB 2.0, DB architecture, Mix/HTTP query API, live query), Query Syntax (filter/projection/aggregation/calculation)
- [remix-nodes-actions.md](./remix-nodes-actions.md) — Actions: File Register, Agent Connect (Service/Local/Amp), Api node (multipart/client-side), Database Save/Delete, Service Agent modules
- [server-apis-auth.md](./server-apis-auth.md) — Auth integrations (OAuth/OIDC/Apple/SFDC plugin configs, callback URLs, token storage, auth0)
- [server-apis-legacy.md](./server-apis-legacy.md) — Legacy Platform Server API (document CRUD, queries, app mgmt, .remix install, files, agents, auth endpoints)
- [server-apis-cloud.md](./server-apis-cloud.md) — Cloud Agent Server API (workspace mgmt, run-agent, cloud queries, remixgen) + v1 API (introspection, permissions, files, signals)

### Mix Language

- [mix-syntax-core.md](./mix-syntax-core.md) — modules/imports, cells (links/aliases/observers/live), defs (foreign/recursive), statements, actions
- [mix-syntax-expressions.md](./mix-syntax-expressions.md) — expressions, literals, lambdas, operators, patterns, types, cases, short projections
- [mix-syntax-semantics.md](./mix-syntax-semantics.md) — undefined semantics (Kleene &&/||, bubbling, filters), operator semantics, stmt-vs-expr, action closures
- [mix-syntax-style.md](./mix-syntax-style.md) — directives (#line/#push/#pop), pragmas (typing/warning), _meta_, code style guide
- [mix-type.md](./mix-type.md) — Type system: hierarchy, type vars, definitions, case/union, hints vs assertions, labeled tuples
- [mix-build.md](./mix-build.md) — Build system: library/executable records, mixc CLI, dependency rules, lib_header
- [mix-agent.md](./mix-agent.md) — _rmx_sessionAgent, _rmx_agentAgent, _rmx_makeAgent (build/deploy pipeline)
- [mix-screen.md](./mix-screen.md) — _rmx_auth, _rmx_error/_rmx_errorFallback, _rmx_entry, _rmx_debugShell
- [mix-driver-compiler.md](./mix-driver-compiler.md) — Driver lib: compilerRule/File/Lib/Exe (DB records), compilerDriver (session orchestration), compilerREPL (send to VM), compilerClean
- [mix-driver-build.md](./mix-driver-build.md) — Driver lib: compilerBuild (high-level build pipeline), codegenDriver/Util (Mix source from screens), remixFile (.remix zip builder), remixRecipe (declarative recipe types)

### Mix Standard Library

- [mix-std-lib.md](./mix-std-lib.md) — ~60 module master index with descriptions
- [mix-lang-core.md](./mix-lang-core.md) — bool, number, string, option, result, bitset, passThrough, regex
- [mix-lang-collections.md](./mix-lang-collections.md) — array, set, map, gset, uniqArray, circuitarray
- [mix-lang-data.md](./mix-lang-data.md) — db, db-changes, dbRemote, minidb, query, registry, resource
- [mix-lang-io.md](./mix-lang-io.md) — http, file, binary, crypto, wasm, mime
- [mix-lang-stream.md](./mix-lang-stream.md) — stream, xml, websocket
- [mix-lang-editor-dom.md](./mix-lang-editor-dom.md) — dom/domUtil, sync cell protocol, editorTypes, editorNode, editorScreen
- [mix-lang-editor-reflect.md](./mix-lang-editor-reflect.md) — reflect, reflectComponent, slateComponent
- [mix-lang-editor-slate.md](./mix-lang-editor-slate.md) — slateDef, slateDyn, slateGen
- [mix-lang-editor-runtime.md](./mix-lang-editor-runtime.md) — container, viewstack, debugger, testing, compiler, vm
- [mix-lang-advanced.md](./mix-lang-advanced.md) — Recursion, stream mutability, pattern matching, modules advanced, links, EscJSON, embedding viewstacks
- [mix-lang-platform-runtime.md](./mix-lang-platform-runtime.md) — agent, auth, env, secrets (runtime identity, invocation, OAuth tokens)
- [mix-lang-platform-concurrency.md](./mix-lang-platform-concurrency.md) — logging, metrics, co (coroutines/channels/pubsub), embed, track, memo, random, util
- [mix-lang-platform-messaging.md](./mix-lang-platform-messaging.md) — calendar, color, messaging (pub/sub topics, cloud/remote), loader (dynamic library loading)

---

## Maintenance Notes

- **Phase 3 pending**: Split 8 oversized topic files (>18KB) into smaller sub-files for better context efficiency.
  Files to split: mix-lang-data.md (19K)
  Already split: mix-lang-platform.md → 3 files, mix-lang-editor.md → 4 files, mix-syntax.md → 4 files, server-apis.md → 3 files, remix-nodes.md → 3 files, mix-driver.md → 2 files
  Already split: mix-lang-platform.md → 3 files, mix-lang-editor.md → 4 files, mix-syntax.md → 4 files, server-apis.md → 3 files
- **Weekly update cadence**: Check Notion pages modified in last 7 days, Slack channels for last 3-4 days
- **Last refreshed**: 2026-02-21
