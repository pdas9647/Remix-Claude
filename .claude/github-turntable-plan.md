# turntable — Exploration Plan

**Repository:** https://github.com/remixlabs/turntable
**Branch:** main
**Stack:** Elm (primary), JavaScript/TypeScript, Vite (bundler), CircleCI (CI/CD), npm, Playwright (e2e), SCSS
**Structure style:** Monorepo — Elm SPA builder app + shared runtime library + 8 web components + webapp host
**Total files:** ~1251

## Architecture Overview

The repo is the **Remix Studio builder** — a visual node-graph IDE written primarily in Elm. It compiles user-designed apps into Mix language code via codegen. The codebase follows Remix's L0/L1/L2/L3 navigation model:
- **L0** — Workspace/app listing
- **L1** — App/screen management
- **L2** — Node graph editor (the core builder surface)
- **L3** — Data editing (tables, trees)

Top-level layout:
- `builder/` — Main Elm builder application (src + tests + migrations + codegen)
- `shared/` — Shared Elm modules + JS runtime (used by both builder and web components)
- `web-components/` — 8 standalone web component packages (rmx-base, rmx-editor, rmx-remix, rmx-runtime, rmx-monaco, rmx-auth-google, rmx-file-input, rmx-graph-viz)
- `webapp/` — Host web application (Elm + Vite, multiple HTML entry points)
- `scripts/` — Maintenance/utility scripts (JS)
- `snowflake/` — Snowflake-specific deployment variant
- `bin/` — CLI/release shell scripts

---

## Layer 1 — Skeleton & Overview (must read)
_Top-level manifests, config, CI, docs — understand repo shape and build system_

| File | Why |
|---|---|
| `README.md` | Repo overview |
| `Makefile` | Top-level build orchestration |
| `.circleci/config.yml` | CI pipeline — reveals build/test/deploy steps |
| `.gitignore` | What's excluded |
| `.npmrc` | npm registry config |
| `.prettierrc` | Code formatting rules |
| `.prettierignore` | Prettier exclusions |
| `package.json` (root) | Root dependencies, workspace config, scripts |
| `rcm.config.mjs` | RCM (Remix Config Manager?) config |
| `tsconfig.base.json` | Base TypeScript config |
| `builder/package.json` | Builder sub-package deps/scripts |
| `builder/elm.json` | Elm dependencies for builder |
| `builder/README.md` | Builder-specific docs |
| `builder/Design.md` | Builder design notes |
| `builder/.babelrc` | Babel config |
| `shared/package.json` | Shared sub-package |
| `webapp/package.json` | Webapp sub-package |
| `webapp/elm.json` | Elm deps for webapp |
| `webapp/README.md` | Webapp docs |
| `webapp/tsconfig.json` | Webapp TS config |
| `webapp/vite.config.js` | Webapp Vite bundler config |
| `webapp/vite.lib.js` | Webapp library build config |
| `scripts/package.json` | Scripts sub-package |
| `scripts/README.md` | Scripts documentation |

**Estimated: ~25 files, ~1500 lines**

---

## Layer 2 — Entry Points & API Surface (high priority)
_Main entry points, routing, ports (Elm↔JS boundary), flags, and host HTML files_

