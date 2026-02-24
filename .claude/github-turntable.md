# turntable

**Repository:** https://github.com/remixlabs/turntable
**Stack:** Elm 0.19.1 (primary), TypeScript, JavaScript, Webpack (builder), Vite (webapp), SCSS, CircleCI, npm workspaces, RCM
**Purpose:** Remix Studio frontend — visual node-graph IDE (builder), multi-platform runtime (webapp), and 8 web components
**Explored layers:** L1–L3 full (Tiers A/B/C). ~100+ files read across all layers.

## Build & Deploy

**Monorepo workspaces:** `builder/`, `webapp/`, `shared/`, `web-components/*` (8 packages), `scripts/`, `snowflake/`

**Prerequisites:** Node.js ^22.16.0, npm ^10.9.2, RCM (Remix Component Manager)

**RCM dependencies** (binary/Wasm assets shared across turntable, desktop, snowflake):

- `amp` — server backend, `protoquery` / `protoquery-wasm` — Mix compiler (OCaml)
- `groovebox` — runtime engine, `mixcore` — Rust FFI bridge (from mix-rs)

**Build commands:**

- `make setup` → rcm-install + npm up
- Builder: `npm run dev` (:3000 Webpack), `npm run prod` (production)
- Webapp: `npm run dev` (:3002 Vite), `npm run build-all` (production + lib builds)
- `make artifacts` → clean production build → `make upload` → GCS (rmx-static/assets)

**CI (CircleCI):** `linux_exe` + `elm_exe` executors. Branch=debug builds; Main=optimized → `upload_assets` → `update-users`.

**Multi-target:** remix.app, Desktop (Tauri/mix-rs), Snowflake (SPCS), local amp. Copy scripts per target.

## Architecture

### TEA Update Chain (The Elm Architecture)

```
Browser/Port Event → Main.update → UpdateRouter → Page-specific update
  Pages: Loading | PageSignIn | PageL0 | PageL1 | PageL2 | PageL3
         | PageCodeModEditor | PagePreview | Migration | FatalError
```

- **L0** `/e/edit` — Workspace/app listing. Model: appMetas, search, filters (App|DesignLib|Theme|AppExecutable|Trash), rightPane (UserPrefs|Plugins). CRUD: create/trash/delete/restore apps. Plugin
  hosting via Runtime.
- **L1** `/e/edit/{dbName}` — App/screen management. Model: modulesLight (Dict), modPositions, guiModel (graph canvas), makeAgent, insertState. CRUD: create/delete/clone modules. MakeAgent build
  pipeline (MQTT → _rmx_makeAgent). Copy/paste modules. File tree management.
- **L2** `/e/edit/{dbName}/{modName}` — Node graph editor (core). Model: nodes, guiModel, sessionPool, l3BeingEdited, rightPane (8 modes), undoStack, compStack. 150+ Msg variants. Init phases:
  ReplAndDbRequests → ReplReturned → DbData → Complete.
- **L3** `/e/edit_l3/{dbName}/{modName}/{nodeKey}` — Data editing (card, query, decision, FSM)

### Dual VM Architecture

- **VmServer (AMP):** Server-backed compilation. Default for web.
- **VmClient (Wasm):** Client-side via protoquery-wasm. Desktop/Snowflake.
- **MixcStatus:** `UsingAmp` → `AwaitingStaticAgents` → `HaveStaticAgents`

### Port Interop (Elm <-> JS)

Bidirectional tag-based JSON through Elm ports:

- **Builder ports** (`toJsBuilder`/`fromJsBuilder`): clipboard, screenshots, file drops, URL nav, client actions, node sizes, user prefs
- **Runtime ports** (`toJsRuntime`/`fromJsRuntime`): machine lifecycle, MQTT, host/client actions, tracking, web components, geolocation
- **VM ports** (`builderVmToJs`/`builderVmFromJs`): MQTT/Mixc agent communication for builder REPL

### Entry Points

| Entry        | File                                       | Purpose                             |
|--------------|--------------------------------------------|-------------------------------------|
| Builder      | `builder/src/builder.js` → `Main.elm`      | Builder IDE (Webpack)               |
| Fullscreen   | `webapp/src/rmx-fullscreen.js` → `App.elm` | AMP/Snowflake runtime               |
| Init Runtime | `webapp/src/rmx-init-runtime.ts`           | remix.app/Desktop/Snowflake wrapper |
| App Record   | `webapp/src/rmx-app-record.ts`             | .remix file loader (DnD, appclip)   |

**Webapp App Modes:** `ModeAmp` (server, token auth), `ModeMixer` (desktop/offline), `ModeMixcoreFlags` (.remix file loading)

