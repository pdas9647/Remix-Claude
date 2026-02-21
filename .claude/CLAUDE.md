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
- [mix-agent.md](./mix-agent.md) — Mix special agents (in static-agents.mixexe): `_rmx_sessionAgent` (compiler+VM session driver: compilerOptions/database/variant/localAgentsConfig params, MQTT control via sessionTopic/machineTopic, msg_compile_runPhrases, idle action pattern), `_rmx_agentAgent` (on-demand agent VM spawner, used by sessionAgent), `_rmx_makeAgent` (build agent: targetLibs/targetExe/targetStandaloneExes, sources/database/filestore input, full deploy pipeline with deployMode/deployToServer/storeRemixFile)
- [mix-screen.md](./mix-screen.md) — Mix special screens: `_rmx_auth` (auth flow, params: reqName/reqParams/authRequest), `_rmx_error`/`_rmx_errorFallback` (error recovery, enable_error_page client var), `_rmx_entry` (external request interceptor, entryRedirection PR1404), `_rmx_debugShell` (test automation stub)
- [mix-driver.md](./mix-driver.md) — Mix `driver` extra core library: `compilerRule` (build rule records: rule/ruleContent/scanFilter types, scan/load/save/delete/makeLibRule/saveLibRule/getExeRuleHash)
- [mix-build.md](./mix-build.md) — Mix build system: library/executable DB record format (name+version+hash, lib_header/lib_stdlib/dataRef), mixc CLI (mklib/ldd/list-libs/clean), dependency rules (stdlib versioning, compatible lib search, force-link with import), track API for library creation, inspecting lib_header params
- [mix-type.md](./mix-type.md) — Mix type system: type hierarchy (null/bool/number/string/word/data/any/undefined), type variables (unconstrained/anonymous/constrained), type definitions (aliases with params), case/union types, type hints (compile-time, no code) vs type assertions (type(T)/runtime check, data-range only), labeled tuples (require type def, labels compile-time only), word/data/any upcasting/downcasting, + operator overloading, typed queries pattern (type() at end), locally polymorphic types (~forall, since PR#1424)
- [mix-syntax.md](./mix-syntax.md) — Mix language syntax reference: modules/imports, cells (var/private/live/alias), initializers, observers (on/when/cellUpdated/cellChanged), defs (functional/recursive/foreign), statements (let/var/if/switch/while/for/assignments), actions (appstate), expressions (let alias/do/if/undefined/literals/lambda/currying/try/operators), patterns (destructure/cases/data types/guards/@-capture/~multiple), types, cases (union/inverse ^), short projections; PLUS: stmt-vs-expr rules (what's forbidden in exprs, do blocks, appstate cell mutation), undefined semantics (bubbling/Kleene &&||/filters/JSON serialization), operator semantics (null handling/== deep equality/ordering/datacase/debug)
- [mix-std-lib.md](./mix-std-lib.md) — Mix Standard Library master index: all ~60 module names with 1-line descriptions, links to detail files
- [mix-lang-core.md](./mix-lang-core.md) — Mix stdlib scalar types & utilities: bool (Kleenean logic), number (arithmetic/math/format/bitsets), string (immutable Unicode,
  split/concat/JSON/patterns), option (null/some), result (error/ok), bitset (integer sets), passThrough (anonymous cells), regex (compile/find/replace/split/templates/lexer)
- [mix-lang-collections.md](./mix-lang-collections.md) — Mix stdlib collection types: array (map/filter/reduce/sort/join/slice), set (string sets), map (string-keyed maps,
  descend/invert), gset (generic sets), uniqArray (key-based unique arrays), circuitarray (module-per-row instantiation: make/makeCached/makeOnce/makeWithKeyedRowCache/extract/setMaxAge)
- [mix-lang-data.md](./mix-lang-data.md) — Mix stdlib database modules: db (CRUD, query pipeline, filter compilability, live queries, references, save options), db-changes (migration notes:
  queries no longer streams, no auto-fallback, no auto-ref-expansion), dbRemote (HTTP protocol variant), minidb (session-only pure-Mix Wasm-compatible DB, transactions), query (AST
  builder: filter/projection nodes, map constructors), registry (session-scoped case value store), resource (static resource URLs, deprecated → use file)
- [mix-lang-io.md](./mix-lang-io.md) — Mix stdlib I/O modules: http (HTTP client: verbs, config map, response shape, URL escaping), file (blob storage: get/read/list/create/register,
  client-side upload flow via remix/upload-file Client Action), binary (byte sequences: slice/concat/index, encode/decode: UTF-8/Base64/Hex/RmxCode), crypto (md5/sha1/sha256/hmac/rsa/aes-gcm),
  wasm (Wasm FFI: load/fun1-9/wasiLoad/wasiCall; prefer mixer wasm CLI), mime (MIME headers, media types, multipart/form-data format/parse)
- [mix-lang-stream.md](./mix-lang-stream.md) — Mix stdlib streaming modules: stream (lazy ops: map/filter/reduce/sort/zip/tee/innerJoin/grouped/lreduce/dedup/permute/enumUnbound),
  xml (parse/format, node types textNode/elementNode, shallow+deep search, id lookup, streaming, entity decoding; server-only), websocket (client: create config, read/write/channels,
  message types, close codes; Wasm limitations)
- [mix-lang-editor.md](./mix-lang-editor.md) — Mix stdlib editor/reflection modules: dom/domUtil (DOM type, sync cell, propertyMap/Add, createNode/Tree/CellRef, toStream/fromStream, symbol
  table, domUtil.restore; don't transmit dom between sessions), editorTypes (direction input/output, visualType vty_* cases, exportVisualType/importVisualType), editorNode (node introspection:
  binding/connection types, byParentRef/filterBy* query builders, toStream/first, children/siblings traversal, followInConnection/followOutConnection), editorScreen (byRef/byName/screens/all,
  getSymbols/getStyles, children→editorNode.query), reflect (modules/moduleInfo/moduleQuery/moduleCheck, screens/agents/components lists + info/query, units/unitQuery, push/pushWithTransition,
  spreadsheet introspection, disablePropagation/enablePropagation, decomposeCase/decomposeCaseData), reflectComponent (instantiateComponent by name → output+view; slow), slateComponent (run
  mesh as component: mod_name/mesh/params → output+view; slow), slateDef (mesh building: createMesh/ScreenParams/OutputParams/ScreenCard/ComponentInstance/ListConstructor/Constant/
  ValueWithPlaceholders/Forwarder/OnLoadEmitter; labels addLabel/lookup/lookupMatch; wire connectInWithOut/append; unwire disconnect*; compose addMesh/strip; transform transformDelete/Replace;
  exportMesh/importMesh; collections), slateDyn (session-scoped: register/registerWithUpdate/registerAgent/registerComponent in _rmx_init; loadCollections), slateGen (generateCode/saveCode/
  getImports), container (safeGet/descend, pathUpdate/pathUpdateAndAugment, shallowMap/deepMap, shallowFind/deepFind/deepFindPath, shallowMerge/deepMerge/deepMergeBuilder/mergeReplaceBuilder,
  toStream/toStreamPairs), viewstack (load/push/switchTop/pop/toFront/reset/refresh; drop/swap; dynamicAppView/screenAppView/moduleAppView; schedule/scheduleFG/scheduleBG; embed/embedWithOptions/
  embedRemote/terminate/cellMessages/invoke/close/embedCompiler/getSessionOfRemote/sendRequestToRemote; selfTerminate; propagate/flush), debugger (debugCommand, debugEvents, debugMirror; cmd_*
  builders for profile/trace/stats/messaging/viewstack/view/testing), testing (startRecording/stopRecording; createTestSequence; snapshots snapshotCreate/Restore/List; replayTest/Visually/
  InSession/AllTests; BRP brpPull/brpInstruct with resume|abort|acceptChange), compiler (type module: libID/library/executable/storedBinary/header/pubMode; formatLibID/parseLibID; blobLoad/Save),
  vm (memstats; preferences; create/destroy connector; outputStream/outputFind; sendCode; load/invoke/introspect_spreadsheet)
- [mix-lang-advanced.md](./mix-lang-advanced.md) — Mix advanced patterns: recursion (mutual recursion, `recurse`/`recurse2`, cells-can't-recurse), stream mutability (first/skip consume), pattern-matching additions (optional field `?`, limitations), modules advanced (singleton, `currentInstanceId`, `cellSessionScope`/`cellScreenScope`, module type vars, REPL commands)
- [mix-lang-platform.md](./mix-lang-platform.md) — Mix stdlib platform modules: agent (local/remote/persistent descriptors, call options, asUser), auth (emptyToken, signIn/Up, resetPassword via
  auth0), env (runtime identity, hostEnvironment types, base URLs, URL builders, client props), secrets (per-user encrypted storage, OAuth token retrieval, tokenExtra), logging (info/warning/error
  → client console), metrics (timers 0–4, profiling cells/views/actions, tracing spreadsheet/action/pubsub, statistics, Mix event log level, Track API debug commands), co (coroutines
  create/createFG/exit/yield, channels emit/channelFrom/channelFromEH, error handling apply/onError/applyRaw, sleep/applyWithTimeout, mutexes, consume live cells, pub/sub pubsubCreate/Connect/Subscribe
  with $workspace/$wsUser/$app/$user/$session topic scopes), embed (show/hide/toggle by group+instance, onclose callback, placeholderStream, dropdown/tabbed pattern), track (amp session control via
  WebSocket: createRemote, msgSessionCreate/Compile/View message builders, response extractors), memo (mutable add/remove container, unique ordered handles, addBy for self-ref, no replacement), random
  (pseudoRandomInt/Float/String seeded at session start, cryptoRandomString for session IDs, predefined alphabet constants), util (withDefault/noDefault global helpers, isDefined/isUndefined,
  ifThenElse without shortcut eval), calendar (t_date/t_dateTime/t_timestamp/t_time types, errorMode failIfBad/undefIfBad/normIfBad, make*/import*/repr* constructors, year/month/day projections,
  toDate/toDateTime/toTimestamp/toString ISO-8601/toDBString UTC/format strftime conversions, set*/add*/sub*/diff* arithmetic, now()/localNow(), dayRange stream, periods makePeriod/periodRange, weeks
  weekUS/weekISO/weekTable/weekRangeISO, quarters makeQuarter/quarterRange), color (SRGB/LinRGB/HSL/HWB spaces double-precision; SRGB_unit/8bit/LinRGB/HSL/HWB constructors; import*/export*;
  toSRGB/toLinRGB/toHSL/toHWB; parseHex/formatHex/formatCSS; add/sub/blend/blendMany; lighter/darker/luminance/perceivedLightness/contrastRatio WCAG; alpha/updateAlpha), messaging (high-level
  pub/sub: viewTopic/sessionTopic/appTopic/userTopic/wsUserTopic/wsTopic, retained set/get vs non-retained send, subscribe/subscribeFG/unsubscribe/close, live queries subscribeToQuery, remote orgs
  mountOrg/remoteWsTopic etc., cloud topics cloudAppUserTopic/SharedTopic/PublicTopic + remote variants), loader (dynamic library loading: load(appstate,sources,libraries,exposedObjects,options),
  expose screens/agents/components by name or *, exposeAs/exposeWithDifferentPrefix, screen _meta_ loadComponents declaration, _dynamicExport_/_dynamicImport_/_dynamicComponentImport_)

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
- [Mix Syntax](https://www.notion.so/1061d464528f81d492f4e1b0f8c8adf4) → mix-syntax.md
- [Type system](https://www.notion.so/1061d464528f81c0ba7bdc8fc2ce8556) → mix-type.md
- [directives](https://www.notion.so/1061d464528f81c08c15e5ae817171ee) → mix-syntax.md (lexing directives #line/#push/#pop, _source_ redirect, ;; double semicolon, all pragmas: warningFilter/autoTypeAssertion/autoMakeStream/autoMakeLink/strictContainerTyping/strictTyping/withCells)
- [cells](https://www.notion.so/1061d464528f81b7afc2e999931a3c8a) → mix-syntax.md (var/normal/private cell details, visibility rule in defs, cellSelectNotNull/NotUnset, freeze)
- [cells-links-and-aliases](https://www.notion.so/1061d464528f81cba83fddeb2e80a3af) → mix-syntax.md (links conceptual model, module link params vs data params, shared link pattern, double-link alias syntax)
- [case-types](https://www.notion.so/1061d464528f81dcaf60cf82adc1d1e9) → mix-syntax.md + mix-type.md (private cases, data compatibility, union constraints, unset)
- [Builtin operations](https://www.notion.so/1061d464528f81a5a5bbff09ca40f2cf) → mix-syntax.md (operator semantics: null handling, +/-/*/% rules, == deep equality, ordering, datacase, debug)
- [undefined](https://www.notion.so/1061d464528f81c1bb53e27eb10a9aea) → mix-syntax.md (undefined semantics: bubbling, Kleene &&/||, filters, JSON serialization)
- [syntax-stmt-expr](https://www.notion.so/1061d464528f8150a392ddc30fbdbb9a) → mix-syntax.md (stmt vs expr rules, do blocks, appstate for cell mutation)
- [xml (representation)](https://www.notion.so/1061d464528f8123bbf7fd135d3e6c7f) → mix-lang-stream.md (tree representation, xmlns handling, namespace normalization)
- [styleguide](https://www.notion.so/1061d464528f819099d2e81ae669855d) → mix-syntax.md (indentation rules, bracket layout, function call formatting, comma rules, let sequences)
- [actionclosures](https://www.notion.so/1061d464528f818cb833fd55e9077e77) → mix-syntax.md (serialized format msg_view_invoke, chain of actions API, closures as agent entry points)
- [examples](https://www.notion.so/1061d464528f815fad1ed849ced729c1) → mix-lang-advanced.md (db.all vs db.head distinction, def redefinition scoping behavior)
- [escjson](https://www.notion.so/1061d464528f8110aae4ef61230e4783) → mix-lang-advanced.md (full tag reference: bin/undefined/nan/posinf/neginf/int/float/bool, blob generator, case format; canonical db ref = {tag}case:db.ref)
- [embedding-viewstacks](https://www.notion.so/1061d464528f81a7a3a0e65bf1f509f6) → mix-lang-advanced.md (secondary viewstack: embed/terminate/cellMessages, DOM stream filtering, co.consumeWithActions live cell pattern, embed/replace action template)
- [screen-rmx-init](https://www.notion.so/1061d464528f81fda16dc22e3f0ab4b0) → platform-builder.md (_rmx_init module: global app init on startup, p.reqName/reqParams, runs before _rmx_entry)
- [links](https://www.notion.so/1061d464528f8159bc12ea2ae03ecbad) → mix-lang-advanced.md (link doesn't create dependency, assign to link vars, inCellName, dependentLink, mapLinkArray, no links between views)
- [json-changes](https://www.notion.so/1061d464528f817aacb5db1a7d575617) → mix-lang-advanced.md ({tag} escape format, formatEscJSON/parseEscJSON, db refs, _escJSON_ notation, Track protocol escape behavior)
- [imperative](https://www.notion.so/1061d464528f81de9e27c7f475f0071a) → mix-lang-advanced.md (outdated page; effects-cannot-escape constraint: def with var cannot return function)
- [gettingstarted](https://www.notion.so/855fbfd7dc8140269ed3a30224acf692) → mix-lang-advanced.md (comments // and /* */); remix-tools.md (mix_client binary install + rlwrap usage)
- [recursion](https://www.notion.so/1061d464528f81a793f6f06a4c38da21) → mix-lang-advanced.md (mutual recursion, recurse/recurse2 combinators, cells cannot be recursive)
- [programming-with-streams](https://www.notion.so/1061d464528f8145b58dfe16d3c998c5) → mix-lang-advanced.md + mix-lang-stream.md (stream mutability, O(1) ++, recursive stream patterns)
- [pattern-matching](https://www.notion.so/1061d464528f8155b917f627971aafb9) → mix-lang-advanced.md (optional field `key?:v`, var not supported, no pattern in def args)
- [modules](https://www.notion.so/1061d464528f813c927efa13f30e9e65) → mix-lang-advanced.md (singleton static module, cellSessionScope/cellScreenScope, currentInstanceId, module type vars [A], REPL #load/#push/#input)
- [Libraries-and-executables](https://www.notion.so/1061d464528f81edb8f2f39d035c38f7) → mix-build.md (library/executable DB records, mixc CLI, dependency rules, lib_header, track API)
- [dom-sync-compress](https://www.notion.so/1061d464528f8112bf01fd7c1d32e2f6) → mix-lang-editor.md (sync cell keyword, $sync/$syncSwitch/$syncDrop protocol, symbol table encoding)
- [agent-sessionAgent](https://www.notion.so/1061d464528f817d9ae0db9dda6c6b0f) → mix-agent.md (_rmx_sessionAgent: compiler+VM session driver, MQTT protocol, msg_compile_runPhrases, idle action, issue format)
- [agent-agentAgent](https://www.notion.so/1061d464528f8183bb27d729af8358b4) → mix-agent.md (_rmx_agentAgent: on-demand agent VM spawner, used by sessionAgent localAgentsConfig)
- [agent-makeAgent](https://www.notion.so/1f11d464528f80d7b7fbfba55aecd426) → mix-agent.md (_rmx_makeAgent: build agent; targetLibs/targetExe/sources/database/filestore/deployMode/storeRemixFile; outputs: logs/warnings/errors/builtLibs/remixFile)
- [_rmx_auth](https://www.notion.so/1821d464528f802a877be36fb7b69fdb) → mix-screen.md (_rmx_auth screen protocol: reqName/reqParams/authRequest params, Trigger User Sign-In, navigate back)
- [screen-rmx-error](https://www.notion.so/1061d464528f813a889ae582452bbf64) → mix-screen.md (_rmx_error/_rmx_errorFallback: enable_error_page client var, message+details params, recovery patterns)
- [screen-rmx-entry](https://www.notion.so/1061d464528f813792f3c132856512a5) → mix-screen.md (_rmx_entry: external request interceptor, reqName/reqParams, entryRedirection since PR1404)
- [screen-rmx-debugShell](https://www.notion.so/1061d464528f81679c6bca4f882f19dd) → mix-screen.md (_rmx_debugShell: no-op debug session starter since PR1160)
- [parsing-csv](https://www.notion.so/1451d464528f80688a22ca276faf82d9) → mix-std-lib.md (extra core lib mixc/parsing: csv.parse string → array(array(string)), RFC-4180)
- [driver-compilerRule](https://www.notion.so/1061d464528f8131b06fcd053f6f0c4b) → mix-driver.md (compilerRule: rule/ruleContent/scanFilter types, scan/load/save/makeLibRule/saveLibRule/getExeRuleHash)
- [driver-compilerFile](https://www.notion.so/1061d464528f81239450ed3112cf974c) → mix-driver.md (compilerFile: file/fileContent/scanFilter types, scan/load/save/make via session)
- [driver-compilerLib](https://www.notion.so/1061d464528f8150a0ffc79f50b4fc16) → mix-driver.md (compilerLib: library/libraryInDB/libraryAtTopic, scan/load/save/strip/publish/closure)
- [driver-compilerExe](https://www.notion.so/1061d464528f816aabfae867d32f1519) → mix-driver.md (compilerExe: executable/executableInDB/executableAtTopic, scan/load/save/publish/createExeFromLibs)
- [driver-compilerDriver](https://www.notion.so/1061d464528f81c99495c44203b5ecec) → mix-driver.md (compilerDriver: options/config/session/res(T) types, createSession/compilePhrases/compileModules/imports/saveLibrary/saveExecutable/pubConfig)
- [driver-compilerREPL](https://www.notion.so/1061d464528f81f08bd0c528f7b07fde) → mix-driver.md (compilerREPL: sendCodeFromCompiler/eval — compile+send phrases to VM via connector)
- [driver-compilerClean](https://www.notion.so/1061d464528f81858188f7ae55868e2e) → mix-driver.md (compilerClean: stub, no API documented yet)
- [driver-compilerBuild](https://www.notion.so/1061d464528f81389251c1c46425fe2d) → mix-driver.md (compilerBuild: mixcOptions/buildRequest/buildProgress types, pipeline builder, execute/execOn/execEmbedImports)
- [driver-codegenDriver](https://www.notion.so/1571d464528f8012a46ee165203c279d) → mix-driver.md (codegenDriver: generate Mix source from editor screen records; scanScreens/loadScreen/generate)
- [driver-codegenUtil](https://www.notion.so/1571d464528f805cbcb8d017c9338563) → mix-driver.md (codegenUtil: mkStylesRecord/mkFileMetaRecord)
- [driver-remixFile](https://www.notion.so/1571d464528f806dadfecfaa18a004b6) → mix-driver.md (remixFile: build .remix zip; addBinary/addLibrary/addManifest/addRuntime/addAppMeta/pack/clean)
- [driver-remixRecipe](https://www.notion.so/1571d464528f8034a9a0fa0ad6f090ea) → mix-driver.md (remixRecipe: declarative .remix recipe types — bundle/assets/recipe/patch/fullModule; parameter substitution <<BUNDLE.NAME>>)
