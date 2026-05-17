# turntable

**Repository:** https://github.com/remixlabs/turntable
**Stack:** Elm 0.19.1, TypeScript, JavaScript, Webpack (builder), Vite (webapp)
**Purpose:** Remix Studio frontend — visual node-graph IDE (builder), multi-platform runtime (webapp), 8 web components
**Explored:** L1–L3 full (~100+ files)

## Build & Deploy

- **Monorepo:** `builder/`, `webapp/`, `shared/`, `web-components/*` (8), `scripts/`, `snowflake/`
- **RCM deps:** `amp`, `protoquery`/`protoquery-wasm`, `groovebox`, `mixcore`
- Builder: Webpack (:3000). Webapp: Vite (:3002). `make artifacts` → GCS.
- **CI:** CircleCI — branch=debug, main=optimized → `upload_assets` → `update-users`

## Architecture

**Pages:** L0 (workspace/app listing), L1 (screen management + make pipeline), L2 (node graph editor, 150+ Msgs, sessionPool/undoStack/compStack), L3 (data editing: card/query/decision/FSM)

**Dual VM:** VmServer (AMP, default web) | VmClient (Wasm, Desktop/Snowflake via protoquery-wasm)

**Port interop:** Builder ports (clipboard, file drops, client actions, node sizes); Runtime ports (MQTT, host actions, web components); VM ports (REPL MQTT/Mixc)

**Entry points:** `builder/src/builder.js → Main.elm` | `webapp/src/rmx-fullscreen.js → App.elm` | `rmx-init-runtime.ts` | `rmx-app-record.ts`

## Data Model

- **Node**: key, displayName, `persistedState` (type), `bindings: Dict String Binding`, originalData
- **PersistedState** (12+ types): Card, QB(1|2), Data, Decision, FSM, Action, Comp, Function, WebComp, Wasm, File, Group
- **Binding**: key, `type_: BuilderType`, direction(In|Out), `connections: Set BindingKey`, defaultValue. Special: `_rmx_response`, `_rmx_error`, `_rmx_on_load`, `_rmx_action_invoke`
- **Module**: `ModuleFull` (Nodes dict) vs `ModuleLight` (L1). Types: Screen|Symbol|Code|LocalAgent|ServiceAgent|Constants|*Plugin
- **BuilderType**: MxString|MxNumber|MxBool|MxColor|MxUrl|MxObject|MxList|MxResult|MxData. Comparison: Strong|Weak|Diff
- **AppMeta**: dbName, appName, settings, styles, appType(App|Theme|AppExecutable), appState(Normal|Trash|Deletable), accessLevel(Run|Edit)
- **TVal** = `RVal Never` (immutable JSON). `RVal a`: TString|TInt|TFloat|TBool|TNull|TList|TObj + TBinding|TConst|TProj|TMix

## Node Types

- **Card** (~15K lines): DOM AST editor (Leaf/Container/Dynamic). 3-panel L3 (preview|tree|props). Codegen ~4300 lines. L2↔L3 via pointer injection.
- **Action** (~12K lines): NavigateAction|PushOverlayAction|PopAction|PubSubAction|AgentAction(Local/Service/Amp)|ApiAction|ClientAction|CustomAction
- **Decision**: Array of Dispatch (predicate + return) + default fallback. Return type switchable.
- **FSM**: States + transitions + GraphViz DOT viz. Per-state output bindings, per-event triggers.
- **Component**: CompStd (headless), CompCallable (agent-style), CompTriggered (event-based)
- **QueryBuilder**: OnLoad|LiveCurrent|LiveAll. DefnStandard|DefnQueryAst|DefnQueryString

## Canvas, Macros & Backend

