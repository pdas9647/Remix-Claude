# Platform Builder — Host & Runtime Integration

> Parent: Remix Documentation > User Guides and Tutorials > Topics
> Split from platform-builder.md — covers Mixed Auth and External Actions (host/runtime integration).

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