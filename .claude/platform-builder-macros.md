# Platform Builder — Macros, Modes & Initialization

> Parent: Remix Documentation > User Guides and Tutorials > Topics
> Split from platform-builder.md — covers Macros, Edit/Run/Configure modes, and _rmx_init.

---

## Macros

> Source: [Macros](https://www.notion.so/2161d464528f801f8ddaf479d816d99f)
> Parent: Remix Studio User Guide
> Sub-page: [Make Agent macro details](https://www.notion.so/1f11d464528f80d7b7fbfba55aecd426)

Commands executed by the builder — programmatic remote control (create nodes, connect bindings, etc.).

### Invocation Methods

**1. Individual macro tiles** — Action tiles found in the new node menu → `Action` tab. Each tile: `invoke` binding (in), `next` binding (out), `error` binding (out). Chain with `next`.

**2. Mix-based MacroRunner** — Function tile produces a `List Object` of macro commands → passed to `MacroRunner` action tile → runs sequentially.

### Macro Tiles

| Tile                     | Description                                                                                                                                                                                                                    |
|--------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| **Node Search**          | Triggers library sidebar search; user selects asset. Bindings: `type` (all\|card\|data\|query\|decision\|function\|action\|annotation), optional lib/tag filters. Returns: `{name, source, displayType, metadata, collection}` |
| **Reload Graph**         | At L1: re-reads DB + refreshes view (e.g. after git pull)                                                                                                                                                                      |
| **Screen Search**        | Sidebar search for screen assets. `type`: all\|screen\|code\|agent\|annotation. Same return format as Node Search                                                                                                              |
| **Make Agent**           | Compiles screen/widget/agent → creates .remix → deploys to workspace. See sub-page for full options                                                                                                                            |
| **Get Module Signature** | Returns in- & out-params of a module                                                                                                                                                                                           |
| **ArtifactSaveAs**       | Brings up 'Save As' modal; all fields configurable. `autosave=true` skips modal if one lib + name provided. Returns share URL on success, error string on cancel                                                               |
| **ArtifactSave**         | Like ArtifactSaveAs but for 'Save' modal. `autosave=true` skips modal if name provided                                                                                                                                         |

### Mix Macro Module Commands

```
// Clipboard
macro.copy(ndNames)
macro.paste()
macro.pasteContent(json-content)           // {command:"PasteContent", content, gridPos, collapsed}

// Nodes
macro.createNode(ndName, ndType)
macro.createNodeFromExternal(ndName, format, content)  // format: "json" (creates data node)
macro.createNodeFromAst(ndName, ast)       // see AST examples below
macro.moveNode(ndName, x, y)
macro.cloneNode(srcNdName, dstNdName)
macro.deleteNode(ndName)
macro.renameNode(oldNdName, newNdName)
macro.replaceNode(nameOrKey, ...)
macro.setRootNode(ndName)
macro.addAnnotation(ndName, annotation)

// Misc
macro.addImport(dbName, modName, nodeName)
macro.createNodeGraphFromMesh(mesh)
macro.noOp()
macro.comment(comment)
macro.renameScreen(screenName)
macro.groupNodes(ndNames, groupName)

// Bindings
macro.addBinding(ndName, path, bdType, direction)   // direction: "in"/"out"
macro.deleteBinding(ndName, bdName, direction)      // DataTee only
macro.addConnection(srcNdName, srcBdName, dstNdName, dstBdName)
macro.autoConnect(srcNdName, srcBdName, dstNdName)
macro.getDefaultValue(ndName, bdName)
macro.renameBinding(ndName, oldBdName, newBdName)
macro.invokeAction(ndName, bdName)
macro.setDefaultValue(ndName, bdName, value)        // absorbing
macro.absorbBinding(ndName, bdName)                 // absorbs static/AST value

// Components
macro.convertToComponent(ndNames, compName)
macro.editComponent(ndName)   // same as clicking "edit component" in builder
macro.doneEdit()              // same as clicking "done" after editing

// DEPRECATED
macro.absorbValue(ndName, bdName, value)  // use setDefaultValue or absorbBinding instead
```

### Node Types (ndType)

`data/in params`, `data/out params`, `data/scalar`, `data/object`, `data/list objects`, `data/list scalars`, `card`, `query`, `decision`, `component`, `action`, `function`

### Binding Types (bdType)

`string`, `number`, `bool`, `date`, `color`, `url`, `image`, `icon`, `ref`, `email`, `card`, `event`, `location`, `data`
Lists: `["ref"]`, `["string"]`, `[["string"]]`
Objects: `{}`, `{"first": "string", "last": "string"}`

### Operator Values

`==`, `!=`, `like`, `>`, `<`, `>=`, `<=`, `exists`, `not exists`, `StringContains`, `ArrayContains`, `ContainsAll`, `ContainsAny`, `MemberOf`

### createNodeFromAst — Function Example (Dec 2025)

```
macro.createNodeFromAst("ai-func", {
  "type": "function",
  "lambda": "fn(x)",
  "helpers": "def fn(x) = x + 1",
  "bindings": [{"name": "x", "type": "number", "value": 0}]
})
```

---

## Edit / Run / Configure Modes

> Source: [Edit / Run / Configure modes](https://www.notion.so/2251d464528f808da28df02fbd305915)
> Parent: Remix Studio User Guide
> Example: `https://remix-dev.remixlabs.com/e/edit/dda-misc/configure_mode`

Three L2 view modes controlling how the nodegraph is presented:

| Mode               | Description                                                          |
|--------------------|----------------------------------------------------------------------|
| **edit** (default) | Regular L2 with all nodes + bindings visible                         |
| **run**            | Screen node shown fullscreen (used by AI to build a screen)          |
| **configure**      | Configurator node shown fullscreen (guided/simpler build experience) |

### Switching Modes

- Via the **top-level menu** in the builder navbar
- Via URL param: `_rmx_mode=<edit | run | configure>`

### Configure Mode Setup

Designate the **configurator node** via the **footer menu of a card** — same mechanism as specifying which node is the screen node.

### Key Properties

- Still a **builder/REPL view** (not runtime) — nodes can be added, bindings connected/modified, etc. (via macros)
- Hides most nodegraph complexity to guide users to a simpler assemble/customize experience
- Navbar allows switching back to Studio or saving the screen as an asset in the desktop library

### Fullscreen Variants

`_rmx_mode=run-fullscreen` or `_rmx_mode=configure-fullscreen` — hides the navbar completely.

**Caution:** Once in fullscreen mode there is no easy way to exit (navbar with mode switcher is not shown).

---

## `_rmx_init` Screen (since PR 1201)

> Source: [screen-rmx-init](https://www.notion.so/1061d464528f81fda16dc22e3f0ab4b0)

If the executable contains a module named `_rmx_init`, it is **activated on session startup** before any other screen. Its output is discarded — it is only for running global app initializations. It
also runs when an agent is invoked.

```
module _rmx_init(p:data, t:data)
  // global initialization actions here
module end
```

Parameters:

- `p.reqName` — name of the initially requested screen or agent
- `p.reqParams` — array of module parameters for the initial request (typically `[paramObject, transitionObject]`)
- `t` — always empty object `{}`

If both `_rmx_init` and `_rmx_entry` are present: `_rmx_init` runs first, then `_rmx_entry`.