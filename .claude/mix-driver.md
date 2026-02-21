# Mix `driver` Extra Core Library

> Sources:
> - [driver-compilerRule](https://www.notion.so/1061d464528f8131b06fcd053f6f0c4b)
> - [driver-compilerFile](https://www.notion.so/1061d464528f81239450ed3112cf974c)
> - [driver-compilerLib](https://www.notion.so/1061d464528f8150a0ffc79f50b4fc16)
> - [driver-compilerExe](https://www.notion.so/1061d464528f816aabfae867d32f1519)
> - [driver-compilerDriver](https://www.notion.so/1061d464528f81c99495c44203b5ecec)
> - [driver-compilerREPL](https://www.notion.so/1061d464528f81f08bd0c528f7b07fde)
> Parent: The Mix Programming Language

The `driver` library is an extra core library (part of `mixc`), separate from the Mix stdlib. It deals
with compiler internals and build infrastructure. All modules since PR1831 unless noted.

---

## `compilerRule` — Build Rule Records

Manages Mix compiler build rules stored in the database (library/executable build graph).

### Types

**`rule`** — Full DB record:
```
{ name: string,               // library name
  modules: map(string),       // module name → file record hash
  loadAlwaysModules: array(string),
  imports: array(string),
  requires: array(string),
  ruleType: string,           // "library" | "executable" (unused so far)
  notBuildable: bool,         // if true, rule is not saved to db
  hash: string,
  part: option(string),
  _rmx_id: db.recordID
}
```

**`ruleContent`** — Net content without derived fields:
```
{ name: string,
  modules: map(string),
  loadAlwaysModules: array(string),
  imports: array(string),
  notBuildable: bool
}
```

**`scanFilter`** — Filter for scanning rules:
```
{ part: option(string), name: option(string) }
```
Use `emptyScanFilter` for default (no filter).

### API

| Function | Signature | Notes |
|---|---|---|
| `setPart` | `string -> rule -> rule` | Set `part` field on a rule |
| `scan` | `db.database -> scanFilter -> stream(rule)` | Scan rules from DB |
| `load` | `db.database -> db.recordID -> rule` | Load single rule by ID |
| `save` | `appstate -> db.database -> rule -> rule` | Save rule to DB |
| `delete` | `appstate -> db.database -> rule -> null` | Delete rule from DB |
| `makeLibRule` | `string -> array(compilerFile.file) -> rule` | Create rule for files under a library name |
| `saveLibRule` | `appstate -> db.database -> option(string) -> string -> array(compilerFile.file) -> [rule, bool]` | Save lib rule; returns `[rule, isNewOrOutOfDate]` |
| `getExeRuleHash` | `array(rule) -> string` | Compute hash for hypothetical executable from rules |

**`makeLibRule(libName, files)`** — Creates a rule for `files` all associated to `libName`. Error if any
file is not associated to that library.

**`saveLibRule(appstate, db, part, libName, files)`** — Additionally saves to DB. Returns `[rule, bool]`
where bool is `true` if the rule is new or the library needs recompilation.

**`getExeRuleHash(rules)`** — Returns the hash for an executable built from the given library rules.

Note: `ruleType` is currently always `"library"`. Executable rules may be added in future.

---

## `compilerFile` — Source File Records

Manages Mix source file records stored in the database. Referenced by `compilerRule`.

**Types:**
- `file` — Full DB record: `{ name, library, loadAlways: bool, code: string, hash, imports: array(string), part: option(string), _rmx_id }`
- `fileContent` — Core content only: `{ name, library, loadAlways: bool, code: string }`
- `scanFilter` — `{ part: option(string), library: option(string), name: option(string) }`

**API:**

| Function | Signature | Notes |
|---|---|---|
| `setPart` | `string -> file -> file` | Set `part` field |
| `scan` | `db.database -> scanFilter -> stream(file)` | Scan files from DB |
| `load` | `db.database -> db.recordID -> file` | Load single file by ID |
| `save` | `appstate -> db.database -> file -> file` | Save file to DB |
| `make` | `appstate -> compilerDriver.session -> fileContent -> result(string, file)` | Create full `file` from content; uses session to compute `imports` |

---

## `compilerLib` — Library Records

Manages compiled library records in the database. Works with types from the `compiler` module:
`compiler.library`, `compiler.libraryInDB` (library + stored binary blobs), `compiler.libraryAtTopic` (library + topic path).

**`scanFilter`:** `{ part: option(string), stdlibId: option(compiler.libID), name: option(string) }`

**Key API groups:**

- **Modifications:** `setPart`, `setPartDB`, `setRuleHash`, `setRuleHashDB`
- **Scanning:** `scan(db, app, variant, flt) -> stream(libraryInDB)` | `scanMap -> map(libraryInDB)` | `scanID -> stream(db.recordID)` | `scanExt(db, app, variant, dbFilter)`
- **Load:** `load(db, app, variant, libID) -> option(libraryInDB)` | `loadBlobs(lib) -> [binary, option(binary)]`
- **Save:** `save(appstate, db, lib, [codeID, dbgID]) -> libraryInDB` | `saveBlobs(appstate, db, [code, dbg]) -> [recordID, option(recordID)]`
- **Delete:** `delete(appstate, lib)` | `deleteBlobs` | `deleteByFilter(appstate, db, variant, flt)` | `deleteByNameExcept(appstate, lib)` — keeps only `lib`, deletes same-name+variant others
- **Strip:** `strip(lib) -> library` — shrinks `header`; VM can load but compiler cannot process. Stripped libs are much smaller.
- **Publish:** `publish(appstate, lib, topic, [code, dbg]) -> libraryAtTopic` | `publishFromDB(appstate, lib, topic)` — publishes at `topic`, debug at `topic + ".dbg"`
- **Publish meta:** `publishMeta(appstate, lib, topic)` | `restoreMeta(appstate, topic) -> option(libraryAtTopic)`
- **Remove:** `remove(appstate, topic, haveDbg)` | `removeMeta(appstate, topic)`
- **Dependency:** `closure(map(libraryInDB), array(libID)) -> result(string, array(libID))` — topological sort; checks all required libs are present. `closureFor(f, map(L), libIDs)` — generalized variant.

---

## `compilerExe` — Executable Records

Manages compiled executable records. Mirrors `compilerLib` but for executables.
Types from `compiler` module: `compiler.executable`, `compiler.executableInDB`, `compiler.executableAtTopic`.

**Additional types on `executable`:** `app`, `requires`, `stdlibId`, `variant`, `header`, `override`, `modules`, `ruleHash`, `part`, `name: exeName`.

**Key API groups (same pattern as compilerLib plus):**

- **Modifications:** `setPart`, `setPartDB`, `setRuleHash`, `setRuleHashDB`, `setName`, `setNameDB`
- **Scanning:** `scan(db, app, variant, exeName, filter)` | `scanID(db, variant, name, filter)` | `scanName(db, variant, filter) -> stream([exeName, recordID])` | `exeStoredIDs(exe) -> array(recordID)`
- **Load:** `get(db, app, variant, name, id) -> option(executableInDB)` | `load(db, app, variant, name) -> option(executableInDB)` (loads most recent) | `loadBlobs`
- **Save / Delete / Publish / Remove** — same pattern as `compilerLib`; `deleteAllExcept(appstate, exe)` keeps only `exe`
- **Relink:** `createExeFromLibs(appstate, opts, db, app, variant, map(libraryInDB), array(libID)) -> result(string, executableInDB)` — creates dynamically linked exe from libs. `opts.deleteOtherExecutables: bool`. **Requires FFI only available in amp** (builder, amp-backed web, flutter mobile). Not available in browser without amp or appclip/widget.

---

## `compilerDriver` — Compiler Session Orchestration

The main module for driving the Mix compiler. Creates sessions, compiles code, manages publish/save lifecycle.

**Types:**
- `options` — compiler config: `lib_create`, `lib_name`, `lib_version`, `lib_autoExpose`, `lib_autoExport`, `exe_ignoreOnDemandLibs`, `lang_warnings`, `lang_infos`, `link_static`
- `config` — session config: `app`, `variant`, `options`, `database`, `libFilter: option([string, string])`, `sendAllLibVersions: bool`
- `session` — opaque compiler session handle
- `res(T)` — result-like wrapper with structured errors and accumulated log messages

**Constructors:** `options() -> options` | `config() -> config` — return defaults.

**Result combinators (`res(T)`):**

| Function | Notes |
|---|---|
| `success(lastReq, v)` | Wrap a value |
| `simpleError(lastReq, msg)` | Create an error |
| `andThen(f, r)` | Chain: calls `f` only on success |
| `resMap(f, r)` | Map wrapped value |
| `resConcat(array(res(T)))` | Collect array of results; fails if any fails |
| `anyway(f, r)` | Call `f()` regardless of success/failure; returns `r` |
| `toResult(r)` | Convert to `result(string, T)` (drops logs) |
| `getPayload(r)` | Extract value (or fail) |

**Sessions:**
- `createSession(appstate, config) -> session` — scans DB for visible libs
- `createSessionWithLibs(appstate, config, array(libraryInDB)) -> session` — uses provided libs, no DB scan
- `destroySession(appstate, session)`
- `preferLibrary(appstate, session, libraryInDB)` — hint: prefer this version if same lib ID needed

**Publishing config:** `pubConfig(session, libMode, exeMode) -> compiler.pubConfig` — controls topic paths (private vs shared storage). Used with `getLibTopic`, `getExeTopic`, `getExeMetaTopic`.

**Publish libs/exes from DB:**
- `publishLibInDB(appstate, pubConfig, libraryInDB) -> libraryAtTopic`
- `publishExeInDB(appstate, pubConfig, executableInDB) -> executableAtTopic`
- `restoreSharedLibrary` / `restoreSharedExecutable` — restore if already at shared topic
- `publishRequiredLibs(appstate, session, pubConfig, array(libID))` — publishes all required libs; checks preferred libs first, then DB, then stdlib

**Compile code:**
- `compilePhrases(appstate, session, array(string))` — send array of Mix phrases to compiler
- `compileModules(appstate, session, map(string))` — send map of module name → source
- `imports(appstate, session, map(string)) -> map(array(string))` — parse source, return imports per module

**Save/publish compiled output:**
- `saveLibrary(appstate, session, libAuxProps) -> libraryInDB` | `publishLibrary -> libraryAtTopic`
- `saveExecutable(appstate, session, exeAuxProps) -> executableInDB` | `publishExecutable -> executableAtTopic`
- `libAuxProps`: `{ ruleHash, part }` | `exeAuxProps`: `{ ruleHash, part, name: exeName }`

**Stdlib access:** `getStdlibId(appstate) -> libID` | `getStdlibProps` | `publishStdlib` | `saveStdlib`

---

## `compilerREPL` — REPL Code Sending

*(Since PR1831.)* Helpers for sending compiled code to a VM via a connector.

| Function | Signature | Notes |
|---|---|---|
| `sendCodeFromCompiler` | `appstate -> vm.connector -> compilerDriver.session -> option(string) -> bool -> null` | Send compiled code from session to VM; also publishes required libs |
| `eval` | `appstate -> vm.connector -> compilerDriver.session -> option(string) -> array(string) -> null` | Compile `phrases` then send to VM |

- `echoOpt`: see `vm.sendCode`
- `doInit`: see `vm.sendCode`
- Both have `Res` variants returning `compilerDriver.res(null)` instead of `null`

---

## `compilerClean` — Cleanup

*(Page exists but has no documented API yet.)*
Source: [driver-compilerClean](https://www.notion.so/1061d464528f81858188f7ae55868e2e)

---

## `compilerBuild` — High-Level Build Orchestration

Source: [driver-compilerBuild](https://www.notion.so/1061d464528f81389251c1c46425fe2d)

Wraps `compilerDriver` into a high-level build pipeline for libraries and executables.

**Types:**
- `mixcOptions` — subset of `compilerDriver.options`: `lib_autoExpose`, `lib_autoExport`, `exe_ignoreOnDemandLibs`, `lang_exposeInternals`, `lang_warnings`, `lang_infos`, `link_static` (ignored for libs)
- `buildRequest` — `{ database, app, part: option(string), options: mixcOptions, variant, targetLibs: array(string), targetExe: option(array(string)), targetSAExes: array(string), extraLibs: array(libID), linkParts: array(string), stopOnFirstError: bool }`
- `buildProgress` — `{ libs: map(res(libraryInDB)), exes: map(res(executableInDB)) }`

**Build request builder (pipeline pattern):**
```
let req =
  compilerBuild.request(database, app, variant)
  |> compilerBuild.setStopOnFirstError(true)
  |> compilerBuild.buildMainExe
```

| Function | Notes |
|---|---|
| `request(db, app, variant)` | Create minimal request |
| `setPart(p, req)` | Isolate to a part |
| `setOptions(opts, req)` | Set compiler options |
| `setStopOnFirstError(bool, req)` | Halt on first error vs continue |
| `useExtraLibs(libIDs, req)` | Make extra libs visible + link into exe |
| `linkParts(parts, req)` | Link other parts' screens/agents into exe (code not accessible, only globally exposed modules) |
| `buildLibraries(libs, req)` | Build named libs + their dependencies |
| `buildMainExe(req)` | Build main exe from all libs |
| `buildMainExeOnlyFrom(libs, req)` | Build main exe from specific libs only |
| `buildStandaloneExes(targets, req)` | Build standalone exes for named targets |

**Execute:**
- `execute(appstate, req) -> [buildProgress, bool]` — run from scratch
- `execOn(appstate, req, progress) -> [buildProgress, bool]` — continue with existing progress
- `execEmbedImports(appstate, req, searchBase) -> [buildProgress, bool]` — special: links dynamically imported libs into exe. Requires: request must target a `part`; non-static main exe. Standalone exes do NOT get dynamic import resolution. Imported libs compiled fresh (stdlib compat not an issue).

On full success, new binaries replace old ones in DB. On failure, new binaries are rolled back, keeping prior versions.

---

## `codegenDriver` — Mix Code Generation from Editor Screens

Source: [driver-codegenDriver](https://www.notion.so/1571d464528f8012a46ee165203c279d)

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

Source: [driver-codegenUtil](https://www.notion.so/1571d464528f805cbcb8d017c9338563)

```
def mkStylesRecord  : map(data) -> db.record
def mkFileMetaRecord : string -> map(data) -> db.record
```

---

## `remixFile` — `.remix` Zip File Builder

Source: [driver-remixFile](https://www.notion.so/1571d464528f806dadfecfaa18a004b6)

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

Source: [driver-remixRecipe](https://www.notion.so/1571d464528f8034a9a0fa0ad6f090ea)

Defines the data model for a declarative recipe that describes how to build `.remix` packages. Types only — no functions.

**Key types:**
- `escValue = data` — a value with escaping (escJSON format)
- `record = map(escValue)` — a DB record
- `patch` — scenegraph patch: `{ relNodeId, relPath: array(word), isStyle: bool, basePatch: { type, key, val }, isOn: bool }`
- `simpleModule` — `{ modType, origin, originalName, appMeta: { derivedData } }`
- `fullModule` — like `simpleModule` + `constants: map(escValue)`, `patches: array(patch)`
- `bundle` — top-level build unit: `{ bundle, database, title, surfaces: array(string), setup, setupData, settings, parameters, includeDebug, includeBuilderAssets, appMeta, globalScope: { modules, constants, styles, mixModules: map(string) }, moduleScope: map(fullModule), saveOptions, selectedRecords, selectedFiles }`
- `assets` — `{ records: map(array(record)), files: array(file) }`
- `recipe` — `{ bundles: map(bundle), assets: assets }`

`bundle.parameters` supports `<<BUNDLE.NAME>>` substitution in `setup` and `setupData`.