| File | Why |
|---|---|
| `builder/src/Main.elm` | Builder Elm entry point — init/update/subscriptions/view |
| `builder/src/Model.elm` | Top-level builder model (state shape) |
| `builder/src/Ports.elm` | Elm→JS port declarations (interop surface) |
| `builder/src/Route.elm` | Builder URL routing |
| `builder/src/Flags.elm` | Startup flags passed from JS to Elm |
| `builder/src/builder.js` | Builder JS entry point (Elm app init, port wiring) |
| `builder/index.html` | Builder HTML shell |
| `builder/index.dev.html` | Dev variant HTML shell |
| `webapp/src/App.elm` | Webapp Elm entry point |
| `webapp/src/Model.elm` | Webapp model |
| `webapp/src/Route.elm` | Webapp routing |
| `webapp/src/Flags.elm` | Webapp startup flags |
| `webapp/src/lib.ts` | Webapp TS entry/lib |
| `webapp/src/rmx-fullscreen.js` | Fullscreen runtime entry |
| `webapp/src/rmx-init-runtime.ts` | Runtime initialization |
| `webapp/src/rmx-desktop-harness.ts` | Desktop (Tauri) harness |
| `webapp/src/rmx-app-record.ts` | App record entry |
| `webapp/src/remix-file-loader.ts` | .remix file loading |
| `webapp/src/token-exchange.js` | Auth token exchange |
| `webapp/index.html` | Main webapp HTML |
| `webapp/index.desktop.html` | Desktop variant HTML |
| `webapp/index.snowflake.html` | Snowflake variant HTML |
| `webapp/index.app-record.html` | App record HTML |
| `webapp/index.dnd.html` | Drag-and-drop HTML |
| `webapp/vite-plugin-runtime-middlewares.ts` | Custom Vite plugin |
| `shared/Runtime/runtime.js` | Shared JS runtime (port implementations) |
| `shared/Runtime/Runtime.elm` | Shared Elm runtime module |
| `shared/Runtime/Model.elm` | Runtime model |
| `shared/Runtime/Msg.elm` | Runtime messages |
| `shared/Runtime/RuntimePorts.elm` | Runtime port declarations |
| `shared/Runtime/Auth.elm` | Runtime auth handling |
| `shared/Runtime/Helpers.elm` | Runtime helpers |
| `shared/Runtime/VmClientHelpers.elm` | VM client-side helpers |
| `shared/Runtime/VmServerHelpers.elm` | VM server-side helpers |
| `shared/Runtime/README.md` | Runtime docs |
| `shared/mixcore/mixrt.js` | Mix runtime core (Wasm bridge) |
| `shared/mixcore/idb_cache.js` | IndexedDB caching for Wasm modules |
| `shared/mixcore/remix-file-loader.js` | .remix file loader (JS) |
| `shared/Loading.elm` | Loading/splash screen |
| `shared/assets/helpers.js` | Shared JS helpers |
| `shared/assets/helpers.d.ts` | Types for helpers |
| `shared/assets/auth.js` | Shared auth JS |
| `shared/assets/machine.js` | Machine/env detection |
| `shared/assets/mqtt.js` | MQTT pub/sub JS |

**Estimated: ~43 files, ~4000 lines**

---

## Layer 3 — Core Modules (medium priority)
_Node types, codegen, graph UI, L0–L3 views, data model, backend queries — the domain logic_

**Sub-priority tiers within L3 (for budget-constrained reads):**
- **Tier A (must sample):** 3a Data Model core types (Node, NodeGraph, Module, Bindings), 3b L2 top-level (L2.elm, Model.elm, Helpers.elm), 3c Codegen engine (Codegen.elm, CodegenInfo.elm)
- **Tier B (high value):** 3b L0/L1 top-level modules, 3c Card + Action node types, 3e Backend queries, 3f shared Data/TVal
- **Tier C (read if budget allows):** 3b L2 helpers (Absorb/Extract/Paste/Swap), 3c remaining node types (Decision, Function, FSM, Files, Transform, QueryBuilder), 3d Graph UI, Macros, Common, 3f shared Backend/Common/Renderer

### 3a. Data Model (read first — types everything else depends on)

