# Mix Standard Library — Editor: Runtime, Viewstack & Tooling

> Sources:
> - [container](https://www.notion.so/1061d464528f81d48148cb3151b18612)
> - [viewstack](https://www.notion.so/1061d464528f819e83a0e22a35b2dde7)
> - [debugger](https://www.notion.so/1061d464528f81fb9ef5d8ba1989cbd5)
> - [testing](https://www.notion.so/1061d464528f81c58a59efbfcef15994)
> - [compiler](https://www.notion.so/1061d464528f819d9e28d8352aeeaf7a)
> - [vm](https://www.notion.so/1061d464528f81688a15c4c27a0fe5be)
>
> Parent: [The Mix Standard Library](https://www.notion.so/1061d464528f8010b0cfc60836c20290)

See also: [mix-lang-editor-dom.md](./mix-lang-editor-dom.md) | [mix-lang-editor-reflect.md](./mix-lang-editor-reflect.md) | [mix-lang-editor-slate.md](./mix-lang-editor-slate.md)

---

## `container` — Array & Map Operations

> Source: [container](https://www.notion.so/1061d464528f81d48148cb3151b18612)

Works on both arrays AND maps. `word` = string (map key) or number (array index).

```
// Lookup
safeGet(key, c) → data           // undefined on any error
descend(path, c) → data          // walk array of keys/indices; undefined on any error

// Update (returns new container)
pathUpdate(path, newVal, c)              // error if path missing
pathUpdateAndAugment(path, newVal, c)   // creates missing fields/indices

// Iterate
deepIterPath(f, c)   // calls f(node, path) for c and all descendants

// Map
shallowMap(f, c)    // map direct children
deepMap(f, c)       // map recursively (post-order)

// Search (shallow = direct children; deep = any descendant)
shallowFind / deepFind(f, c) → data           // first match or undefined
shallowFindMapped / deepFindMapped(f, c) → S // first defined result of f
shallowFindIndex(f, c) → word                 // key/index of first match
deepFindPath(f, c) → array(word)              // path to first match

// Merge
shallowMerge(c1, c2)      // outer level only; c2 keys win
deepMerge(c1, c2)         // recursive; c2 keys win
deepMergeBuilder(c1, c2)  // like deepMerge but won't replace with null (builder semantics)
mergeReplaceBuilder(c1, c2) // recursive; c2 always wins (Object node "without passthrough")

// Convert
toStream(c) → stream(data)           // elements of array or map
toStreamPairs(c) → stream([key, val])
```

---

## `viewstack` — Viewstack Navigation & Embedded Views

> Source: [viewstack](https://www.notion.so/1061d464528f819e83a0e22a35b2dde7)

### AppViewRequest

Create with `appView M(p1,p2)` syntax, or dynamically:

- `dynamicAppView(name, params)` — screen if called from screen, agent if from agent
- `screenAppView / moduleAppView / agentAppView(name, params)` — explicit type (PR1201)
- `currentAppView()` — request that created the current view (runtime only)
- `entryRedirection(req)` / `screenRedirection(screenName, req)` — route via `_rmx_entry` (PR1404)

### Stack Manipulation (immediate effect)

```
load(req)                                    // replace entire stack with new view
push(req) → data                             // push new view; returns pop value
pushWithTransition(transition, req) → data
switchTop(req) → data                        // replace current view
switchCaller(k, req) → data                  // drop k views then replace top
pop(v)                                       // remove current view; v passed to push caller
popOrIgnore(v)                               // pop or no-op if at bottom
popCaller(k, v)                              // drop k extra then pop
toFront(k, v) → data                         // bring index-k view to front; suspends current
reset()                                      // re-run current screen from initial params
refresh()                                    // recompute cells using onRefresh
```

### Invisible View Manipulation

```
drop(k)       // remove view at index k (k >= 1)
swap(j, k)    // swap views at indices j and k
```

### Inspection

```
length() → number         // always >= 1
get(k) → data             // description of view at index k
currentViewName() → string
currentViewId() → string
```

### Schedule Actions

```
schedule(action, arg)    // inherits FG/BG from caller
scheduleFG / scheduleBG(action, arg)
```

Useful for triggering loops via `on` — each scheduled action is a new user interaction.

### Embedded Views

```
embed(req) → embeddedView
embedWithOptions(req, {trace:bool}) → embeddedView
embedRemote(req, {baseURL, app, token}) → embeddedView   // remote app session
terminate(ev)                       // terminate embedded viewstack; ends cellMessages stream
cellMessages(ev) → stream(data)     // {cellname, cellvalue}; $location for navigation, $close for close
invoke(appstate, ev, ac, arg, {nextActions}) → {ok, nextActions}
close(v)                            // close embedded view; sends $close msg with v
popOrClose(v)                       // pop if underlying view exists, else close
selfTerminate(appstate)             // terminate own embedded viewstack (PR1657)
```

### Embedded Compiler Sessions (PR909)

```
embedCompiler({baseURL, app, token, options}) → embeddedView
getSessionOfRemote(ev) → track.sid
sendRequestToRemote(ev, msg)
errorMessages(ev) → stream(data)
logMessages(ev) → stream(data)
```

### Propagation

```
propagate(appstate)   // force immediate cell propagation mid-action
flush(appstate)       // propagate + flush output buffers to client
```

---

## `debugger` — Remote Debug Commands & Streams

> Source: [debugger](https://www.notion.so/1061d464528f81fb9ef5d8ba1989cbd5) (since PR 1160)

```
// Send command to a session with an open debug channel
debugCommand(appstate, app, debugChannel, cmd) → result(string, data)

// Streams (infinite, cannot be closed)
debugEvents(app, channel) → stream(data)    // Mix events (adjust level with cmd_mixEventLogging_set)
debugMirror(app, channel) → stream(data)    // DOM output of target app
```

**Opening a debug channel:** append `?_rmx_debugChannel=xyz` to any app URL, or send `cmd_debugChannel_enable("xyz")` via `track.msgViewInvoke`.

**Command builders** (pass result to `debugCommand`):

- Profile: `cmd_profile_enable/disable/clear/get/save/status`
- Trace: `cmd_trace_enable/disable/status/setLevels/getLevels`
- Stats: `cmd_stats_clear/get/get_clear`
- Mix events: `cmd_mixEventLogging_set(level) / _status`
- Debug channel: `cmd_debugChannel_enable(id) / _disable / _status`
- DOM restore: `cmd_domRestore_enable / _disable`
- VM: `cmd_vmMemStats`
- Messaging: `cmd_messaging_list / _set / _send / _get`
- Viewstack: `cmd_viewstack_topViewID / _screenFlush / _list / _get`
- View/action: `cmd_view_get / _spreadsheet / _spreadsheet_omitValue / _actions`, `cmd_action_get`
- DB: `cmd_db_query`
- Testing: `cmd_testing_startRecording / _stopRecording / _replayTest / _replayTestOverride / _replayTestVisually / _replayTestVisuallyOverride`

---

## `testing` — App-Level Test Recording & Replay

> Source: [testing](https://www.notion.so/1061d464528f81c58a59efbfcef15994) (heavily modified PR1160)

```
// Record
startRecording(appstate, recordingID)
stopRecording(appstate)

// Configure checks
type configChecks = { enableText, enableInput, enableIcon, enableImage, enableStyle, enableColor: bool }
type createConfig = { root: array(word), checks: configChecks }
def emptyConfigChecks : configChecks

// Create test from recording
createTestSequence(appstate, db, recordingID, shotID, appViewRequest, createConfig)
listAll(db) / listForScreen(db, screenName) / delete(appstate, db, id)

// Snapshots (store user records = records without _rmx_type)
snapshotCreate(appstate, db, name)
snapshotRestore(appstate, db, name)   // delete user records + copy snapshot to head
snapshotDelete(appstate, db, name)
snapshotList(db) → array(data)        // testSnapshotHead records
snapshotTimestamp(db, record) → string

// Replay overrides
type overrides = { snapshot: option(string), delay: number,
                   breakAndResume: option(string), breakBeforeInvoke: bool }
def emptyOverrides : overrides

// Batch replay
replayTest(appstate, name, overrides, output_channel) → {good:bool, message:string}
replayTestVisually(appstate, name, overrides)      // sends output to current display
replayTestInSession(appstate, app, name, overrides) // headless session of another app
replayAllTests(appstate, app) → array(testSequenceHead + {good,message})

// Break-and-resume protocol (BRP)
brpPull(id) → data          // get next status message
brpInstruct(id, cmd)        // send: resume | abort | acceptChange
// BRP status codes: started, step, break, resumed, abort, acceptedChange,
//                   testError, mismatch, noSuchTest, outdatedFormat, good
```

DB record types created: `recording`, `testSequenceHead`, `testSequence`, `testSnapshotHead`, `testSnapshot`.

---

## `compiler` — Library & Executable Type Definitions

> Source: [compiler](https://www.notion.so/1061d464528f819d9e28d8352aeeaf7a) (since PR 1306; changed PR1831/1869)

Primarily a **type module** used by the build toolchain. Contains no business logic.

```
type libID = string   // "<name>-<version>@<hash>"
type pubMode = pubKeep | pubAutoDel | pubShared
type pubConfig = { libPubMode, exePubMode, privatePrefix, sharedPrefix }
type storedBinary = { db, _rmx_id, dataRef }
type library = { name, version, hash, app, id:libID, requires, stdlibId, variant, header, modules, ruleHash, part }
type libraryAtTopic = { lib, topic:string, haveDbg:bool }
type libraryInDB = { lib, stored:storedBinary, storedDbg:option(storedBinary) }
type exeName = mainExe | standaloneExe(string)   // PR1869
type executable = { app, requires, stdlibId, variant, header, override, modules, ruleHash, part, name:exeName }
type executableAtTopic / executableInDB
type hdrReq = { reqname, reqversion, reqhash }
type ondemandLib = { odid:hdrReq, odorigins, odtriggers }
type header = { format, isexe, abi, requires, version, hash, numsections, numglobals, ondemand, ... }

// Helpers
formatLibID(name, version, hash) / formatLibID_Req(hdrReq) → libID
parseLibID(id) → result(string, [name, version, hash])
parseLibID_Req(id) → result(string, hdrReq)
mkStoredBinary(db, record) / storedIDs / libStoredIDs
parseOrigin / formatOrigin    // originLoaded | originDatabase(string)
blobLoad(storedBinary) → binary
blobSave(appstate, db, binary) → db.recordID
```

---

## `vm` — Additional VM Instances

> Source: [vm](https://www.notion.so/1061d464528f81688a15c4c27a0fe5be) (PR1831: some functions moved to `compilerREPL`)

```
// VM memory statistics
memstats() → data

// VM preferences (agentVariant etc.)
type preferences = { agentVariant: string }
defaultPreferences : preferences
loadPreferences(db) → [preferences, db.recordID]
savePreferences(appstate, db, prefs_with_id) → null

// FFI interception
case syncFFI(appstate -> array(data) -> data)
case asyncFFI(appstate -> array(data) -> co.channel(result(string,data)))
type ffi = syncFFI | asyncFFI

// VM config & lifecycle
type config = { app, variant, interceptFFI:map(ffi), interceptDynloadFFI:bool,
                subscribeToOutput:bool, extraEnv:map(data), session:string }
type connector / cellMsg / logMsg / output

config() → config           // default config
create(appstate, config) → connector
destroy(appstate, connector)

// Output
outputStream(connector) → stream(output)
outputEcho(output) → option(string)
outputError(output) → option(map(data))
outputValue(cellName, output) → result(map(data), option(cellMsg))
outputFind(cellName, option(string), stream(output)) → result(...)
forwardOutput(appstate, connector)   // pipe VM output to current session

// Send code
sendCode(appstate, connector, libs:array(libraryAtTopic), exe:executableAtTopic, override, debug)

// Interactions
load(appstate, connector, screenName, params)
invoke(appstate, connector, actionName, env, arg, nextActions, opts)
introspect_spreadsheet(appstate, connector, filter)
introspect_debug(appstate, connector, channel, params)
```