## Data Model

### Core Types (`builder/src/Data/`)

**Node** (~1100 lines): `key`, `displayName`, `persistedState` (determines type), `bindings` (Dict String Binding), `originalData` (DBRef, parentRef, screenRef, MD5).

**PersistedState** variants (12+ node types): `CardState` (UI), `QBState (QB1|QB2)` (query), `DataState` (CRUD), `DecisionState` (if-then), `FSMState` (state machine), `ActionState` (agent/HTTP),
`CompState` (component with nested Nodes), `FunctionState`, `WebCompState`, `WasmState`, `FileState`, `GroupState`.

**Bindings** (~800 lines): `Binding` = { key: (nodeKey, bdTag), type_: BuilderType, direction: InBinding|OutBinding, connections: Set BindingKey, defaultValue: TVal }. Special tags: `_rmx_response`,
`_rmx_error`, `_rmx_on_load`, `_rmx_action_invoke`, `_content_`.

**Module** (~700 lines): `ModuleFull` (with Nodes dict) vs `ModuleLight` (L1 canvas). ModuleType: Screen|Symbol|Code|LocalAgent|ServiceAgent|Constants|DashboardPlugin|ProjectPlugin|ScreenPlugin|etc.

**BuilderType** (~900 lines): `MxString|MxNumber|MxBool|MxColor|MxUrl|...`, `MxObject (Dict String BuilderType)`, `MxList`, `MxResult`, `MxData` (wildcard), `MxOther String`. Type comparison:
BTC_Strong|BTC_Weak|BTC_Diff.

**AppMeta** (~830 lines): `dbName`, `appName`, `settings` (RuntimeSettings), `styles` (Styles), `appType` (App|Theme|AppExecutable), `appState` (Normal|Trash|Deletable), `collaborators`, `errors`,
`accessLevel` (Run|Edit).

### Value System (`shared/Data/TVal/`)

- **TVal** = `RVal Never` — immutable JSON-like data (no builder constructs)
- **RVal a** — polymorphic: TString|TInt|TFloat|TBool|TNull|TList|TObj + builder-only: TBinding a|TConst|TProj|TMix. When `a=String`, carries binding references for persistence.

## Node Types (L3 Editors)

### Card (`builder/src/Card/`, ~15K lines total)

Tree-centric DOM AST editor. `DomType` union: EmptyTree | DN DomNode. DomNode variants: Leaf, LeafDynamic, ContainerStatic, ContainerDynamic, SvgLeaf. Props = `Dict String (RVal String)`, Styles =
`Dict String (RVal String, StyleUnit)`.

**L3 editing:** 3-panel (preview | tree | properties). Undo via full tree snapshots. Pointer injection for inline Mix codegen (dom_number, dom_date, etc.). L2↔L3 roundtrip: `l2ToL3` (inject
pointers) / `l3ToL2` (extract, reconcile bindings). Codegen (~4300 lines) generates Mix dom calls per primitive.

### Action (`builder/src/Action/`, ~12K lines total)

Form-centric editor. ActionType union: NavigateAction|PushOverlayAction|PopAction|PubSubAction|AgentAction|ApiAction|ClientAction|CustomAction.

**AgentAction:** LocalAgent (mixer sidecar) | ServiceAgent (Cloud DB, wizard-based setup) | AmpAgent. Bindings: invoke (in), next (out), agent/db/token (in), Out (out).

**ApiAction:** URL template + method + token + params + headers + expect (JSON|String|Empty). Path param extraction from URL. Validation checks all template params bound. Codegen generates HTTP calls
with binding resolution.

### Decision (`builder/src/Decision/`, ~1160 lines)

Conditional dispatcher. Array of `Dispatch` structs each with predicate + return value. Default dispatch fallback. Return type switchable (MxEvent ↔ data types). In-loop optimization: count-based
results.

### FSM (`builder/src/FSM/`, ~560 lines)

State machine. States with entry/exit events. Transitions map event → target state + action flag. GraphViz DOT visualization. Bindings per-state (output) and per-event (trigger). JSON persistence.

### Component (`builder/src/Component/`)

Three subtypes: CompStd (headless, content binding), CompCallable (agent-style, dataOutKey), CompTriggered (event-based, dataOutKey).

### QueryBuilder (`builder/src/QueryBuilder/`)

MainFilter (entity/rmxType/unused/record). QueryType: OnLoad | LiveCurrent | LiveAll. QueryDefinition: DefnStandard | DefnQueryAst | DefnQueryString.

## GraphUI & Canvas (`builder/src/GraphUI/`)