| File | Why |
|---|---|
| `builder/src/Data/Node.elm` | Core node data type |
| `builder/src/Data/NodeGraph.elm` | Node graph structure |
| `builder/src/Data/NodeHelpers.elm` | Node utility functions |
| `builder/src/Data/NodeConstants.elm` | Node constants |
| `builder/src/Data/NodeMetadata.elm` | Node metadata |
| `builder/src/Data/NodeRef.elm` | Node references |
| `builder/src/Data/NodeState.elm` | Node state management |
| `builder/src/Data/Module.elm` | Module (screen) data type |
| `builder/src/Data/Bindings.elm` | Binding system |
| `builder/src/Data/Operator.elm` | Operator definitions |
| `builder/src/Data/GNode.elm` | Generic node abstraction |
| `builder/src/Data/Group.elm` | Node grouping |
| `builder/src/Data/GroupHelpers.elm` | Group helpers |
| `builder/src/Data/Source.elm` | Data source definitions |
| `builder/src/Data/Uid.elm` | UID generation/handling |
| `builder/src/Data/BuilderType.elm` | Builder type system |
| `builder/src/Data/Constants.elm` | Data constants |
| `builder/src/Data/AppMeta.elm` | Application metadata |
| `builder/src/Data/RmxAsset.elm` | Remix asset type |
| `builder/src/Data/RmxFile.elm` | .remix file handling |
| `builder/src/Data/RmxBackend.elm` | Backend config data |
| `builder/src/Data/RmxMode.elm` | Mode enums |
| `builder/src/Data/RuntimeInfo.elm` | Runtime info |
| `builder/src/Data/ConfigNode.elm` | Config node |
| `builder/src/Data/Plugins.elm` | Plugin system data |
| `builder/src/Data/CompiledLib.elm` | Compiled library data |
| `builder/src/Data/QueryDB.elm` | Query DB definitions |
| `builder/src/Data/FuncDef.elm` | Function definitions |
| `builder/src/Data/Tags.elm` | Tag system |
| `builder/src/Data/Sort.elm` | Sort definitions |
| `builder/src/Data/ViewState.elm` | View state |
| `builder/src/Data/UserPrefs.elm` | User preferences |
| `builder/src/Data/SyncedPrefs.elm` | Synced preferences |
| `builder/src/Data/Lineage.elm` | Asset lineage |
| `builder/src/Data/SaveMode.elm` | Save mode |
| `builder/src/Data/SaveType.elm` | Save type |
| `builder/src/Data/ScreenRef.elm` | Screen reference |
| `builder/src/Data/DotRemixManifest.elm` | .remix manifest |
| `builder/src/Data/FileManifest.elm` | File manifest |
| `builder/src/Data/FileItem.elm` | File item |
| `builder/src/Data/DbLocation.elm` | DB location |
| `builder/src/Data/DBRef.elm` | DB reference |
| `builder/src/Data/CloudServer.elm` | Cloud server config |
| `builder/src/Data/LibLocation.elm` | Library location |
| `builder/src/Data/AssetLibs.elm` | Asset libraries |
| `builder/src/Data/EnvBuilder.elm` | Builder environment |
| `builder/src/Data/SearchPayload.elm` | Search payload |
| `builder/src/Data/SingleOrGroupNode.elm` | Single/group node union |
| `builder/src/Data/Misc.elm` | Miscellaneous data |
| `builder/src/Data/DataInProj.elm` | Data in projection |
| `builder/src/Data/ClipboardValue.elm` | Clipboard value |
| `builder/src/Data/ClipboardValueHelpers.elm` | Clipboard helpers |
| `builder/src/Data/Profiler.elm` | Profiler data |
| `builder/src/Data/GetSetState.elm` | Get/set state |
| `builder/src/Data/InlineEditor.elm` | Inline editor data |
| `builder/src/Data/L2Preview.elm` | L2 preview data |
| `builder/src/Data/L3TrackResponse.elm` | L3 track response |
| `builder/src/Data/GcsIndexJson.elm` | GCS index JSON |
| `builder/src/Data/Global/Config.elm` | Global config |
| `builder/src/Data/Global/Info.elm` | Global info |
| `builder/src/Data/Logic/Logic.elm` | Logic node data |
| `builder/src/Data/Logic/Model.elm` | Logic model |
| `builder/src/Data/SortableTree/SortableTree.elm` | Sortable tree |
| `builder/src/Data/SortableTree/Model.elm` | Sortable tree model |
| `builder/src/Data/SortableTree/Helpers.elm` | Sortable tree helpers |
| `builder/src/Data/ModDerivedData.elm` | Module derived data |
| `builder/src/Data/ModDerivedValue.elm` | Module derived values |
| `builder/src/Data/ModMeta.elm` | Module metadata |
| `builder/src/Data/ModMetadata.elm` | Module metadata (alt) |
| `builder/src/Data/ModPosition.elm` | Module positioning |
| `builder/src/Data/MD5Sig.elm` | MD5 signatures |
| `builder/src/Data/MetadataMenu.elm` | Metadata menu |
| `builder/src/Data/ReleaseData.elm` | Release data |
| `builder/src/Data/RValBinding.elm` | RVal binding |
| `builder/src/Data/RValString.elm` | RVal string |
| `builder/src/Data/PersistedProjection.elm` | Persisted projection |
| `builder/src/Data/PersistedStates.elm` | Persisted states |
| `builder/src/Data/ProjectionsTree.elm` | Projections tree |
| `builder/src/Data/SignalInfo.elm` | Signal info |
| `builder/src/Data/StaticNgSummary.elm` | Static NG summary |
| `builder/src/Data/BuilderTask.elm` | Builder task |
| `builder/src/Data/ArtifactSavePayload.elm` | Artifact save payload |

