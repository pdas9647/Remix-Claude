# Remix Deployment — Methods, Widgets, Self-Hosted Server

> Split from [remix-infra.md](./remix-infra.md).
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

### Running .remix Files

> Source: [Running .remix files](https://www.notion.so/1971d464528f80e5bed2ec2a380d331f)

#### Via remix.app

| Method          | URL Pattern                                    | Notes                       |
|-----------------|------------------------------------------------|-----------------------------|
| **By key**      | `https://remix.app/<collection>/<item>`        | Registered Remix Clip       |
| **By URL**      | `https://remix.app/run?_rmx_url=<.remix URL>` | Any hosted .remix file      |
| **Drag & Drop** | `https://remix.app/run`                        | Drop .remix file in browser |

#### Embedded on a Web Page

Load the Remix runtime directly in any web page:

```html
<link href="https://remix.app/static/rmx-fullscreen.css" rel="stylesheet"/>
<script type="module">
    import {load} from "https://dev.remix.app/js/rmx-app-record.js";

    const appRecord = "about/remix"; // or a full AppRecord object
    load(appRecord, "https://dev.remix.app/static/mixcore.wasm", {});
</script>
```

**Required assets** (can be self-hosted): `rmx-app-record.js`, `rmx-fullscreen.css`, `mixcore.wasm`, `mixrt.wasm`

**Optional:** `firebase-messaging-sw.js` (for Web Push Messaging), host event handler callbacks

**Other embedding:** iframe of remix.app, web-components

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

iOS Widgets and Shortcuts are **special agents** compiled to `executable_standalone` format via a separate static compilation process. They run on a **WASM+Rust runtime** (distinct from the standard Mix VM).

### Widget Agents

Widget records live in the `_rmx_widgets` db. A default agent called `new` takes users to a widget creation screen, which stores `widget` entities with fields: `name` (String), `description`, `database`, `agent`, `sizes` (List of String), `config`.

The widget infrastructure reads these records and invokes the `agent` from `database` with the `config` params.

**In Params:**

| Param | Type | Description |
|-------|------|-------------|
| `size` | String | `small` \| `medium` \| `large` \| `extra-large` (iPad) \| `circular` (lock screen) \| `rectangular` (lock screen) |
| `isPreview` | Bool | Always false except for the default widget in the iOS Widget picker |
| `config` | Object | From the `config` field of the widget record (except for default widget) |
| `widget_id` | Ref | `_rmx_id` of the widget record. Can be passed as URL param to a screen (`ontap` target) to load widget metadata |

**Out Params:**

| Param | Type | Description |
|-------|------|-------------|
| `refresh` | Number (optional) | Unix timestamp (seconds since epoch) when widget should next refresh |
| `timeline` | List of Object (optional) | Array of `{date: Number (unix timestamp), view: Card}` — widget shows each view at its scheduled time |

**Clickable zones** (medium and larger): add a `url` property on a group leaf card element to make it tappable.

### Shortcut Agents

**In Params:**

| Param | Type | Description |
|-------|------|-------------|
| `input` | String | Only used if the agent name contains `_input`; value passed by the user |

**Out Params:**

| Param | Type | Description |
|-------|------|-------------|
| `success` | Bool (required) | Whether the task succeeded. If false, subsequent shortcut actions are skipped |
| `output` | String (required) | Return value if `success=true`, or error message if `success=false` |
| `reloadWidgets` | Bool (optional, default false) | Whether to reload all widgets after running |
