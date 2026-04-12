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

## PR Archive — Mar 1–Apr 3, 2026

★ Key: table primitives; BlobGet action; tab/window opening refactor (breaking: location-changed payload); CodegenLib catalog; namespaced user prefs; paste replace modal; webcomp methods with args;
File node content binding; catalog fetch O(n²)→O(n); OPFS backend; data-rmx-cid anchors; pub-sub + Cloud Sub UI; screenshot overhaul; callable build fixes; L3 QB codegen; remote DBs into SyncedPrefs (
CloudDbMode refactored) ★; .remix v2.1 manifests; Desktop download file (+ mix-rs#1071) ★; IDB→OPFS migration in worker; RmxMode simplified (Run variant removed); File Register returns full file ★;
screenName in AppRecord; extract out bindings ★; viewstack embed node ★ (Apr 2, long-running).

## Recent PRs — Apr 5–10, 2026 (18 merged)

★ High-signal marked.

| PR           | Summary                                                                                                                                                                                                  | Author     |
|--------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|------------|
| **#11886** ★ | **Viewstack embed** (18 commits, 17 files, +684/-422) — New `Embed` node type implementing Gerd's viewstack embedding. Will include triggerable runner in due course.                                    | simonh1000 |
| **#11894** ★ | **Simplified settings** (13 commits, 20 files, -267 net) — Settings editing moved from L2 → L1 modal. Removed 'transition page' card (never used). Simplifies L2 save code and navbar.                   | dprophete  |
| **#11893** ★ | **L1 rebuild+make macro** (4 commits, 25 files, +290/-173) — New macro `RebuildAndMake` invokable by plugins (e.g. Gerd's tooling plugin). Threads continuation action through async make pipeline.      | simonh1000 |
| **#11905** ★ | **Remove Mixer.elm** (7 commits, 31 files, +161/-216) — All mixer alias types replaced by `{mixerHost:String, workspace:String}`. Removes dedicated `Mixer.elm` module; consolidation refactor.          | simonh1000 |
| **#11910** ★ | **Event overrides apply to Elm-handled client actions** (3 commits, 12 files, +132/-90) — Host-provided client action overrides now take precedence even for actions previously decoded directly in Elm. | simonh1000 |
| **#11907** ★ | **FocusElement & CloseApp → client actions** (8 commits, 11 files, +140/-161) — Both actions removed from hardcoded Elm path and routed through client action pipeline. Pairs with #11910.               | simonh1000 |
| **#11899** ★ | **Flip between list and loop** (6 commits, 7 files, +2172/-73) — Binding editor: list↔loop flip now supported. Cards special-cased to bind into list container. Large diff primarily from new tests.     | dprophete  |

### Fixes & minor

- #11891 — Better L2 error handling (-430 net, dead code removal + renamings)
- #11892 — EDo/EInitialize take a list (re-land after Apr 2 revert)
- #11898 — Fixed webcomp font issues
- #11901+#11902+#11903 — File node tests + bugfix
- #11904 — Clean up runtime code (refactor) | #11906 — Remove passthrough RuntimeAction
- #11908 — Model `SwitchApp_` | #11913 — Fix compile error
- #11653 — Lodash security bump (dependabot, `builder/rmx-files`)
