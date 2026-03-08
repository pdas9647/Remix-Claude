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

## Recent PRs — Feb 22–28, 2026

25 PRs merged. High-signal marked ★.

### High-signal

| PR           | Summary                                                                                                               | Author     |
|--------------|-----------------------------------------------------------------------------------------------------------------------|------------|
| **#11726** ★ | **Simplified styles / dynamic themes** — themes dynamically resolved at L2 open; `rmx_style`/`rmx_styles` deprecated. | dprophete  |
| **#11760** ★ | **Remove .remix client action** — deprecated + removed (caused v1 .remix files, broken on Desktop).                   | simonh1000 |
| **#11759** ★ | **Custom Client Action node** — non-hard-coded client actions, replaces External Action node.                         | simonh1000 |
| **#11734** ★ | **Multiple Cloud Server locations** — builder stores multiple locations; Service Agent wizard.                        | simonh1000 |
| **#11710** ★ | **Desktop: REPL for Function node** — uses SessionPool (works on Desktop).                                            | simonh1000 |
| **#11757** ★ | **Rebuilding fixes** — detect missing consts via stdLibId; pass `didClean` as `deleteNonTargetBinaries`.              | simonh1000 |

### Minor

- #11746 — Restore "download package" | #11762 — L1 "New" to sidebar | #11768 — L1 delete warnings
- #11765 — Model rebuild errors | #11764 — Compiler tests | #11756 — Plugin search | #11753 — Plugins button
- #11758/#11754/#11755 — Minor updates, remove Reload Route, CSS revert
- #11669 — `deleteNonTargetBinaries` | #11744 — Service Agent field bindings refactor
- #11748 — `available` out param | #11743 — Refresh modal | #11742 — `InActive` → `Inactive`
- #11741 — Debug msg | #11740 — Open link in new tab | #11738 — Remove Card derived state

## Recent PRs — Mar 1–7, 2026

22 PRs merged. High-signal marked ★.

### High-signal

| PR           | Summary                                                                                                                                                                                                       | Author     |
|--------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|------------|
| **#11774** ★ | **Table primitives** (6 commits, 9 files, +542). New DOM/card primitives for `<table>/<thead>/<tbody>/<tfoot>/<tr>/<th>/<td>`. Context-aware wrapping rules in button bar. Inline ungroup button.             | tlentz     |
| **#11792** ★ | **Table primitives — new feature flag** — registers in NewFeatures.elm.                                                                                                                                       | tlentz     |
| **#11724** ★ | **BlobGet Action** (37 commits, 10 files, +2198). Builder support for `binary.blobGet` — reads `LocalFile` into spreadsheet via client VM. Requires protoquery#2226.                                          | simonh1000 |
| **#11751** ★ | **Refactor Builder tab/window opening** (18 commits, 26 files). **Breaking:** `remix/location-changed` now passes `{path: string}` not `string`. New `remix/builder_open` + `remix/runtime_open` for Desktop. | simonh1000 |
| **#11750** ★ | **CodegenLib supports catalog items** (+548). New `generate2` API accepting catalog item clipboard values.                                                                                                    | simonh1000 |
| **#11788** ★ | **Namespaced user prefs** — `builder.` prefix for builder prefs. Prep for mobile/desktop/extension sharing prefs namespace.                                                                                   | dprophete  |
| **#11786** ★ | **Paste replace modal** — L1 paste shows "Keep Both / Replace" dialog (macOS Finder style) instead of always creating copies.                                                                                 | dprophete  |

### Features & fixes

- #11790 — Copy URL action | #11787 — Small UI tweaks (dprophete)
- #11785 — Fix make after rename | #11784 — Api node handles `remix://` URIs via `Uri.parse`
- #11783 — LocalFile improvements to L2 preview | #11782 — Paste curl support
- #11781 — DnD uses window event handler — fixes erratic L2 DnD, especially Desktop
- #11778 — Prevent L1 navigation while makeAgent running
- #11777 — StdLibIds in Desktop dev environment | #11775 — Misc fixes
- #11776 — DRYer JS host effects code (refactor, -165/+65 lines)
- #11773 — Remove ancient upload support | #11770 — Compilation fix
- #11771 — Fix env inconsistency between builder and runtime
- #11769 — Skip L1 Stage1 codegen for Desktop — removes unnecessary extra codegen round for local DB
