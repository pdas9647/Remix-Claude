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

## Recent PRs — Apr 18–25, 2026 (21 merged)

★ High-signal marked.

| PR           | Summary                                                                                                                                                                                                                                                                                                  | Author     |
|--------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|------------|
| **#11677** ★ | **State nodes** (12 commits, 9 files, +165/-91) — Long-lived (Feb 4→Apr 21). State data section is clickable to select a field/path; bound values marked as such in the UI.                                                                                                                              | simonh1000 |
| **#11938** ★ | **Client Actions in the Runtime** (24 commits, 14 files, +122/-129) — `ClientAction` runtime type gets `expectResultType` field (false for External Actions). Codegen uses `msg_perform_host_effect` for External, `msg_perform_client_action` for Client. Completes client action pipeline unification. | simonh1000 |
| **#11965** ★ | **Desktop builder shows channel** (4 commits, 6 files, +81/-11) — Builder UI now displays its release channel, read via existing client action.                                                                                                                                                          | simonh1000 |
| **#11955**   | **EAction takes a list of exprs** (4 commits, 13 files, +257/-275) — Action codegen always wraps in `{ }` blocks; prevents accidental codegen bugs.                                                                                                                                                      | simonh1000 |
| **#11963**   | **Runtime error cleanup** (9 commits, 7 files, +78/-108) — Better fatal error type names; `OnInitDataForVmServer` no longer handled individually in Runtime.                                                                                                                                             | simonh1000 |

### Fixes & minor

- #11966 ensure content binding on local file [bug] | #11964 QB2 init handles wizard-in-use case | #11962 explicit amp runtime
- #11961 CreateApp macro from L1 & L2 | #11960 rcm update | #11959 minor fixes | #11958 compiler error fix
- #11956 separate out shared code | #11954 better modelling of flags for remix.app | #11953 files /public folder
- #11948 builder emitted as ES6 | #11949 merge query params while using flags | #11947 minor fixes
- #11952 macro codegen bugfix | #11951 restore webpack ws proxy | #11946 remove hard-coded dev file paths
