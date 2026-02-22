# Mix `driver` — Build & Codegen Modules

> Sources:
> - [driver-compilerBuild](https://www.notion.so/1061d464528f81389251c1c46425fe2d)
> - [driver-codegenDriver](https://www.notion.so/1571d464528f8012a46ee165203c279d)
> - [driver-codegenUtil](https://www.notion.so/1571d464528f805cbcb8d017c9338563)
> - [driver-remixFile](https://www.notion.so/1571d464528f806dadfecfaa18a004b6)
> - [driver-remixRecipe](https://www.notion.so/1571d464528f8034a9a0fa0ad6f090ea)
    > Parent: The Mix Programming Language

**Covers:** compilerBuild, codegenDriver, codegenUtil, remixFile, remixRecipe

---

## `compilerBuild` — High-Level Build Orchestration

Wraps `compilerDriver` into a high-level build pipeline for libraries and executables.

**Types:**

- `mixcOptions` — subset of `compilerDriver.options`: `lib_autoExpose`, `lib_autoExport`, `exe_ignoreOnDemandLibs`, `lang_exposeInternals`, `lang_warnings`, `lang_infos`, `link_static` (ignored for
  libs)
- `buildRequest` —
  `{ database, app, part: option(string), options: mixcOptions, variant, targetLibs: array(string), targetExe: option(array(string)), targetSAExes: array(string), extraLibs: array(libID), linkParts: array(string), stopOnFirstError: bool }`
- `buildProgress` — `{ libs: map(res(libraryInDB)), exes: map(res(executableInDB)) }`

**Build request builder (pipeline pattern):**

```
let req =
  compilerBuild.request(database, app, variant)
  |> compilerBuild.setStopOnFirstError(true)
  |> compilerBuild.buildMainExe
```

| Function                            | Notes                                                                                          |
|-------------------------------------|------------------------------------------------------------------------------------------------|
| `request(db, app, variant)`         | Create minimal request                                                                         |
| `setPart(p, req)`                   | Isolate to a part                                                                              |
| `setOptions(opts, req)`             | Set compiler options                                                                           |
| `setStopOnFirstError(bool, req)`    | Halt on first error vs continue                                                                |
| `useExtraLibs(libIDs, req)`         | Make extra libs visible + link into exe                                                        |
| `linkParts(parts, req)`             | Link other parts' screens/agents into exe (code not accessible, only globally exposed modules) |
| `buildLibraries(libs, req)`         | Build named libs + their dependencies                                                          |
| `buildMainExe(req)`                 | Build main exe from all libs                                                                   |
| `buildMainExeOnlyFrom(libs, req)`   | Build main exe from specific libs only                                                         |
| `buildStandaloneExes(targets, req)` | Build standalone exes for named targets                                                        |

**Execute:**

- `execute(appstate, req) -> [buildProgress, bool]` — run from scratch
- `execOn(appstate, req, progress) -> [buildProgress, bool]` — continue with existing progress
- `execEmbedImports(appstate, req, searchBase) -> [buildProgress, bool]` — special: links dynamically imported libs into exe. Requires: request must target a `part`; non-static main exe. Standalone
  exes do NOT get dynamic import resolution. Imported libs compiled fresh (stdlib compat not an issue).

On full success, new binaries replace old ones in DB. On failure, new binaries are rolled back, keeping prior versions.

---

## `codegenDriver` — Mix Code Generation from Editor Screens

Generates Mix module source from editor screen DB records.

```
type screen = { name: string, main: db.record, nodes: array(db.record), modPosition: option(db.record) }
case remixDev | case remixBeta | case remixProd
type remixHost = remixDev | remixBeta | remixProd
type genOptions = { like: option(remixHost) }
type genResponse = { mixModules: map(string) }   // module name → source code

def scanScreens : db.database -> array(string)
def loadScreen  : db.database -> string -> result(string, screen)
def generate    : appstate -> genOptions -> screen -> result(string, genResponse)
```

- `scanScreens(db)` — list all screen names in DB
- `loadScreen(db, name)` — load a screen's node records
- `generate(appstate, opts, screen)` — generate Mix source for the screen; `genResponse.mixModules` is the output map of module name → source

---

## `codegenUtil` — Code Generation Utilities

```
def mkStylesRecord  : map(data) -> db.record
def mkFileMetaRecord : string -> map(data) -> db.record
```

---

## `remixFile` — `.remix` Zip File Builder

Assembles `.remix` package zip files in the database, then packs them to a blob record.

**Add raw entries:**

- `addBinary(appstate, db, zipName, path, binary)`
- `addString(appstate, db, zipName, path, string)`
- `addViaRef(appstate, db, zipName, path, recordID)` — reads blob from DB record

**Add msgPacked entries** (binary-serialized map(data)):

- `addMsgPacked(appstate, db, zipName, name, bool, number, map(data))`
- `addMsgPackedJSON`, `addMsgPackedViaRef`, `addMsgPackedViaIndirRef` — variants

**Add compiled binaries:**

- `addLibrary(appstate, db, zipName, libraryInDB)`
- `addExecutable(appstate, db, zipName, executableInDB)`

**Add builder assets:**

- `addBuilderRecordFromDB` / `addBuilderRecordIndirFromDB`

**Manifest** (`manifest.json` in zip):

```
type manifest = { apps: map(bool), records: map(array(recordID)), setup: string,
                  metadata: data, builderAssets: array(string), bare: array(string),
                  save_options: array(string) }
type saveOptions = { upsert, keepOriginal, originalIdAlways, tryToUseId: bool }
def makeManifest(name, bool, bool) -> manifest
def withSetup(setup, data, manifest) -> manifest
def withSaveOptions(saveOptions, manifest) -> manifest
def addManifest(appstate, db, zipName, manifest)
```

**Runtime** (`runtime.json`):

```
type runtime = { dbName, appName: string, settings: map(data), styles: map(data), debugInfo }
type debugInfo = { debugObjectRegistryURL, errorReportURL, errorViewURL, org, workspace: string }
def makeRuntime(dbName, appName) -> runtime  |  withSettings / withStyles / withDebugInfo
def addRuntime(appstate, db, zipName, runtime)
```

**App meta:**

```
type appMeta = { _rmx_type, _rmx_access_level, name, display_name, settings, styles,
                 asset_type, app_state, collaborators, derivedData, errors, last_published_at }
def makeAppMeta(name, display_name, asset_type, app_state) -> appMeta
def withRuntime / withCollaborators / withModInfos → appMeta
def addAppMeta(appstate, db, zipName, appMeta)
```

**Files and user records:**

- `addFile` / `addFileFromDB` — add files at given path
- `addUserRecord(appstate, db, zipName, key, map(data), manifest) -> manifest`
- `addUserRecordIndirFromDB` — via DB reference

**Finalize:**

- `pack(appstate, db, zipName) -> result(string, recordID)` — compress all entries into blob record
- `clean(appstate, db, zipName)` — remove all temp zip-specific entities from DB

---

## `remixRecipe` — Declarative `.remix` Build Recipe

Defines the data model for a declarative recipe that describes how to build `.remix` packages. Types only — no functions.

**Key types:**

- `escValue = data` — a value with escaping (escJSON format)
- `record = map(escValue)` — a DB record
- `patch` — scenegraph patch: `{ relNodeId, relPath: array(word), isStyle: bool, basePatch: { type, key, val }, isOn: bool }`
- `simpleModule` — `{ modType, origin, originalName, appMeta: { derivedData } }`
- `fullModule` — like `simpleModule` + `constants: map(escValue)`, `patches: array(patch)`
- `bundle` — top-level build unit:
  `{ bundle, database, title, surfaces: array(string), setup, setupData, settings, parameters, includeDebug, includeBuilderAssets, appMeta, globalScope: { modules, constants, styles, mixModules: map(string) }, moduleScope: map(fullModule), saveOptions, selectedRecords, selectedFiles }`
- `assets` — `{ records: map(array(record)), files: array(file) }`
- `recipe` — `{ bundles: map(bundle), assets: assets }`

`bundle.parameters` supports `<<BUNDLE.NAME>>` substitution in `setup` and `setupData`.