### 3b. L0–L3 Navigation & Views

| File | Why |
|---|---|
| `builder/src/L0/L0.elm` | L0 update logic |
| `builder/src/L0/Model.elm` | L0 model |
| `builder/src/L0/Msg.elm` | L0 messages |
| `builder/src/L0/View.elm` | L0 view |
| `builder/src/L0/Navbar.elm` | L0 navbar |
| `builder/src/L0/Sidebar.elm` | L0 sidebar |
| `builder/src/L0/RemixFileInstaller.elm` | .remix file installer |
| `builder/src/L1/L1.elm` | L1 update logic |
| `builder/src/L1/Model.elm` | L1 model |
| `builder/src/L1/Msg.elm` | L1 messages |
| `builder/src/L1/View.elm` | L1 view |
| `builder/src/L1/Navbar.elm` | L1 navbar |
| `builder/src/L1/Helpers.elm` | L1 helpers |
| `builder/src/L1/HelpersPlugin.elm` | L1 plugin helpers |
| `builder/src/L1/Files.elm` | L1 files |
| `builder/src/L1/FilesModel.elm` | L1 files model |
| `builder/src/L1/AppPlugin.elm` | L1 app plugin |
| `builder/src/L1/PublishToLibModal.elm` | L1 publish modal |
| `builder/src/L1/UpdateAssetModal.elm` | L1 update asset modal |
| `builder/src/L1/Sidebar/*.elm` | L1 sidebar (6 files) |
| `builder/src/L2/L2.elm` | L2 update logic (main builder) |
| `builder/src/L2/Model.elm` | L2 model |
| `builder/src/L2/Msg.elm` | L2 messages |
| `builder/src/L2/View.elm` | L2 view |
| `builder/src/L2/ViewNode.elm` | L2 node view |
| `builder/src/L2/Navbar.elm` | L2 navbar |
| `builder/src/L2/Helpers.elm` | L2 helpers |
| `builder/src/L2/HelpersInit.elm` | L2 init helpers |
| `builder/src/L2/HelpersSave.elm` | L2 save helpers |
| `builder/src/L2/HelpersSync.elm` | L2 sync helpers |
| `builder/src/L2/HelpersConnections.elm` | L2 connection helpers |
| `builder/src/L2/HelpersNodeMsg.elm` | L2 node message helpers |
| `builder/src/L2/HelpersAbsorb.elm` | L2 absorb helpers |
| `builder/src/L2/HelpersExtract.elm` | L2 extract helpers |
| `builder/src/L2/HelpersPaste.elm` | L2 paste helpers |
| `builder/src/L2/HelpersPlugin.elm` | L2 plugin helpers |
| `builder/src/L2/HelpersRuntimeActions.elm` | L2 runtime action helpers |
| `builder/src/L2/HelpersSwap.elm` | L2 swap helpers |
| `builder/src/L2/HelpersTrackResponse.elm` | L2 track response helpers |
| `builder/src/L2/L3Update.elm` | L3 update from L2 |
| `builder/src/L2/Rebuilder.elm` | L2 rebuilder |
| `builder/src/L2/Publisher.elm` | L2 publisher |
| `builder/src/L2/RmxAuth.elm` | L2 auth |
| `builder/src/L2/RtiCompute.elm` | RTI compute |
| `builder/src/L2/RtiDeviceCompile.elm` | RTI device compile |
| `builder/src/L2/RtiLoopSrc.elm` | RTI loop source |
| `builder/src/L2/RtiValidate.elm` | RTI validation |
| `builder/src/L2/CodegenScript.elm` | L2 codegen script |
| `builder/src/L2/CompHelpers.elm` | L2 component helpers |
| `builder/src/L2/Echo.elm` | L2 echo/debug |
| `builder/src/L2/SettingsNode.elm` | L2 settings node |
| `builder/src/L2/Spreadsheet.elm` | L2 spreadsheet view |
| `builder/src/L2/BulkPublishToLibModal.elm` | Bulk publish modal |
| `builder/src/L2/UpdateAssetModal.elm` | Update asset modal |
| `builder/src/L2/Sidebar/*.elm` | L2 sidebar (9 files) |
| `builder/src/L3Data/*.elm` | L3 data modules (9 files) |
| `builder/src/Preview/*.elm` | Preview mode (5 files) |