- **GraphUI**: zoom, drag (Binding|Nodes|Magnifier), selection. bindingWidth=20px, gridSnap=30px. Binding classification: Card|Event|Data.
- **Macro System**: 40+ ops (CreateNode, DeleteNode, CloneNode, AddBinding, AutoConnect, MakeAgent, RunPlugin, SimulateClientAction…). MacroSource: Plugin|DotRemix|Builder.
- **MakeAgent**: Build via MQTT → _rmx_makeAgent → logs/warnings/errors/builtLibs/builtExe. MakeOptions: targetLibs, targetExe, variant(Debug|Release), clean.
- **SessionAgent**: L2/L3 REPL (`$$-l2-$$`, `##-l3-##`). Config: compilerEnabled, database, localAgentConfig.
- **AstQueries**: CRUD for AppMeta/Module/Node/libs. Abstracts Amp vs Mixer backends.

## Web Components

rmx-base (Elm), rmx-editor (Elm), rmx-remix (JS runtime), rmx-runtime (JS executor), rmx-monaco (TS/Lit, Mix syntax), rmx-auth-google (Elm), rmx-file-input (Elm), rmx-graph-viz (JS)

## Open Questions

- Migrations: M_018–M_020 not read. Tests: ~100+ Elm + ~210 JSON fixtures not explored. Renderer: `shared/Renderer/` not read.

## PR Archive — Mar 1–Apr 17, 2026

