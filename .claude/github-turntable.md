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

## PR Archive — Mar 1–21, 2026

★ Key: #11774/#11792 table primitives; #11724 BlobGet action; #11751 tab/window opening refactor (breaking: location-changed payload); #11750 CodegenLib catalog items; #11788 namespaced user prefs;
#11786 paste replace modal; #11789 webcomp methods with args; #11791 File node content binding; #11794 catalog fetch perf (O(n²)→O(n)); #11539 OPFS storage backend (major); #11834 data-rmx-cid
anchors; #11807 pub-sub modelling + Cloud Sub UI; #11815 screenshot overhaul (html-to-image); #11820+#11823 callable build fixes; #11813 L3 QB codegen standardization.

## Recent PRs — Mar 22–Apr 3, 2026 (55 merged)

★ High-signal marked.

| PR           | Summary                                                                                                                                                                                                       | Author                                                   |
|--------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|----------------------------------------------------------|
| **#11829** ★ | **Remote DBs in synced prefs** (23 commits, 31 files, +440/-363) — `remoteSources` moved from per-app `CloudDbMode` into `SyncedPrefs` (synced per-workspace per-user). `CloudDbMode` simplified to `Off      | On{selected}`. Remote sources now in Edit menu at L1/L2. | dprophete  |
| **#11836** ★ | **Decode .remix v2.1 manifests** — front-end support for protoquery build server v2.1 format.                                                                                                                 | simonh1000                                               |
| **#11837** ★ | **Desktop: download file** (3 files, +27/-2) — client action `remix/download-file` for Tauri; `<a download>` unsupported in Tauri. Pairs with mix-rs#1071.                                                    | simonh1000                                               |
| **#11850** ★ | **IDB→OPFS migration in worker + disable** (15 commits, 3 files) — migration moved to mixcore worker using `createSyncAccessHandle`; migration disabled due to macOS/Safari `createWritable` incompatibility. | benozol                                                  |
| **#11858** ★ | **RmxMode simplification** — removed `Run` from `RmxMode`; L2 mode switcher moved from navbar dropdown → Edit menu. Configure/artifact-viewer mode shows single back-to-edit button.                          | dprophete                                                |
| **#11869** ★ | **File Register returns full `file`** (11 commits, 7 files, +181/-41) — breaking change: output binding now exposes entire file object (`kind: client`) instead of a specific field.                          | simonh1000                                               |
| **#11873** ★ | **screenName param in AppRecord** — `.remix` files' `screenName` param now respected to set start page.                                                                                                       | simonh1000                                               |
| **#11890** ★ | **Extract out bindings** (4 commits, 5 files, +209/-95) — binding editor now supports extracting out bindings (was in-bindings only). No value initialization needed; data flows naturally from source.       | dprophete                                                |
| **#11685** ★ | **Pretty Comp in-params** — long-running PR (Feb 5) merged: improved component in-param display.                                                                                                              | simonh1000                                               |

### Features & fixes

- #11833 — Fix mixcore webcomp (OPFS follow-up for web components after #11539)
- #11832 — LocalFile node content binding (complement to #11791)
- #11841 — `switchApp` terminates machine (VM lifecycle leak fix)
- #11853 — Use new query syntax for sorting (QB 2.0 sort codegen)
- #11831 — Stdlib .dox in Studio (pairs with protoquery#2100)
- #11856 — Builder with mixer uses `routeInfo` in flags (plugin params fix)
- #11867 — L2 echoes for subscriptions
- #11877 — Disable URL updating from custom page with AppRecord
- #11879 — AMP `index.html` updates
- #11880 — Redirect to sign-in on anon token

### Minor / UI

- #11830 — Paste at L1 adds you as collaborator | #11839 — baseflags/loadflags fix
- #11843 — Always ask before navigating away | #11844 — Rename Debug data
- #11845 — Preview: more consistent UI | #11846 — Model fatal errors better
- #11847 — Remove `id` deprecation warning | #11848 — Remove UnrecognisedAction
- #11849 — State machine mouseover fix | #11851 — Refresh bindings fix
- #11852 — Remote data name | #11855 — L3 function session pool fix
- #11857 — Small UI fixes | #11859 — TUI CSS change | #11860 — Service agent fix
- #11861 — Close VM when closing L2 | #11862 — No double-fetch blob URLs
- #11863 — remix-required.css flex-shrink fix | #11864 — Desktop artifact viewer fix
- #11865 — WebApp documentation | #11870 — Datatree fx codegen
- #11871 — Don't pass selector to toastify | #11874 — Fix isDisabled on cloned mod
- #11875 — Small UI tweaks | #11878 — Fix mixcore.wasm references (OPFS)
- #11881 — file-plus icon for File Register | #11882 — More tile descriptions
- #11883 — L1 macros refactoring | #11884 — Runtime error not reset by cell data
- #11885+#11887 — EDo/EInitialize list (merged then reverted) | #11888 — L2 node sizing fixes
- #11889 — Fix toasts in builder/plugins | #11895 — Group node + L3 usage width fixes
- #11797 — Renamed collapsed nodes (long-running, finally merged Apr 1)
