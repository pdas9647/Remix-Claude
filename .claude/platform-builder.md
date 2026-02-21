# Platform Builder — Automation & Runtime Mechanics

> Parent: Remix Documentation > User Guides and Tutorials > Topics
> Split from platform-topics.md — covers builder automation, auth patterns, and runtime interaction.

---

## Mixed Auth

> Source: [Mixed Auth](https://www.notion.so/1741d464528f808cbb5ff371d571c437)
> Sub-page: [Mixed auth test](https://www.notion.so/1b91d464528f802ebf9efe3417f1ea6f) — embedded test previews only (Dev/Beta/Prod), no stable content
> Parent: Remix Studio User Guide

**Key rule:** You need a **different `_rmx_auth`** for fullscreen vs embedded cases.

See also: [Enable anonymous access](https://www.notion.so/14c1d464528f8079b1bcd79c5fdbf4c6) for marking screens as anonymous.

### Full Screen (working)

Use `rmx-app-record` + `rmx-fullscreen` code (as deployed on remix.app).

**`_rmx_auth` setup:**

- A default `_rmx_auth` module is built automatically
- To customize: copy from `https://remix-dev.remixlabs.com/e/edit/builder-examples/_rmx_auth` → edit styling, add other login options, etc.
- Future: `_rmx_auth` screen versions will be in the catalog

### Embedded

Example: `https://remix-dev.remixlabs.com/e/edit/builder-examples/_rmx_auth_1`

### Fullscreen-Embedded Case

Implement mixed auth with `rmx-remix` web component when you own the host page. Host page pattern:

```javascript
<rmx-remix src="http://localhost:3100/r/assets/anon-app.remix"></rmx-remix>
<script type="module">
  import * as auth from "../../shared/assets/auth.js";
  const app = document.querySelector("rmx-remix");
  const urlParams = new URLSearchParams(window.location.search);
  const isUpgradeToken = !!urlParams.get("_rmx_upgrade_token");
  if (window.location.hash.includes("token") && isUpgradeToken) {
    // Handle auth redirect — posts token to browser channel
    auth.handleSignedInRedirect();
} else {
    app.addRemixEventListener("remix/user-signin", (args, cb) => {
      auth.handleUserSignin(null, (token) => cb(token));
    });
}
</script>
```

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

## External Actions

> Source: [External Actions](https://www.notion.so/2161d464528f8014ac9ce7eda3a134ab)
> Parent: Remix Studio User Guide
> **Note:** Moving to **Client Actions** (preferred — better builder support + error handling by default). Ask dev team to convert Host Effects to Client Actions.

Mechanism for the Remix runtime to communicate with the local embedding environment (browser on desktop, or Flutter with native app).

### Pattern

```javascript
rmx_init({flags, node, onEvent});
// onEvent: { actionName: (args, coordinates, cb) => cb(returnValue) }
```

Each external action is an async local function call: identified by name (string), receives params (any type), returns value via `cb`.

In Studio: use `External Actions` action tile at L2.

### Default External Actions

| Name                         | Params                         | Returns                             | Notes                                                                                                                                         |
|------------------------------|--------------------------------|-------------------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------|
| `remix/test`                 | string                         | string (echo)                       | Tests wiring                                                                                                                                  |
| `remix/download-remix-file`  | `{manifest, filename}`         | bool                                | Downloads .remix; opens share sheet on iOS                                                                                                    |
| `remix/download-file`        | `{url, filename}`              | bool                                | Downloads file; supports `data:` URIs                                                                                                         |
| `remix/ipc`                  | `{channel, params}`            | varies                              | Generic inter-process communication (electron + mobile)                                                                                       |
| `remix/logout`               | url? (post-logout destination) | —                                   | Logs out; default → `/_rmx_home/home`                                                                                                         |
| `remix/keyboard`             | list string (key combos)       | string (combo pressed)              | Key listener via ctrl-keys lib. Format: `{modifiers}+{char}`. Blocked in: input/textarea focus, selected text, Monaco editor, contentEditable |
| `remix/check-limits`         | none                           | bool (always true)                  | Shows soft/hard limit toasts/modals                                                                                                           |
| `remix/get-messaging-status` | none                           | "default"\|"rejected"\|token string | Push notification permission status                                                                                                           |

### Mobile External Actions

| Name                                | Params                                     | Returns                                                      | Platform                           |
|-------------------------------------|--------------------------------------------|--------------------------------------------------------------|------------------------------------|
| `remix/mobile/permissions/location` | none                                       | denied\|deniedForever\|whileInUse\|always\|unableToDetermine | Mobile                             |
| `remix/mobile/open_drawer`          | none                                       | —                                                            | Mobile                             |
| `remix/wipedb`                      | url?                                       | —                                                            | Mobile — wipes DB then logs out    |
| `remix/mobile/reload-widgets`       | none                                       | bool                                                         | Mobile                             |
| `remix/mobile/donate-intent`        | `{title, url}`                             | bool                                                         | iOS — Shortcuts/home screen intent |
| `remix/mobile/add-to-siri`          | `{title, url, phrase?}`                    | bool                                                         | iOS — Siri shortcut registration   |
| `remix/mobile/widget-debugger`      | none                                       | bool                                                         | iOS — debug/testing only           |
| `remix/mobile/quicklook-open`       | none                                       | bool                                                         | iOS                                |
| `remix/mobile/share-navbar`         | `{save-enabled, main-title}`               | bool                                                         | iOS — sharesheet navbar config     |
| `remix/mobile/share-close`          | none                                       | bool                                                         | iOS                                |
| `remix/mobile/share-invoke-agent`   | `{db, agent, params (JSON string)}`        | `{success, error}`                                           | iOS                                |
| `remix/mobile/share-render`         | `{scene, filename, width, height, format}` | bool                                                         | iOS — renders card as png/pdf      |
| `remix/mobile/share-widget-render`  | `{id, size, filename, format}`             | bool                                                         | iOS — renders widget as png/pdf    |
| `remix/mobile/write-nfc`            | `{url}`                                    | —                                                            | iOS — writes URL to NFC tag        |
| `remix/mobile/esc-pos-print`        | `{printer url, printer port, commands[]}`  | `{ok}\|{error}`                                              | iOS + Android                      |

**ESC-POS print command types:** `cut`, `feed`, `empty`, `hr`, `beep`, `qrcode`, `image`, `text`, `row`
Row cols: sum of `width` values must equal **12**.

### Local Storage External Actions

Keys namespaced with `rmx-local-data--` prefix.

| Name                              | Params         | Returns                                               |
|-----------------------------------|----------------|-------------------------------------------------------|
| `remix/local-storage/get-item`    | string (key)   | `{tag:"success", value}` \| `{tag:"error", message}`  |
| `remix/local-storage/set-item`    | `{key, value}` | `{tag:"success"}` \| `{tag:"error"}`                  |
| `remix/local-storage/remove-item` | string (key)   | `{tag:"success"}` \| `{tag:"error"}`                  |
| `remix/local-storage/list-keys`   | none           | `{tag:"success", value: [keys]}` (namespace stripped) |

### Payment External Actions

| Name                            | Params            | Returns                                           |
|---------------------------------|-------------------|---------------------------------------------------|
| `remix/store/current-plan`      | none              | `{plan: basic\|monthly\|annual, txDate, expDate}` |
| `remix/store/purchase`          | `monthly\|annual` | bool (whether store popup shown)                  |
| `remix/store/restore-purchases` | none              | bool                                              |

### Unstable (may be removed/changed)

| Name                | Params        | Returns |
|---------------------|---------------|---------|
| `remix/send-beacon` | `{url, data}` | bool    | Browser `navigator.sendBeacon` wrapper |

---

## Edit / Run / Configure Modes

> Source: [Edit / Run / Configure modes](https://www.notion.so/2251d464528f808da28df02fbd305915)
> Parent: Remix Studio User Guide
> Example: `https://remix-dev.remixlabs.com/e/edit/dda-misc/configure_mode`

Three L2 view modes controlling how the nodegraph is presented:

| Mode | Description |
|------|-------------|
| **edit** (default) | Regular L2 with all nodes + bindings visible |
| **run** | Screen node shown fullscreen (used by AI to build a screen) |
| **configure** | Configurator node shown fullscreen (guided/simpler build experience) |

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

If the executable contains a module named `_rmx_init`, it is **activated on session startup** before any other screen. Its output is discarded — it is only for running global app initializations. It also runs when an agent is invoked.

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
