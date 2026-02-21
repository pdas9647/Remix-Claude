# Mix Standard Library — Editor & Reflection Modules

> Sources:
> - [dom](https://www.notion.so/1061d464528f81e4b0b6f274bd0e53be)
> - [editor-node](https://www.notion.so/1061d464528f8169ad98d80c2c816f5f)
> - [editor-screen](https://www.notion.so/1061d464528f81dea0d3d7ef8eb5a157)
> - [editor-types](https://www.notion.so/1061d464528f815a9b59eca2599c6b7e)
> - [reflect](https://www.notion.so/1ec2708f02674490843320f09c45ed20)
> Parent: [The Mix Standard Library](https://www.notion.so/1061d464528f8010b0cfc60836c20290)

---

## `dom` / `domUtil` — DOM Values & Manipulation

> Source: [dom](https://www.notion.so/1061d464528f81e4b0b6f274bd0e53be) (since PR 781/786)

### DOM Literal Syntax
```
{ _rmx_type: "dom_group",
  props: { style: { color: "red" }, events: { onClick: handler }, key: val },
  children: [ ... ]
}:dom
```
- `_rmx_type` must be a literal string.
- `style` and `events` inside `props` should be literal objects, or `domProps` values from `dom.propertyMap`.
- Children can be a literal array or a factored-out variable.
- **Do NOT transmit `dom` values between sessions** — symbol table is session-local.

### `sync cell`
```
sync cell x = <dom expression>
```
Special cell containing DOM, transmitted to client via sync protocol. When referenced from another cell, produces a **cell reference** rather than inlining the DOM:
```
cell y = { _rmx_type: "dom_group", children: [x] }:dom
// y contains a cell reference to x, not x's DOM directly
```
`domUtil.restore(y)` → `{_rmx_type:"dom_group", children:[{_rmx_type:"cell", index:<n>}]}`

### Property Lists
```
dom.propertyEmpty : domProps
dom.propertyMap : map(data) -> domProps      // creates domProps (has some cost vs literals)
dom.propertyAdd : domProps -> domProps -> domProps   // p2 takes precedence on conflicts
dom.restoreProperties : domProps -> map(data)
```

### Create Dynamically
```
dom.empty : dom                              // missing/null DOM
dom.createNode : string -> map(data) -> map(data) -> map(data) -> map(data) -> map(data) -> array(dom) -> dom
  // (rmxType, staticProps, dynProps, staticStyle, dynStyle, events, children)
  // static = put into symbol table; dynamic = volatile; events always volatile
dom.createNodeFromProps : string -> domProps -> domProps -> domProps -> domProps -> domProps -> array(dom) -> dom
dom.createCellRef : number -> option(string) -> dom
dom.createTree : data -> dom                 // slowest; converts whole nested object
```

### Access
```
dom.children : dom -> array(dom)
dom.rmxType : dom -> string
dom.props / dom.style / dom.events : dom -> domProps
```

### Serialize (session-local symbol references)
```
dom.toStream / dom.toArray : dom -> number* / array(number)
dom.fromStream / dom.fromArray : number* / array(number) -> dom
```

### Helpers
```
dom.isDom : data -> bool
dom.toDom : data -> option(dom)      // typed as dom if true
dom.isEmpty / dom.isNode / dom.isCellRef : data -> bool
```

### Symbol Table
Constants and accessors for low-level DOM serialisation format:
- Control symbols: `emptyControlSymbol`, `firstChildControlSymbol`, `nextChildControlSymbol`, `endChildControlSymbol`, `dynamicControlSymbol`, `singleCellRefControlSymbol`
- `numControlSymbols`=16, `numIntegerLiterals`=2001 (0–2000), `symbolOffset`=2048, `integerOffset`=16
- `dom.symbolNumber(n)` — symbol for integer n; `dom.symbol(d)` — symbol for any data; `dom.lookup(sym)` — value for symbol

### `domUtil`
```
domUtil.deref : dom -> dom                             // follow cell references
domUtil.restore : dom -> data                          // dom → conventional object (recursive)
domUtil.restoreIf : bool -> dom -> data                // conditional restore
domUtil.restoreArrayIf : bool -> array(dom) -> array(data)
```

---

## DOM Synchronization Protocol (`sync cell`)

> Source: [dom-sync-compress](https://www.notion.so/1061d464528f8112bf01fd7c1d32e2f6)

### `sync cell` keyword
Marks a `dom`-typed cell for automatic synchronization to the client:
```
sync cell p = {_rmx_type:"foo"}:dom    // value must be dom type
```
- When content changes, the cell is retransmitted automatically
- When a `sync cell` is referenced in another cell, it's included **by reference** (as `{_rmx_type:"cell", index:N}`), not by value — only the referenced cell is re-sent on change
- Requires `syncProtocol: 1` in `msg_session_create` to enable

### Enabling sync in session creation
```json
{ "_rmx_type": "msg_session_create", "syncProtocol": 1, ... }
```

### Protocol messages
- `$sync = [sheet, index1, value1, index2, value2, ...]` — transmits new cell contents as arrays of symbols for a named spreadsheet
- `$syncSwitch = "view-N"` — switches current spreadsheet (top of viewstack); bare cell refs use this as target sheet
- `$syncDrop = "view-N"` — drops a closed spreadsheet (when popping views)
- `$symbols = [v1, v2, ...]` — extends the symbol table; values can be: scalars, `{literal:[...]}` wrapped arrays/maps, or key/value lists (alternating key/val symbol pairs for static props)

### Symbol encoding
Symbols ≥ 2048 → index into symbol table (offset 2048). Range 16–2016 → integer literals (16=0, 17=1, ...). Range 0–15 → control symbols:
- `0` = empty DOM node (`dom.empty`)
- `1` / `2` / `3` = firstChild / nextChild / endChildren markers
- `4` = dynamic properties/styles/events follow
- `5` = cell reference follows (cellIndex, optSheet)

---

## `editorTypes` — Binding Direction & Visual Types

> Source: [editor-types](https://www.notion.so/1061d464528f815a9b59eca2599c6b7e) (since PR 1020)

```
module editorTypes
  case input
  case output
  type direction = input | output
  def revDirection : direction -> direction

  case vty_bool | vty_card | vty_color | vty_data | vty_date | vty_email
     | vty_event | vty_icon | vty_image | vty_list(data) | vty_location
     | vty_number | vty_object | vty_ref | vty_string | vty_token | vty_url
  type visualType = ...  // union of above cases

  def isEvent : visualType -> bool
  def isList : visualType -> bool
  def exportVisualType : visualType -> data
  def importVisualType : data -> result(string, visualType)
module end
```

- `direction` — used in `editorNode.binding` to describe whether a bindpoint is input or output.
- `visualType` — the editor-visible type of a bindpoint (e.g. `vty_string`, `vty_card`, `vty_event`).

---

## `editorNode` — Nodegraph Node Introspection

> Source: [editor-node](https://www.notion.so/1061d464528f8169ad98d80c2c816f5f)

```
module editorNode
  type t        // a loaded, parsed editor node
  type query    // lazy query that loads certain nodes

  type binding = {
    cellName: string, bdName: string, displayName: string,
    direction: editorTypes.direction, realDirection: editorTypes.direction,
    order: number, active: bool, isPayload: bool,
    vty: editorTypes.visualType, defaultValue: data, duplicate: bool
  }

  type connection = {
    outCellName: string, outBdName: string,
    inCellName: string, inBdName: string
  }

  // Query builders
  def byParentRef : string -> query
  def filterByType : string -> query -> query
  def filterByInParams / filterByOutParams : query -> query
  def filterByCellName : string -> query -> query
  def filterByInCellName / filterByOutCellName : connection -> query -> query
  def filterByDisplayName : string -> query -> query

  // Execute query
  def toStream : query -> stream(t)
  def toArray : query -> array(t)
  def first : query -> t

  // Node info
  def getType : t -> string
  def getCellName : t -> string
  def getDisplayName : t -> string

  // Traversal
  def children : t -> query
  def siblings : t -> query

  // Connections & bindings
  def bindings : t -> array(binding)
  def outConnections / inConnections : t -> array(connection)
  def followOutConnection : t -> connection -> [t, binding]
  def followInConnection : t -> connection -> [t, binding]
module end
```

---

## `editorScreen` — Screen-Level Editor Introspection

> Source: [editor-screen](https://www.notion.so/1061d464528f81dea0d3d7ef8eb5a157)

```
module editorScreen
  type t   // a loaded, parsed editor_screen record

  def byRef : string -> option(t)      // look up by permanent ref ID
  def byName : string -> option(t)     // look up by name
  def screens : null -> array(t)       // all regular screens (excludes Symbols/Styles)
  def getSymbols : null -> t           // the Symbols module
  def getStyles : null -> t            // the Styles module
  def all : null -> array(t)           // screens + Symbols + Styles

  def getRef : t -> string
  def getType : t -> _type
  def getName : t -> string
  def children : t -> editorNode.query
module end
```

---

## `reflect` — Module, Screen & Spreadsheet Introspection

> Source: [reflect](https://www.notion.so/1ec2708f02674490843320f09c45ed20) (since PR 171)

```
module reflect
  type info = {
    name: string, libID: string, arity: number,
    params: array(string), meta: data, space: modSpace
  }
  type unitInfo = { libID: string, requires: array(string) }

  // Physical modules (since PR1213: names are libID-prefixed)
  def modules : null -> array(string)
  def moduleInfo : string -> option(info)    // pass qualified name
  def moduleQuery : null -> stream(info)
  def moduleCheck : A -> option(moduleDefinition)   // PR905: is value a module instance?

  // Screens, agents, components (since PR1201)
  def screens / agents / components : null -> array(string)
  def screenInfo / agentInfo / componentInfo : string -> option(info)
  def screenQuery / agentQuery / componentQuery : null -> stream(info)

  // Libraries (since PR1213)
  def units : null -> array(string)
  def unitQuery : null -> stream(unitInfo)

  // Navigation
  def push : string -> array(data) -> data
  def pushWithTransition : string -> string -> array(data) -> data   // PR989

  // Spreadsheet introspection
  def spreadsheet : data -> data    // null = whole spreadsheet; or filter map
  def enablePropagation / disablePropagation : data -> data

  // Case decomposition
  def decomposeCase : any -> option([string, option(any)])      // PR1160
  def decomposeCaseData : data -> option([string, option(data)])  // PR1312

  // Deprecated (use moduleInfo instead)
  def moduleArity / moduleParams / moduleMeta : string -> ...
module end
```

### Key Notes

**Module names (since PR1213):** `modules()` returns qualified names like `foo-20221213@abcdef.M`. Pass these to `moduleInfo`. The `space` field: `modPhysical` for real modules; `modScreen/modAgent/modComponent` for meshes.

**`reflect.spreadsheet(filter)`** → map of cell records:
```
{ "cellName": { value, mode, mixName, meta, fullName, frozen, dependsOn, aliasOf } }
```
- `mode`: `"in"` (var), `"out"`, `"prop"` (private propagating), etc.
- Use `reflect.spreadsheet({fullName: cellName(x)})` to query a specific cell.

**`reflect.push(name, params)`** — navigate to a screen by name dynamically. Returns value when the pushed screen is popped. Can fail if: name unknown, wrong param count, non-serializable params, or module too simple.

**`reflect.decomposeCase(v)`** → `some(["caseName", some(arg)])` or `null` if not a case value. E.g. `decomposeCase(some(123))` → `some(["some", some(123)])`.

**`reflect.disablePropagation()`** / **`reflect.enablePropagation()`** — temporarily pause cell update propagation in REPL; `enablePropagation` re-evaluates whole spreadsheet.

**`reflect.moduleCheck(v)`** — returns `some({name: "X"})` if `v` is a module instance, otherwise `null`.

**Spreadsheet statistics** (`reflect.statistics()`):
```
{ cells, inCells, outCells, propCells, undefCells, frozenCells,
  redefinitions, aliases, updates, outputs, inversions, actionClosures }
```

---

## `reflectComponent` — Instantiate Registered Components by Name

> Source: [reflectComponent](https://www.notion.so/1061d464528f817ebfc7d9c8a6e67b84) (since PR 1201)

```
module reflectComponent
  def instantiateComponent : string -> map(link(data)) -> module type slateComponent
module end
```

- Runs a registered component by name. Component must appear in `reflect.components()`.
- `params` is a `map(link(data))` — use `link(cell)` or `newConstCell(expr)` for arbitrary values.
- Returns a module with `.output` (map(data) of output params) and `.view` (DOM from screen card).
- **Performance warning**: runs via slate — much slower than normal components. Not for use in lists.

```
private cell p1 = 42
private cell c = reflectComponent.instantiateComponent("MyComp", { p1: link(p1) }:map)
alias c_output = c.output
alias c_view = c.view
```

---

## `slateComponent` — Run a Mesh as a Component

> Source: [slateComponent](https://www.notion.so/1061d464528f81caaa78f020598d6157) (since PR 1201)

```
module slateComponent(mod_name:string, mesh:slateDef.mesh, params:map(link(data)))
  cell output : data    // map(data) of output params; empty map if no output node
  cell view : data      // value of the screen card
module end
```

- `mod_name` — used for cell naming in traces/profiling; does not need to be unique.
- `params` — pass with `link(existingCell)` or `newConstCell(expr)` for computed values.
- **Performance warning**: much slower than normal components. Not for use in lists.

---

## `slateDef` — Mesh Definition (Build Slates Programmatically)

> Source: [slateDef](https://www.notion.so/914ed5fb0ee44c6480d1c689298fcaf9) (since PR 1020, formerly `symbolMesh`)

A **slate** is a live screen built from simple elements without compilation. Defined via a `mesh`.

### Types
```
type node / nodeType / mesh
type screenParam = { name: string, vty: editorTypes.visualType, value: data }
type spot = { path: array(word), vty: editorTypes.visualType }
case mandatoryValue   // PR1363: marks required inputs (runtime error if left unconnected)
```

### Create Mesh & Nodes
```
createMesh() → mesh
createScreenParams(m, [screenParam])  → node   // in-params tile (one per mesh)
createOutputParams(m, [screenParam])  → node   // out-params tile (agents/components; PR1201)
createScreenCard(m) → node            // final DOM output; bindings: "single" | "multi"
createComponentInstance(m, moduleName) → node  // moduleName = exact Mix module name
createComponentInstanceV2(m, name, some(collection)) → node  // PR1228: dynamic collection load
findComponent(screenName, displayName) → string  // resolve display name → module name
createListConstructor(m, vty) → node  // input "_append_" repeatedly; output "_content_"
createConstant(m, vty, value) → node  // output "_content_"; value can be mandatoryValue
createValueWithPlaceholders(m, vty, value, spots) → node  // bindable path substitutions
createForwarder(m, vty, default) → node  // "_input_" → "_content_"; default=mandatoryValue = required
createOnLoadEmitter(m) → node          // output "_trigger_" for event bindings on load
createNode(m, nodeType, comment) → node  // generic; use n[#nodeType] to duplicate
```

### Labels
```
addLabel(m, node, label)          // node can have multiple labels; label moved if reused
renameLabel(m, oldLabel, newLabel)
lookup(m, label) → node           // error if not found
lookupMatch(m, pattern) → array(node)  // pattern supports "*" wildcard (not dot)
```

### Wire / Unwire
```
// Wire (inNode.inBinding ← outNode.outBinding)
connectInWithOut(m, inNode, inBd, outNode, outBd)
connectInWithOutLabel(m, inLabel, inBd, outLabel, outBd)
connectMultiInWithOutLabel(m, inPattern, inBd, outLabel, outBd)  // pattern with *
append(m, listConstructorNode, outNode, outBdName)
appendLabel / appendMultiLabel(m, inLabel, outLabel/pattern, outBdName)

// Unwire (disconnects are logged in mesh, survive export/import)
disconnectInFromOut / disconnectInFromOutLabel(m, inNode, inBd, outNode, outBd)
disconnectNodes / disconnectNodesLabel(m, node1, node2)
```

### Compose, Transform, Export
```
addMesh(m1, labelPrefix, m2)         // add copy of m2 into m1; labels prefixed with "prefix."
strip(m) → mesh                      // clean up: eliminate disconnect log
exportMesh(m) → data / importMesh(data) → mesh
transformDelete / transformDeleteLabel(m, node/label) → mesh  // non-mutating copy
transformReplace / transformReplaceLabel(m, node/label, fn) → mesh
transformEliminateDeadNodes(m) → mesh
mergeTemplateMeshWithInstanceMesh(template, instance) → mesh
  // instance node labels override template connections for same-named nodes
collections(m) → array(string)            // collections referenced by mesh
importCollections(exportedMesh) → array(string)  // collections from exported data
```

---

## `slateDyn` — Dynamic Slate Registration

> Source: [slateDyn](https://www.notion.so/1061d464528f81d4b82cf08975c802fc) (since PR 1026, formerly `symbolMeshDyn`)

Registrations are **session-scoped only** (not persistent). Do registrations in `_rmx_init` screen.

```
module slateDyn
  def register : appstate -> string -> slateDef.mesh -> null
  def registerWithUpdate : appstate -> string -> slateDef.mesh -> messaging.topic -> null  // PR1102
  def registerAgent : appstate -> string -> slateDef.mesh -> null      // PR1201
  def registerComponent : appstate -> string -> slateDef.mesh -> null  // PR1201
  def loadCollections : appstate -> map(data) -> array(string) -> null // PR1228
module end
```

- `register(appstate, screenName, mesh)` — screen becomes navigable; appears in `reflect.screens()`. Re-registering overrides for new pushes only. Slower than compiled screens.
- `registerWithUpdate(appstate, screenName, mesh, topic)` — also accept live mesh updates via messaging topic (exported mesh as message). Updated mesh should be derived from original (don't create fresh).
- `registerAgent(appstate, agentName, mesh)` — mesh must have output params node.
- `registerComponent(appstate, compName, mesh)` — mesh must have screen card + output params node.

### `loadCollections`
```
loadCollections(appstate, sources, collections)
```
- `sources`: `{ collectionName: {type:"rmx", scope:"global", db:"bar"} }` (optionally + `library` field)
- Since PR1430: collection name can be `<dbname>/<libname>` shorthand.
- After load, components are available for `createComponentInstance` / `createComponentInstanceV2`.
- Component naming: `coll_<colname>__comp_<libname>_<screenname>`

```
// Auto-derive collections from exported mesh:
slateDef.importCollections(exportedMesh) |> slateDyn.loadCollections(appstate, {});
let mesh = slateDef.importMesh(exportedMesh);
slateDyn.register(appstate, "page", mesh);
```

---

## `slateGen` — Generate Mix Code from Mesh

> Source: [slateGen](https://www.notion.so/1061d464528f8138b512caa39c617b5b) (since PR 1020, formerly `symbolMeshGenCode`)

```
module slateGen
  def generateCode : string -> slateDef.mesh -> string         // returns Mix module source
  def saveCode : appstate -> string -> slateDef.mesh -> db.recordID  // saves as _rmx_type:"file"
  def getImports : slateDef.mesh -> array(string)              // imported module names
module end
```

- `generateCode(screenName, mesh)` → Mix module `module NAME(m_params:data, m_transition:data) ... module end`
- `saveCode` stores code in DB; can then compile and load it.
- To run generated code in REPL, also send:
  ```
  import NAME(param, transition)
  alias view_alias = NAME.view
  cell view = view_alias
  ```

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
