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

| PR           | Summary                                                                                                                                                                                                                                                                                                  | Author     |
|--------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|------------|
| **#11726** ★ | **Simplified styles / dynamic themes** (29 commits, 16 files). Themes now dynamically resolved at L2 open — no more make/publish step. `rmx_style`/`rmx_styles` no longer auto-included; deprecated warning shown if referenced. Changes to theme B instantly reflected in apps using it. Closes #11716. | dprophete  |
| **#11760** ★ | **Remove .remix client action** — deprecated + implementation removed (caused v1 .remix files, broken on Desktop). Not yet removed from primitives to avoid broken bindings.                                                                                                                             | simonh1000 |
| **#11759** ★ | **Custom Client Action node** — non-hard-coded client actions, replaces old External Action node; enforces error bindings.                                                                                                                                                                               | simonh1000 |
| **#11734** ★ | **Multiple Cloud Server locations** (25 commits, 24 files). Builder stores multiple cloud server locations; Service Agent wizard for new node setup; existing Service Agents can save to Cloud Locations library.                                                                                        | simonh1000 |
| **#11710** ★ | **Desktop: REPL for Function node** — uses SessionPool (works on Desktop). Session stays open during REPL; closed on switch to code editor, restarted on return.                                                                                                                                         | simonh1000 |
| **#11757** ★ | **Rebuilding fixes** (27 commits, 12 files). Detect missing consts via stdLibId; recompile code mod on close; pass `didClean` as `deleteNonTargetBinaries`; replace `[*]` with explicit app list in executable.                                                                                          | simonh1000 |

### Minor fixes & polish

- #11746 — Restore "download package" from L0 menu (lost in .remix v2 switch)
- #11762 — L1 "New" button moved to sidebar
- #11768 — L1 delete warnings | #11765 — Model rebuild errors
- #11756 — Proper plugin search | #11753 — Plugins button
- #11764 — Compiler tests compile | #11761 — L2 preview updates
- #11758 — Minor updates | #11754 — Remove Reload Route | #11755 — CSS revert
- #11669 — Implement `deleteNonTargetBinaries`
- #11744 — Service Agent: field bindings refactor
- #11748 — Add `available` out param (benozol)
- #11743 — Design-requested refresh modal | #11742 — `InActive` → `Inactive`
- #11741 — Debug message update | #11740 — Open link in new tab
- #11738 — Remove Card model derived state (tech debt)