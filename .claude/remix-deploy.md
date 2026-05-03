# Remix Deployment — Methods, Widgets, Self-Hosted Server

> Originally part of remix-infra; see also [remix-infra-auth.md](./remix-infra-auth.md) and [remix-infra-setup.md](./remix-infra-setup.md).
> Parent Notion pages linked per section below.

---

## Deploying Remix Applications

> Source: [Deploying Remix applications](https://www.notion.so/11b1d464528f8030a107dc7f5e8d27cc)
> Parent: Remix Documentation > User Guides and Tutorials

### Deployment Methods (10)

1. Flow in the Remix container app on iOS and Android
2. iOS or Android widgets
3. Full screen web page
4. Embedded in someone else's web page
5. Flows in the Remix Chrome Extension
6. iOS native app clip
7. Cloud agents
8. ISV — your own mobile app with a Remix SDK
9. Embed in other ecosystems
10. Embedded widget in a web portal

### Self-Hosted Remix Server

> Source: [Self-hosted Remix Server](https://www.notion.so/22a1d464528f8019be54edf3f3b1f576)

A monolithic, self-hosted deployment option for Remix apps and backend services.

- **Architecture:** Cloud Agent Server (mixer) extended with self-contained authentication
- **Distribution:** Docker image via AWS AMI Marketplace (listing in development), Snowflake Marketplace (Snowpark Container Service), or directly for other managed container runtimes
- **Requirements:** OAuth/OIDC auth provider config + attached persistent disk for internal DB
- **No other external dependencies**
- **File hosting:** Attached disk or any S3-compatible backend (e.g. Cloudflare R2)
- **Capabilities:** Agent execution runtime + static file host for Remix web apps + Remix Studio builder (coming soon)

---

### .remix File Format V2

> Source: [Remix Files V2](https://www.notion.so/21b1d464528f8043a176cfea87324ba5)

Goals: simplify creation/reading/transformation of .remix files from regular Mix code, easier DB name changes, store executables in files (not DB records), cleaner update semantics.

**Manifest v2.1** (current) — path: `manifest_v2.json`. Single-app format (v2.0 supported multi-app):

```json
{
  "version": "2.1",
  "id": "<random>",
  "updates": "",
  "name": "app1",
  "app": {
    "recordsets": [
      "rs1"
    ],
    "filesets": [
      "fs1"
    ],
    "metadata": {},
    "platforms": [
      "web"
    ],
    "surfaces": [
      "app"
    ]
  },
  "created": {
    "timestamp": "...",
    "rmxUser": "...",
    "buildHost": "...",
    "recipe": "..."
  }
}
```

**`updates` semantics:** `""` or missing = delete existing apps first; `"*"` = install on any existing or fresh; `"**"` = must install on existing; `"<id>"` = only on matching .remix ID.

**`appMeta.json`** — path: `apps/<appName>/appMeta.json`. Usual contents but **no `name` field** (added back on import).

**`runtime.json`** — path: `apps/<appName>/runtime.json`. Usual contents but **no `dbName` field** (added back on import).

**Recordsets** — path: `recordsets/<rsName>/rsMeta.json` + `recordsets/<rsName>/*.json` (arrays of JSON records).

```json
{
  "onUpdateDelete": "entity == \"company\"",
  "saveOptions": [
    "upsert"
  ],
  "representationType": "JSON",
  "contentType": "userRecords"
}
```

`onUpdateDelete`: query string or AST — executed before install (on update) to delete matching records. `representationType`: `JSON` (supported) or `db-image` (planned). `contentType`: `userRecords`or
`builderAssets`.

**Filesets** — path: `filesets/<fsName>/fsMeta.json` + `filesets/<fsName>/root/<path>/<file>`.

```json
{
  "onUpdateDelete": [
    "/dir1/",
    "/dir2/",
    "/some_file",
    "*.def"
  ],
  "code": false
}
```

`onUpdateDelete` patterns: `/path/` = delete dir hierarchy; `/path` = delete matching files; `name` (no `/`) = match base name. Supports glob: `?`, `*`, `**`, `[chars]`, `{alt1,alt2}`. `code: true` =
fileset contains executable binaries.

**Executable files** — reserved path `/code` inside a fileset. Contains `.mixlib`, `.mixlib.dbg`, `.mixexe`, `.mixexe.dbg` files. Libraries not needed by any executable are auto-deleted on update.

---

### Running .remix Files

> Source: [Running .remix files](https://www.notion.so/1971d464528f80e5bed2ec2a380d331f)

#### Via remix.app

| Method          | URL Pattern                                   | Notes                       |
|-----------------|-----------------------------------------------|-----------------------------|
| **By key**      | `https://remix.app/<collection>/<item>`       | Registered Remix Clip       |
| **By URL**      | `https://remix.app/run?_rmx_url=<.remix URL>` | Any hosted .remix file      |
| **Drag & Drop** | `https://remix.app/run`                       | Drop .remix file in browser |

---

### `rmx-remix` Web Component Embedding

> Source: [Embedding a .remix into a webpage](https://www.notion.so/33a1d464528f80c7b9cdd44e7c2baa0f)
> Prod: `https://remix.app/js/rmx-remix.js` | Dev: `https://dev.remix.app/js/rmx-remix.js`

Simplest way to embed a `.remix` into any webpage. For full-screen performance, prefer the [full-screen stack](https://www.notion.so/1ae1d464528f80f88f23e4dd46fa3464).

**Attributes:**

| Attribute     | Type         | Required | Description                                          |
|---------------|--------------|----------|------------------------------------------------------|
| `src`         | string       | Yes      | URL of the `.remix` file                             |
| `screen-name` | string       | No       | Screen to display (defaults to home screen)          |
| `params`      | JSON string  | No       | Params to pass to the first screen                   |
| `no-shadow`   | boolean flag | No       | Disable shadow DOM                                   |

**Single embed:**

```html

<script src="https://remix.app/js/rmx-remix.js" type="module"></script>
<rmx-remix src="https://agt.files.remix.app/iaEj4QYboi/_rmx_files/apps/embed.remix"></rmx-remix>
```

**With params:**

```html

<rmx-remix
        src="https://agt.files.remix.app/iaEj4QYboi/_rmx_files/apps/embed.remix"
        params='{"first":"john","last":"doe"}'
></rmx-remix>
```

**Multiple on one page:**

```html

<rmx-remix src="...embed.remix" params='{"first":"john"}'></rmx-remix>
<rmx-remix src="...schedule.remix" screen-name="home" params='{"entity":"contact","phone":"+1..."}'></rmx-remix>
```

### Remix Runtime Library (RRL)

> Source: [The Remix Runtime Library (RRL)](https://www.notion.so/3451d464528f80fc9317e827b1378902)
> Template: https://rrl-template.pages.dev/ | GitHub: https://github.com/remixlabs/template-website

The RRL (`rmx-app-record.js`) is the JavaScript interface to Remix's Runtime. Used by remix.app, Desktop, legacy Amp instances, and the Elm dev environment. Supports embedded or full-screen display of
`.remix` files. **Alternative for simpler cases:** the `rmx-remix` web component above.

#### Entry Points (`rmx-app-record.js`)

| Function         | Mode        | Signature                                              |
|------------------|-------------|--------------------------------------------------------|
| `loadAppRecord`  | Full-screen | `loadAppRecord(appRecord, flags, eventOverrides)`      |
| `loadRemixUrl`   | Full-screen | `loadRemixUrl(url, flags, eventOverrides)`             |
| `embedAppRecord` | Embedded    | `embedAppRecord(appRecord, id, flags, eventOverrides)` |
| `embedRemixUrl`  | Embedded    | `embedRemixUrl(url, id, flags, eventOverrides)`        |

- `appRecord` can be a string key (e.g. `"about/remix"`) or a full `AppRecord` object with `key`, `remix_url`, `title`, `constants`, `params`
- `id` is the DOM element id to mount the embedded runtime into

**Full-screen example:**
```html
<!doctype html>
<head>
    <link rel="stylesheet" href="rmx-fullscreen.css" crossorigin/>
</head>
<body>
<script type="module">
    import {loadRemixUrl} from "./rmx-app-record.js";

    loadRemixUrl("https://agt-dev.files.remix.app/simon/files/corporate.remix", {}, {});
</script>
</body>
```

**Embedded example:**

```html

<div id="uid-remix"></div>
<script type="module">
    import {embedAppRecord} from "/runtime/rmx-app-record.js";

    const appRecord = {
        key: "",
        remix_url: "https://agt-dev.files.remix.app/simon/files/corporate.remix",
        title: "My Embedded Remix",
        constants: {company: "Salesforce"},
        params: {subtitle: "You need us!"},
    };
    embedAppRecord(appRecord, "uid-remix", {}, {});
</script>
```

**For DB-stored apps** (Desktop preview): use `rmx-init-runtime.js` → `initRuntime(flags, "", eventOverrides)`. Loads the runtime directly; used internally by the other entry points. (TypeScript
typings pending mixcore resolution.)

#### Flags (all optional)

| Flag           | Description                                                   |
|----------------|---------------------------------------------------------------|
| `assetsPrefix` | Prefix for platform (rcm) assets (default: `location.origin`) |
| `mixerHost`    | Mixer host (for Desktop / SPCS mode)                          |
| `workspace`    | Mixer workspace                                               |
| `dbName`       | Mixer DB name                                                 |
| `screenName`   | Mixer screen name                                             |
| `params`       | Params to pass to the first screen                            |
| `mixcore`      | Override mixcore location                                     |
| `debug`        | `true` → extra debugging output; forces mix error page        |

#### Event Overrides

Handlers for Client Actions. Platform provides defaults; pass `{}` in normal use. Used to implement Custom Client Actions.

#### Query Params

| Param                  | Description                                       |
|------------------------|---------------------------------------------------|
| `_rmx_host`            | Sets mixer host                                   |
| `_rmx_ws`              | Sets mixer workspace                              |
| `_rmx_base`            | Sets Amp base (legacy)                            |
| `_rmx_debug`           | Turns debug mode on; forces mix error page        |
| `_rmx_mixc`            | Overrides compiler location                       |
| `_rmx_debugChannel`    | Sets up VM debug channel (untested)               |
| `_rmx_anon=true`       | Prevents passing user token to runtime (untested) |
| `_rmx_vm=server`       | Use server VM via Amp (default)                   |
| `_rmx_vm=client`       | Use client-side Wasm VM (no Amp server VM)        |
| `_rmx_vm=client:<URL>` | Run Wasm app at URL with no Amp session (legacy)  |

#### Required Directory Structure (self-hosted)

```
/rcm/
  mixrt.wasm
  mixcore-opfs.wasm
  mixcore_bg.wasm
  mixcore.js
/runtime/js/
  rmx-remix.js          # required for auth
rmx-app-record.js       # entry point (can be anywhere)
rmx-fullscreen.css      # fullscreen stylesheet (can be anywhere)
```

**Optional:** `firebase-messaging-sw.js` (for Web Push Messaging), host event handler callbacks

---

### Creating iOS and Android Widgets

> Source: [Creating iOS and Android widgets in Remix](https://www.notion.so/1191d464528f8025a11be59d14e12ffd)

Step-by-step tutorial (also has video walkthrough on the Notion page).

#### 1. Create the Widget

- L1 → `New` → select **Widget** → name it
- Enter L2 → out-params has a `refresh` value (in seconds, minimum 15 minutes)
- Set widget size via In Params: **small**, **medium**, or **large**
- Build the screen as normal → Save → Publish from L1

#### 2. Configure the Widget

- L1 → 3 dots menu on widget → **Agent info**
- Set: display name, color, icon → Save
- Click **New Widget Config** → set display name, surfaces (iOS/Android), size(s), category → Save

#### 3. Package the .remix

- Profile image (top right) → **Tools and Settings**
- Fill in project info → **Download Package** → **Package and Download**

#### 4. Use the Widget

- Install latest Remix mobile app
- Install the .remix file on phone
- Add the Remix widget (same as any native widget) → select size → edit → choose widget

---

## iOS Widget Debugging — Dynamic Widget

> Source: [iOS widget debugging/testing](https://www.notion.so/2091d464528f806bb14ffa0bb037851b)

iOS widgets simulate HTML/CSS rendering (imperfect emulation — must test on device).

**Fast workflow** (vs traditional build→airdrop→install cycle):

1. Tweak widget in builder
2. Use "save as dyn widget" component (from `remix-libraries` library) to invoke save action
3. On device: add dynamic widget to home screen or use widget debugger (right-swipe menu)

- Dyn widget component: [library asset](https://remix.app/remix/asset?source=https://agt.remixlabs.com/ws/remix-libraries/E0SwIiws4GFxl1NSycxgG6F9TchlLEEF)
- Widget debugger app: `https://storage.googleapis.com/rmx-static/draft/dyn-widgets.remix` (auto-synced now)

---

## iOS Widget & Shortcut Agents

> Source: [iOS Widget & Shortcut Agents](https://www.notion.so/2161d464528f803f867fe9bc1d4b61a6)
> Parent: Remix Studio User Guide
> Runtime details: [mix-rs-runtime-for-shortcuts-and-widgets](https://github.com/remixlabs/turntable/wiki/mix-rs-runtime-for-shortcuts-and-widgets)

iOS Widgets and Shortcuts are **special agents** compiled to `executable_standalone` format via a separate static compilation process. They run on a **WASM+Rust runtime** (distinct from the standard
Mix VM).

### Widget Agents

Widget records live in the `_rmx_widgets` db. A default agent called `new` takes users to a widget creation screen, which stores `widget` entities with fields: `name` (String), `description`,
`database`, `agent`, `sizes` (List of String), `config`.

The widget infrastructure reads these records and invokes the `agent` from `database` with the `config` params.

**In Params:**

| Param       | Type   | Description                                                                                                       |
|-------------|--------|-------------------------------------------------------------------------------------------------------------------|
| `size`      | String | `small` \| `medium` \| `large` \| `extra-large` (iPad) \| `circular` (lock screen) \| `rectangular` (lock screen) |
| `isPreview` | Bool   | Always false except for the default widget in the iOS Widget picker                                               |
| `config`    | Object | From the `config` field of the widget record (except for default widget)                                          |
| `widget_id` | Ref    | `_rmx_id` of the widget record. Can be passed as URL param to a screen (`ontap` target) to load widget metadata   |

**Out Params:**

| Param      | Type                      | Description                                                                                           |
|------------|---------------------------|-------------------------------------------------------------------------------------------------------|
| `refresh`  | Number (optional)         | Unix timestamp (seconds since epoch) when widget should next refresh                                  |
| `timeline` | List of Object (optional) | Array of `{date: Number (unix timestamp), view: Card}` — widget shows each view at its scheduled time |

**Clickable zones** (medium and larger): add a `url` property on a group leaf card element to make it tappable.

### Shortcut Agents

**In Params:**

| Param   | Type   | Description                                                             |
|---------|--------|-------------------------------------------------------------------------|
| `input` | String | Only used if the agent name contains `_input`; value passed by the user |

**Out Params:**

| Param           | Type                           | Description                                                                   |
|-----------------|--------------------------------|-------------------------------------------------------------------------------|
| `success`       | Bool (required)                | Whether the task succeeded. If false, subsequent shortcut actions are skipped |
| `output`        | String (required)              | Return value if `success=true`, or error message if `success=false`           |
| `reloadWidgets` | Bool (optional, default false) | Whether to reload all widgets after running                                   |