**Model** (~420 lines): zoom (level + center), drag state (DGBinding|DGNodes|DGMagnifier), selection, hover, grid. Constants: bindingWidth=20px, bindingHeight=15px, gridSnap=30px,
collapsedNodeWidth=150px.

**Rendering** (~2800+ lines): Binding classification (Card|Event|Data) with CSS classes. Collapsed groups: inner bindings materialized on group node via `wrapCollapsedGroupBdKey`. Visual states:
is-bound, is-state, is-set, is-active-source/target, is-weak, is-proj.

## Macro System (`builder/src/Macro/`)

40+ macro variants (declarative graph operations): CreateNode, DeleteNode, MoveNode, CloneNode, AddBinding, DeleteBinding, AddConnection, AutoConnect, SetDefaultValue, GetDefaultValue,
ConvertToComponent, GroupNodes, Copy, Paste, NodeSearch, ScreenSearch, ArtifactSave, MakeAgent, GetCodegen, RunPlugin, SimulateClientAction, etc.

**Runner** (~1500 lines): Executes macros sequentially, stops on error. MacroSource: SrcPlugin|SrcDotRemix|SrcBuilder|SrcRunner. 11 menu primitives (Highlight, Search, Save, Make, etc.).

## Backend & Agents

**AstQueries** (~500 lines): CRUD for AppMeta, Module, Node, compiled libs. Abstracts Amp (GET query) vs Mixer (POST query) backends. AST version tracking for migrations.

**BuilderQueries** (~150 lines): RmxAsset CRUD via catalog agent. `runAgentTask` invokes agents on catalog workspace.

**MakeAgent** (~280 lines): Build machine via MQTT. Init VM → load _rmx_makeAgent → TrackResponse → Output (logs, warnings, errors, builtLibs, builtExe, deploymentDone). MakeOptions: srcDbName,
targetLibs, targetExe, variant (Debug|Release), clean, deploy options.

**SessionAgent** (~80 lines): L2/L3 REPL sessions. Machine IDs: `$$-l2-$$`, `##-l3-##`. Config: compilerEnabled, database, variant=ByteCode, localAgentConfig.

**Shared Backend** (`shared/Backend/`): Crud.elm (generic HTTP with Bearer auth), Mixer.elm (desktop protocol bridge `remix://`), Track.elm (runtime session protocol, _rmx_type).

## Wasm Bridge (`shared/mixcore/`)

**mixrt.js** (~300 lines): Uint32 buffer codec. Tags: UNDEFINED, BOOL, NUMBER, STRING, BINARY, ARRAY, MAP, CASE, TOKEN, OPAQUE.

**remix-file-loader.js** (~400 lines): MixcoreWasmIntf class. WASI shim, IDB 3-tier caching (SHA-256), FFI hooking (auto-save on write ops).

## Web Components (`web-components/`)

| Package         | Stack  | Key Feature                                                          |
|-----------------|--------|----------------------------------------------------------------------|
| rmx-base        | Elm    | Base WC, Elm app wrapper                                             |
| rmx-editor      | Elm    | Builder editor WC                                                    |
| rmx-remix       | JS     | Remix runtime (extends RmxMixcoreWebcomp)                            |
| rmx-runtime     | JS     | Runtime executor (extends RmxGrooveboxWebcomp, setup/config merging) |
| rmx-monaco      | TS/Lit | Monaco editor with Mix syntax highlighting + error decorations       |
| rmx-auth-google | Elm    | Google auth WC                                                       |
| rmx-file-input  | Elm    | File input WC                                                        |
| rmx-graph-viz   | JS     | GraphViz visualization WC                                            |

## Key Patterns & Conventions

- **Elm 0.19.1 TEA** — Model/Msg/update/view/subscriptions throughout
- **Port-based interop** — tag-based JSON, bidirectional, async callbacks
- **Dual build** — Webpack (builder, historical) + Vite (webapp, modern)
- **URL-first routing** — Route type → URL string → port push
- **Undo** — Stack of `Dict String (Maybe Node)` diffs (L2), full tree snapshots (Card L3)
- **Macro DSL** — 40+ declarative graph operations, plugin-executable
- **Collapsed groups** — Inner bindings materialized on group node with `group/nodeKey/bdTag` format
- **Pointer injection** — Card nodes needing inline Mix codegen get pointer props at L2↔L3 boundary

## Open Questions

- **Migrations**: Schema migration framework (M_018–M_020) — not read
- **Tests**: ~100+ Elm unit test files + ~210 JSON fixtures — not explored
- **Renderer**: SceneGraph-based rendering in `shared/Renderer/` — not explored
- **Shared Common**: ~14 Elm helper modules in `shared/Common/` — not read
- **Function/Transform/Files nodes**: Internal structure not explored