### 3c. Node Types & Codegen

| File | Why |
|---|---|
| `builder/src/Action/**/*.elm` | Action node type (24 files: 11 top-level + Agent/ 3 + Api/ 7 + Custom/ 3) |
| `builder/src/Card/*.elm` | Card/UI node type (13 files + Primitives/ 10 files) |
| `builder/src/Component/*.elm` | Component node type (4 files) |
| `builder/src/Decision/*.elm` | Decision node type (3 files) |
| `builder/src/Function/*.elm` | Function node type (3 files) |
| `builder/src/FSM/*.elm` | FSM node type (6 files) |
| `builder/src/Transform/*.elm` | Transform node type (2 files) |
| `builder/src/Files/*.elm` | File node types (11 files) |
| `builder/src/Codegen/*.elm` | Code generation engine (14 files) |
| `builder/src/QueryBuilder/*.elm` | Query builder QB1+QB2 (14 files) |

### 3d. Graph UI & Common

| File | Why |
|---|---|
| `builder/src/GraphUI/*.elm` | Graph UI (7 files) |
| `builder/src/DirectManip/*.elm` | Direct manipulation (2 files) |
| `builder/src/Macro/*.elm` | Macro system (5 files) |
| `builder/src/Common/*.elm` | Common builder utilities (~25 files) |
| `builder/src/Page/*.elm` | Page-level views (7 files) |

### 3e. Backend & Agents

| File | Why |
|---|---|
| `builder/src/Backend/*.elm` | Backend queries (7 files) |
| `builder/src/BuilderAgents/*.elm` | Builder agents — VM, compiler (5 Elm + 1 JS) |

### 3f. Shared Elm Modules

| File | Why |
|---|---|
| `shared/Backend/*.elm` | Shared backend (Crud, Mixer, Track, V0) — 4 files |
| `shared/Common/*.elm` | Shared common helpers — 14 files |
| `shared/Data/*.elm` | Shared data types — ~30 files |
| `shared/Data/TVal/*.elm` | TVal (typed value) system — 5 files |
| `shared/Renderer/*.elm` | Renderer (SceneGraph, Model, Renderer) — 3 files + Design.md |
| `shared/Vendor/*.elm` | Vendored Elm packages — 4 files |

**Estimated Layer 3: ~350 files, ~40000+ lines (Tier A alone: ~15 files, ~3000 lines)**

---

## Layer 4 — Data & Config (medium priority)
_Migrations, codegen tooling, build scripts, deployment config_

