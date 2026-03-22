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

## Recent PRs — Mar 8–13, 2026

18 PRs merged. High-signal marked ★.

### High-signal

| PR           | Summary                                                                                                                                                                                                                        | Author     |
|--------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|------------|
| **#11789** ★ | **Webcomp methods with arguments** (+441/-256, 20 files) — web component methods now accept arguments declared in manifest; invoked via new payload-based protocol.                                                            | simonh1000 |
| **#11791** ★ | **File node content binding** (+617/-224, 8 files) — File node exposes a `content` output binding with optional JSON parsing, enabling inline data extraction from uploaded files.                                             | simonh1000 |
| **#11794** ★ | **Catalog fetch perf** (+80/-71, 10 files) — catalog search was downloading 3MB+ (full appclips) and freezing UI. Now sends only previews to L1 (500KB). Also fixed O(n²)→O(n) preview rendering via `Dict` instead of `List`. | dprophete  |
| **#11814** ★ | **Remove name from ModMeta** (1 commit, 21 files, -31 net) — dropped unused `name` field from ModMeta; was causing Build server problems.                                                                                      | simonh1000 |

### Features & fixes

- #11808 — Fix `CompState.isChanged` calculation after Save and when deep-linking to L2
- #11803 — Fix sessionAgent not started when loading preview URL directly (broke L2); fix debug msgs in mixer
- #11805 — Agent refresh for nested callable (bug fix)
- #11810 — Fix codegen regression for `publish + retained` (regression from #11081)
- #11802 — Remove unused node validities; better missing-DB error handling in Query codegen
- #11798 — Add trace echo to subscriptions (investigation for cloud subscribe echo bug)
- #11795 — Add `setIsBusy` to stage1/2 codegen
- #11793 — Better paste (dprophete)

### Minor

- #11811 — Fix cell counts | #11812 — Minor code cleanup | #11809 — Fix vmID-in-use on new project
- #11796 — Table primitives in `simplifySetDomGroup` (global styles fix for table containers)
- #11779 — Remove unhandled promise in StartWasm

## Recent PRs — Mar 15–21, 2026

18 PRs merged. High-signal marked ★.

### High-signal

| PR                  | Summary                                                                                                                                                                                                  | Author               |
|---------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|----------------------|
| **#11539** ★        | **OPFS storage backend** (60 commits, 17 files) — replaced IndexedDB WASI shim with OPFS-based mixcore Wasm. New worker install, IDB→OPFS migration. Removed `idb_cache.js`/`mixrt.js`. CI→`make setup`. | benozol              |
| **#11834** ★        | **`data-rmx-cid` attribute** (15 files) — internal actions/webcomps use `data-rmx-cid` instead of `id`. Frees `id` as user-facing property for anchors. Runtime selectors via `selectorByCid()`.         | tlentz               |
| **#11807** ★        | **Pub-Sub modelling + Cloud Sub UI** (12 files, +681/-378) — models 4 pub/sub variants. CloudDB support with workspace/DB selector wizard.                                                               | simonh1000           |
| **#11815** ★        | **Screenshot overhaul** (20 files) — replaced `html2canvas` with `html-to-image`. Base64 JPG in editor_screen4 (3–4KB). Desktop Safari fixes. L0 shows home screen only.                                 | dprophete            |
| **#11820+#11823** ★ | **Callable build fixes** — fixed callable list computation (wrong prefixes) causing makeAgent failures on Mixer. Callables excluded from targets.                                                        | dprophete+simonh1000 |
| **#11813** ★        | **L3 QB codegen standardization** (14 files) — all QB L3 codegen now uses standard `cell out_tmp`.                                                                                                       | simonh1000           |

### Features & fixes

- #11835 — L1 Files preview of `.mixsrc` files (simonh1000)
- #11825 — Plugin inEvents on regular screens — `Select nodes` available outside plugins (dprophete)
- #11826 — InEvents UI: show descriptions per system in-event (dprophete)
- #11822 — Add Gerd's new tooling plugin | #11821 — Remove download file button for Desktop (simonh1000)
- #11827 — No "replace/keep" warning on L1 clone (dprophete)

### Minor

- #11828 — Standardize `cloudDB`→`cloudDb` naming (23 files) | #11824 — OPFS follow-up fixes
- #11819 — L1 search wording | #11817 — Desktop Rebuild+Make styles fix | #11818 — Exclude `action` prop from screenshot MD5