★ Key Mar 1–Apr 3: table primitives; BlobGet; tab/window refactor (breaking: location-changed payload); CodegenLib catalog; namespaced prefs; webcomp method args; File node content binding; catalog O(
n²)→O(n); OPFS backend; pub-sub + Cloud Sub UI; screenshot overhaul; remote DBs→SyncedPrefs ★; .remix v2.1; Desktop download file ★; IDB→OPFS worker migration; File Register returns full file ★;
viewstack embed node ★.
★ Key Apr 5–10 (#11886 viewstack embed; #11894 settings L2→L1 modal; #11893 RebuildAndMake macro; #11905+#11907+#11910 Mixer.elm removed + client action pipeline; #11899 list↔loop flip; #11900 File
node db/path→binding defaults ★).
★ Key Apr 13–17 (#11921 pub/sub subscribe via client action codegen ★; #11915 External+Client runtime actions unified ★; #11923 webapp JS→TypeScript + rmx-init-runtime merged ★; #11929
CreateFromExternal accepts Webcomp+Wasm manifests; #11928 invoke+eval JS via client action ★).

## PR Archive — Apr 18 – May 1, 2026

★ Apr 18–25: #11677 State nodes — data clickable to select field/path, bound values marked (long-lived Feb→Apr 21); #11938 Client Actions in Runtime — `ClientAction.expectResultType`; External→`msg_perform_host_effect`, Client→`msg_perform_client_action`; #11965 Desktop builder shows release channel; #11955 EAction takes list of exprs; #11963 runtime error cleanup. Also fixes: #11966, #11964, #11962, #11961, #11960, #11959, #11958, #11956, #11954, #11953, #11948, #11949, #11947, #11952, #11951, #11946.

★ Apr 26–May 1: #11980 Runtime as embeddable JS library — `embedRuntime(uid, flags)`/`embedAppRecord`/`embedRemixUrl` mount Elm app into DOM node; new `Embed.elm` (Vibecode layer); #11897 consolidate mixcore js — `ClientVars`→`EnvClient` rename across 30+ files, `handleAuthReturn` cleans URL, `VmRegister.alive` ignores stale MQTT, web-comp double-registration guard (long-lived, benozol); #11979 `_rmx_auth` rewrite — dynamic signin URL via `get_appclip`, `msg_perform_client_action` for rmx-remix, appclip surface detection, `ModeAmp` supports Mixer DB via URL params (dprophete); #11971 plugin system — `mapRuntime` unifies PanePluginRunning+PaneDotRemixPlugin, `createRmxAuthIfMissing`, `ReloadAppMetas` macro, `list_projects_l0`, rebuild detection via `rmx_type=file` records; #11978 `ModeAmp` supports Mixer DB; #11972 web comp race condition — `createOrUpdate` unified. Also: #11983 perform-client-action moved to standalone export; #11982 rmx-auth followup; #11976 QB2 fixes; #11977 remove AppClip surface override; #11981 simplify rmx-init-runtime; #11974 token saving fix; #11973 unhappy path; #11967 authParams handling; #11968 amp URL rewriting; #11970 gitignore.

## Recent PRs — May 2–16, 2026 (32 merged)

★ Key: #12030 port turntable to rcm2 (Gerd May 16); #11988 unify `Amp`+`Mixer` → single `DbBacked` variant in `VmClientAssetsLocation` (Simon May 13); #12013 new Pagination node (Simon May 11); #12017 remove PluginAmp (Simon May 12); #12004 remove localFFIs (Simon May 8); #12000 rmx-remix React wrapper (DDA May 7).

| PR           | Date   | Summary                                                                                                                                                                                                                                                                                                                                                                                                                  | Author     |
|--------------|--------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|------------|
| **#12030** ★ | May 16 | **Port turntable to rcm2** — `rcm.config.mjs` → `rcm.config.json`. CI drops `rcm script authorize-gcloud` calls; `update-users` → `users update`; contexts `rmx-prod-uploads` → `rcm-editor`. `upload_web` uses `rcm gcs cp --compress` per-file. | gerdstolpmann |
| **#11988** ★ | May 13 | **Modelling Db-backed apps in runtime** — Unifies `Amp` + `Mixer` variants of `VmClientAssetsLocation` into single `DbBacked { dbPrefix, assetsPrefix, mixcore }`. Branch moves to `ModeAmp` itself. Major Runtime refactor preparing for post-AMP. | simonh1000 |
| **#12013** ★ | May 11 | **Pagination node** — New node type for paginating data sources; modifier-style (re-triggers Data Fetch). [Enhancement]                                                                                                                                                                                                                                                                                                  | simonh1000 |
| **#12017** ★ | May 12 | **Remove PluginAmp** — Drops the AMP-specific plugin variant; unified plugin codepath now.                                                                                                                                                                                                                                                                                                                              | simonh1000 |
| **#12004** ★ | May 8  | **Remove localFFIs** — Drops the `localFFIs` config from `StartWASM2`/groovebox starter. Local FFIs no longer needed since client actions handle the same role.                                                                                                                                                                                                                                                          | simonh1000 |
| **#12000** ★ | May 7  | **rmx-remix React wrapper** — React wrapper component for `rmx-remix` web component; enables embedding Remix apps in React-based hosts.                                                                                                                                                                                                                                                                                  | dprophete  |
| #11993       | May 13 | **Live binding values for plugins** — Plugin VMs receive live-updating binding values. | simonh1000 |
| #12027       | May 15 | **mixcore debug + playwright** — Threads `debug` flag through `MixcoreConfig` (paired w/ mix-rs#1120); Playwright scaffolding. | benozol    |
| #12015       | May 11 | **db-key + db-key-suffix for webcomp** — New webcomp attrs for DB targeting. | dprophete  |
| #11984       | May 4  | **RRL accepts JSON params + constants** — Remix Runtime Library API enhancement. | simonh1000 |
| #11997, #12003, #11998, #11987 | May 5–7 | **Misc enhancements** — `assetPrefix` env var [Enh.], advanced QB2 [Enh.], better L2 JSON editing, remove obsolete `authCallbackPrefix`. | simonh1000 |

### Fixes & minor

- **rmx-remix / webcomp**: #12014 null pointer, #12010 expose uid, #12009 DB prefixes, #12012 rmx-img CSS, #12005 img fallback URL, #12001 full CSS reset, #12008 legacy img URL removed
- **Auth/runtime**: #12016 token fix, #11990 graceful bad-`.remix` URL, #11991 codegen bound main in-binding of out-params, #11986 selected anon/auth nodes, #11989 preview perf
- **Editor**: #12023 close VM on .remix preview nav, #12020 moustache parser edge case, #12007 group REPL TrackResponses, #12006 clean uids from clipboard, #12026 rename clipboard type, #12019 remove paste special, #11996 minor UI, #11995 root node fix, #11994 renaming, #12002 Transform:filter codegen uses Expr