| File | Why |
|---|---|
| `builder/Migrations/Migration.elm` | Migration framework |
| `builder/Migrations/MigrationHelpers.elm` | Migration helpers |
| `builder/Migrations/M_018/Migration.elm` | Migration 18 |
| `builder/Migrations/M_019/Migration.elm` | Migration 19 |
| `builder/Migrations/M_020/Migration.elm` | Migration 20 |
| `builder/Migrations/M_open_close/Migration.elm` | Open/close migration |
| `builder/Migrations/MigrateLocal.elm` | Local migration |
| `builder/Migrations/MigrateJsonFiles.elm` | JSON file migration |
| `builder/Migrations/ImportJsonFiles.elm` | JSON import |
| `builder/Migrations/Integration.elm` | Integration tests |
| `builder/Migrations/Rebuilder.elm` | Rebuilder migration |
| `builder/Migrations/UpdateTests.elm` | Test updater |
| `builder/Migrations/*.js` | Migration JS runners (8 files) |
| `builder/Migrations/package.json` | Migration deps |
| `builder/Migrations/README.md` | Migration docs |
| `builder/codegen/*.elm` | Codegen tooling (1 file) |
| `builder/codegen/*.js` | Codegen JS (3 files) |
| `builder/codegen/package.json` | Codegen deps |
| `builder/codegen/README.md` | Codegen docs |
| `builder/codegen/vite.config.js` | Codegen vite config |
| `builder/review/src/ReviewConfig.elm` | Elm review config |
| `builder/review/elm.json` | Review deps |
| `builder/rmx-files/` | rmx-files sub-package (all files) |
| `bin/*` | CLI/release scripts (9 files) |
| `scripts/*.mjs` | Utility scripts |
| `scripts/util/*.mjs` | Script utilities |
| `scripts/app_update/*.mjs` | App update scripts |
| `scripts/rebuild/index.js` | Rebuild script |
| `scripts/save_local/index.js` | Save local script |
| `scripts/garbage_collect/index.js` | GC script |
| `scripts/prune_db/index.js` | DB prune script |
| `scripts/blobs/*.js` | Blob management |
| `scripts/files/*.js` | File pruning |
| `snowflake/*` | Snowflake deployment (3 files) |
| `snowflake.mjs` | Snowflake entry script (root level) |
| `playwright.config.ts` | E2E test config |

**Estimated: ~52 files, ~3000 lines**

---

## Layer 5 — Supporting (low priority, skip if budget tight)
_Web components, tests, static assets, JS runtime helpers_

### 5a. Web Components (8 packages)

| Package | Key files |
|---|---|
| `web-components/rmx-base/` | Base web component — `src/RmxBase.elm`, `src/rmx-base.js`, `src/rmx-base-webcomp.js`, `src/lib.js` |
| `web-components/rmx-editor/` | Editor WC — `src/RmxEditor.elm`, `src/rmx-editor.js` |
| `web-components/rmx-remix/` | Remix runtime WC — `src/rmx-remix.js`, `src/rmx-mixcore-webcomp.js` |
| `web-components/rmx-runtime/` | Runtime WC — `src/rmx-runtime.js`, `src/rmx-groovebox-webcomp.js`, `src/dotremix.js` |
| `web-components/rmx-monaco/` | Monaco editor WC — `src/rmx-monaco.ts`, `src/mix.ts`, `src/customMonaco.ts` |
| `web-components/rmx-auth-google/` | Google auth WC — `src/Main.elm`, `src/rmx-auth-google.js` |
| `web-components/rmx-file-input/` | File input WC — `src/Main.elm`, `src/rmx-file-input.js` |
| `web-components/rmx-graph-viz/` | Graph viz WC — `src/rmx-graph-viz.js` |

Each package also has: `package.json`, `elm.json` (if Elm), `vite.config.js`, `index.html`, `README.md`

### 5b. Runtime JS modules

| File | Why |
|---|---|
| `shared/Runtime/js/_constants.js` | Runtime constants |
| `shared/Runtime/js/cloud.js` | Cloud runtime |
| `shared/Runtime/js/dragdrop.js` | Drag/drop |
| `shared/Runtime/js/localstorage.js` | Local storage |
| `shared/Runtime/js/mixrt.js` | Mix runtime (JS) |
| `shared/Runtime/js/rmxfiles.js` | Remix files |
| `shared/Runtime/js/subscribe.js` | Subscription |
| `shared/Runtime/js/webcomps.js` | Web components |

