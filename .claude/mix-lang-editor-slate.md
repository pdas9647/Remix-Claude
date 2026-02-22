# Mix Standard Library — Editor: Slate Definition & Registration

> Sources:
> - [slateDef](https://www.notion.so/914ed5fb0ee44c6480d1c689298fcaf9)
> - [slateDyn](https://www.notion.so/1061d464528f81d4b82cf08975c802fc)
> - [slateGen](https://www.notion.so/1061d464528f8138b512caa39c617b5b)
>
> Parent: [The Mix Standard Library](https://www.notion.so/1061d464528f8010b0cfc60836c20290)

See also: [mix-lang-editor-dom.md](./mix-lang-editor-dom.md) | [mix-lang-editor-reflect.md](./mix-lang-editor-reflect.md) | [mix-lang-editor-runtime.md](./mix-lang-editor-runtime.md)

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