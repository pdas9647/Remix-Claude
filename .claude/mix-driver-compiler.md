# Mix `driver` — Compiler Modules

>
Sources: [compilerRule](https://www.notion.so/1061d464528f8131b06fcd053f6f0c4b) · [compilerFile](https://www.notion.so/1061d464528f81239450ed3112cf974c) · [compilerLib](https://www.notion.so/1061d464528f8150a0ffc79f50b4fc16) · [compilerExe](https://www.notion.so/1061d464528f816aabfae867d32f1519) · [compilerDriver](https://www.notion.so/1061d464528f81c99495c44203b5ecec) · [compilerREPL](https://www.notion.so/1061d464528f81f08bd0c528f7b07fde) · [compilerClean](https://www.notion.so/1061d464528f81858188f7ae55868e2e)
> Parent: The Mix Programming Language

The `driver` library is an extra core library (part of `mixc`), separate from the Mix stdlib. It deals with compiler internals and build infrastructure. All modules since PR1831 unless noted.

---

## `compilerRule` — Build Rule Records

Manages Mix compiler build rules stored in the database (library/executable build graph).

**`rule`** (DB record):
`{ name, modules: map(string), loadAlwaysModules: array(string), imports: array(string), requires: array(string), ruleType: string, notBuildable: bool, hash, part: option(string), _rmx_id }`
**`ruleContent`** (net content): `{ name, modules: map(string), loadAlwaysModules: array(string), imports: array(string), notBuildable: bool }`
**`scanFilter`**: `{ part: option(string), name: option(string) }` — use `emptyScanFilter` for default.

| Function         | Signature                                                                                         | Notes                                                                 |
|------------------|---------------------------------------------------------------------------------------------------|-----------------------------------------------------------------------|
| `setPart`        | `string -> rule -> rule`                                                                          | Set `part` field                                                      |
| `scan`           | `db.database -> scanFilter -> stream(rule)`                                                       | Scan rules from DB                                                    |
| `load`           | `db.database -> db.recordID -> rule`                                                              | Load single rule by ID                                                |
| `save`           | `appstate -> db.database -> rule -> rule`                                                         | Save rule to DB                                                       |
| `delete`         | `appstate -> db.database -> rule -> null`                                                         | Delete rule from DB                                                   |
| `makeLibRule`    | `string -> array(compilerFile.file) -> rule`                                                      | Create rule for files under a library name (error if file not in lib) |
| `saveLibRule`    | `appstate -> db.database -> option(string) -> string -> array(compilerFile.file) -> [rule, bool]` | Save lib rule; bool = new or out-of-date                              |
| `getExeRuleHash` | `array(rule) -> string`                                                                           | Hash for hypothetical executable from rules                           |

Note: `ruleType` is currently always `"library"`. Executable rules may be added in future.

---

## `compilerFile` — Source File Records

Manages Mix source file records stored in the database. Referenced by `compilerRule`.

**`file`** (DB record): `{ name, library, loadAlways: bool, code: string, hash, imports: array(string), part: option(string), _rmx_id }`
**`fileContent`** (core): `{ name, library, loadAlways: bool, code: string }`
**`scanFilter`**: `{ part: option(string), library: option(string), name: option(string) }`

| Function  | Signature                                                                   | Notes                                                       |
|-----------|-----------------------------------------------------------------------------|-------------------------------------------------------------|
| `setPart` | `string -> file -> file`                                                    | Set `part` field                                            |
| `scan`    | `db.database -> scanFilter -> stream(file)`                                 | Scan files from DB                                          |
| `load`    | `db.database -> db.recordID -> file`                                        | Load single file by ID                                      |
| `save`    | `appstate -> db.database -> file -> file`                                   | Save file to DB                                             |
| `make`    | `appstate -> compilerDriver.session -> fileContent -> result(string, file)` | Create full `file` from content; uses session for `imports` |

---

## `compilerLib` — Library Records

Manages compiled library records in the database. Types from `compiler` module: `compiler.library`, `compiler.libraryInDB` (lib + stored binary blobs), `compiler.libraryAtTopic` (lib + topic path).

**`scanFilter`:** `{ part: option(string), stdlibId: option(compiler.libID), name: option(string) }`

**API groups:**

- **Modify:** `setPart`, `setPartDB`, `setRuleHash`, `setRuleHashDB`
- **Scan:** `scan(db, app, variant, flt) -> stream(libraryInDB)` | `scanMap -> map` | `scanID -> stream(recordID)` | `scanExt(db, app, variant, dbFilter)`
- **Load:** `load(db, app, variant, libID) -> option(libraryInDB)` | `loadBlobs(lib) -> [binary, option(binary)]`
- **Save:** `save(appstate, db, lib, [codeID, dbgID]) -> libraryInDB` | `saveBlobs(appstate, db, [code, dbg]) -> [recordID, option(recordID)]`
- **Delete:** `delete(appstate, lib)` | `deleteBlobs` | `deleteByFilter(appstate, db, variant, flt)` | `deleteByNameExcept(appstate, lib)` — keeps only `lib`
- **Strip:** `strip(lib) -> library` — shrinks `header`; VM can load but compiler cannot process. Much smaller.
- **Publish:** `publish(appstate, lib, topic, [code, dbg]) -> libraryAtTopic` | `publishFromDB(appstate, lib, topic)` — publishes at `topic`, debug at `topic + ".dbg"`
- **Publish meta:** `publishMeta(appstate, lib, topic)` | `restoreMeta(appstate, topic) -> option(libraryAtTopic)`
- **Remove:** `remove(appstate, topic, haveDbg)` | `removeMeta(appstate, topic)`
- **Dependency:** `closure(map(libraryInDB), array(libID)) -> result(string, array(libID))` — topological sort; checks all required libs present. `closureFor(f, map(L), libIDs)` — generalized variant.

---

## `compilerExe` — Executable Records

Mirrors `compilerLib` but for executables. Types: `compiler.executable`, `compiler.executableInDB`, `compiler.executableAtTopic`.

**`scanFilter`:** `{ part: option(string) }`

**API groups (same pattern as compilerLib plus):**

- **Modify:** `setPart`, `setPartDB`, `setRuleHash`, `setRuleHashDB`, `setName`, `setNameDB`
- **Scan:** `scan(db, app, variant, exeName, flt)` | `scanID` | `scanName(db, variant, flt) -> stream([exeName, recordID])` | `exeStoredIDs(exe) -> array(recordID)`
- **Load:** `get(db, app, variant, name, id) -> option(executableInDB)` | `load(db, app, variant, name)` (most recent) | `loadBlobs`
- **Save / Delete / Publish / Remove** — same pattern as `compilerLib`; `deleteAllExcept(appstate, exe)` keeps only `exe`
- **Relink:** `createExeFromLibs(appstate, opts, db, app, variant, map(libraryInDB), array(libID)) -> result(string, executableInDB)` — dynamically linked exe from libs.
  `opts.deleteOtherExecutables: bool`. **Requires FFI only in amp** (builder, amp-backed web, flutter mobile). Not in browser without amp or appclip/widget.

---

## `compilerDriver` — Compiler Session Orchestration

Main module for driving the Mix compiler. Creates sessions, compiles code, manages publish/save lifecycle.

**`options`**: `{ lib_create, lib_name, lib_version, lib_autoExpose, lib_autoExport, exe_ignoreOnDemandLibs, lang_exposeInternals, lang_warnings, lang_infos, link_static }`
**`config`**: `{ app, variant, options, database, libFilter: option([string, string]), sendAllLibVersions: bool }`
**`session`** — opaque handle. **`res(T)`** — result-like wrapper with structured errors + accumulated logs.

**Constructors:** `options()` / `config()` → defaults.

**Result combinators (`res(T)`):** `success(lastReq, v)` | `simpleError(lastReq, msg)` | `andThen(f, r)` — chain on success | `resMap(f, r)` | `resConcat(array(res))` — fails if any fails |
`anyway(f, r)` — call f regardless | `toResult(r)` — drops logs | `getPayload(r)`

**Sessions:** `createSession(appstate, config)` — scans DB for visible libs | `createSessionWithLibs(appstate, config, libs)` — uses provided libs | `destroySession` |
`preferLibrary(appstate, session, lib)` — hint: prefer this version if same lib ID needed

**Publish config:** `pubConfig(session, libMode, exeMode) -> compiler.pubConfig` — controls topic paths (private vs shared). Topic getters: `getLibTopic`, `getExeTopic`, `getExeMetaTopic`.

**Publish from DB:** `publishLibInDB` | `publishExeInDB` | `restoreSharedLibrary` / `restoreSharedExecutable` | `publishRequiredLibs(appstate, session, pubConfig, libIDs)` — checks preferred → DB →
stdlib

**Remove published:** `removeLibrary` | `removeExecutable` | `removeSharedLibsNoLongerRequired(appstate, pc, keepExes, delExes)` — removes libs needed by `delExes` but not by `keepExes`

**Compile:** `compilePhrases(appstate, session, array(string))` | `compileModules(appstate, session, map(string))` | `imports(appstate, session, map(string)) -> map(array(string))`

**Save/publish compiled output:** `saveLibrary(appstate, session, libAuxProps) -> libraryInDB` | `publishLibrary -> libraryAtTopic` |
`saveExecutable(appstate, session, exeAuxProps) -> executableInDB` | `publishExecutable -> executableAtTopic`
`libAuxProps`: `{ ruleHash, part }` | `exeAuxProps`: `{ ruleHash, part, name: exeName }`
NB: `publishLibrary`/`publishExecutable` cannot pass `pubConfig`; workaround: save to DB, then use `publishLibInDB`/`publishExeInDB`.

**Stdlib:** `getStdlibId(appstate) -> libID` | `getStdlibProps` | `publishStdlib` | `saveStdlib`

---

## `compilerREPL` — REPL Code Sending

Helpers for sending compiled code to a VM via a connector.

| Function               | Signature                                                                        | Notes                                             |
|------------------------|----------------------------------------------------------------------------------|---------------------------------------------------|
| `sendCodeFromCompiler` | `appstate -> vm.connector -> session -> option(string) -> bool -> null`          | Send compiled code to VM; publishes required libs |
| `eval`                 | `appstate -> vm.connector -> session -> option(string) -> array(string) -> null` | Compile `phrases` then send to VM                 |

- `echoOpt` / `doInit`: see `vm.sendCode`. Both have `Res` variants returning `compilerDriver.res(null)`.

---

## `compilerClean` — Cleanup

*(No documented API yet.)* Source: [driver-compilerClean](https://www.notion.so/1061d464528f81858188f7ae55868e2e)