### 5c. Shared Static Assets

| File | Why |
|---|---|
| `shared/assets/css/*.css` | CSS files (4 files) |
| `builder/src/assets/css/*.css` | Builder CSS (5 files) |
| `builder/src/assets/scss/*.scss` | Builder SCSS (1 file) |
| `builder/src/assets/js/*.js` | Builder JS assets (6 files) |

### 5d. Tests

| Directory | Description |
|---|---|
| `builder/tests/` | Extensive Elm unit tests — mirrors src/ structure (~100+ test files) |
| `builder/tests/json_nodegraphs/` | JSON test fixtures — node graph snapshots (~120 files) |
| `builder/tests/json_nodetests/` | JSON test fixtures — node test data (~90+ files) |
| `webapp/tests/` | Webapp Elm tests (2 files) |
| `tests/` | Playwright e2e test (1 file) |

### 5e. Webapp Support

| File | Why |
|---|---|
| `webapp/src/types/index.d.ts` | TypeScript type declarations |
| `webapp/src/assets/*.js` | Firebase messaging assets |
| `webapp/src/rmx-fullscreen.scss` | Fullscreen styles |
| `webapp/public/*` | Static public assets |
| `builder/public/*` | Builder static assets |
| `etc/static/branch.html` | Static branch page |

**Estimated Layer 5: ~350+ files (mostly test fixtures)**

---

## Excluded (do not read)

| Pattern | Reason |
|---|---|
| `**/package-lock.json` | Generated lockfiles |
| `rcm-lock.json` | Lock file |
| `**/node_modules/` | Dependencies |
| `builder/old/*.elm` | Deprecated/archived code |
| `builder/Migrations/Old/` | Old migrations + zip |
| `builder/Migrations/unused/` | Unused migration code |
| `builder/.clocignore` | cloc config (trivial) |
| `builder/tests/json_nodegraphs/*.json` | ~120 JSON test fixtures (bulk data) |
| `builder/tests/json_nodetests/*.json` | ~90+ JSON test fixtures (bulk data) |
| `builder/tests/Generated/` | Generated test output |
| `builder/tests/Compile/` | Compiler test harness |
| `shared/assets/images/` | Image assets |
| `shared/assets/mdi/` | Material Design Icons (vendored fonts/css) |
| `builder/public/img/` | Builder images |
| `webapp/public/img/` | Webapp images |
| `webapp/public/js/` | Bundled WC JS (generated) |
| `web-components/*/bundle.js` | Pre-built bundles |
| `builder/codegen/package-lock.json` | Lock file |
| `.github/PULL_REQUEST_TEMPLATE.md` | PR boilerplate template |
| `.github/ISSUE_TEMPLATE/*.md` | Issue boilerplate templates |

---

## Reading Strategy

**Budget target: ~80 files / ~5000 lines**

1. **L1 (Skeleton):** Read all ~25 files. Focus on `Makefile`, `elm.json` files, and CI config to understand the build/deploy pipeline.
2. **L2 (Entry Points):** Read Main.elm, Model.elm, Ports.elm, Route.elm, Flags.elm for both builder and webapp. Read key JS entry points. Read shared/Runtime and shared/mixcore.
3. **L3 (Core — selective):** Prioritize Data model files (Node, NodeGraph, Module, Bindings), L2 (the main builder surface), and Codegen. Skip individual node types unless specifically needed.
4. **L4–L5:** Skip unless the exploration topic demands it.

**Key relationships to trace:**
- `builder/src/Main.elm` → `L0.elm` → `L1.elm` → `L2.elm` (TEA update chain)
- `builder/src/Codegen/` → generates Mix code from node graphs
- `builder/src/BuilderAgents/` → communicates with Mix compiler (protoquery) and runtime (mix-rs)
- `shared/Runtime/` → runtime execution of compiled apps
- `shared/mixcore/mixrt.js` → Wasm VM bridge (connects to mix-rs)
- `web-components/rmx-*` → standalone deployable units wrapping builder/runtime
