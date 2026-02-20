# OAuth — Third-Party Authentication Flows

> Sources:
> - [OAuth Handler](https://www.notion.so/29a1d464528f807784fcdce13675f665)
> - [Remix Desktop Third Party OAuth](https://www.notion.so/29c1d464528f80b4aba2f4414a2656ce)
> - [Remix Extension Third Party OAuth](https://www.notion.so/29c1d464528f80648125fe390838fe0e)
> - [Remix Widgets Third Party OAuth](https://www.notion.so/29c1d464528f808f8b86cf7a7d860239)
> Parent: Remix Documentation > Remix Catalogs and Toolkits

---

## OAuth Handler (Library Asset)

> Source: [OAuth Handler](https://www.notion.so/29a1d464528f807784fcdce13675f665)

Templated screen for subscribing to the completion of a 3rd-party OAuth sign-in process.

### Screen: `connect_extension`

- **Extension-specific**, customizable sign-in screen
- Takes a **`redirect_screen`** name parameter — the screen to push to after OAuth completes
- After sign-in, opens: `https://remix.app/remix/oauth_handler?redirect_screen=<REDIRECT_SCREEN>`
- The `remix.app` screen publishes the **`oauthredirect`** topic

**Library asset:** [connect_extension](https://remix.app/remix/asset?source=https://agt.remixlabs.com/ws/remix_labs/NLyRdK3X9uv265UQoqwnduoFMlcjuGQA)

**Source app:** `https://remix.remixlabs.com/e/edit/oauth_handler`

---

## Desktop OAuth Flow

> Source: [Remix Desktop Third Party OAuth](https://www.notion.so/29c1d464528f80b4aba2f4414a2656ce)

Uses a **popup window** pattern within the Desktop app.

### Flow

1. User clicks OAuth sign-in button (e.g. HubSpot) while configuring a widget in artifact view
2. Desktop app opens a popup → navigates to `oauth_signin` screen in `_rmx_artifact`
3. `oauth_signin` acts as middleware → redirects popup to 3rd-party OAuth URL
4. User authenticates on 3rd-party platform
5. 3rd party redirects back to `confirmation` screen in `_rmx_artifact`
6. `confirmation` screen:
   - Publishes a message the artifact config view is listening for (sign-in complete)
   - Closes the popup
7. Artifact config view receives completion message → flow complete

### Dependencies

`_rmx_artifact` must include two screens:
- **`oauth_signin`** — middleware redirect to 3rd-party OAuth URLs
- **`confirmation`** — publishes completion message + closes popup

Screens location: `https://remix.remixlabs.com/e/edit/_rmx_artifact`

### Library Assets

- [Desktop oauth component](https://remix.app/remix/asset?source=https://agt.remixlabs.com/ws/remix_labs/EtCExHxVwc8sn9SCT9BrlVXIHQaScs11)
- [Desktop artifact example](https://remix.app/remix/asset?source=https://agt.remixlabs.com/ws/com_remixlabs_wilber/KgPmsZ9ceKz0KFTpY1boeWvQNDSp5Nqw)

---

## Chrome Extension OAuth Flow

> Source: [Remix Extension Third Party OAuth](https://www.notion.so/29c1d464528f80648125fe390838fe0e)

Uses a **hosted web app + service agent** pattern (no intermediate Remix screens).

### Flow

1. User opens a screen requiring 3rd-party auth in Chrome Extension, clicks auth button
2. Extension opens browser popup **directly** to 3rd-party OAuth URL (no intermediate screen)
3. User authenticates on 3rd-party platform
4. 3rd party redirects to **OAuth Handler** at `https://remix.app/remix/oauth_handler`
5. OAuth Handler calls the **publish service agent** in the `remix` workspace on `https://agt.remixlabs.com`
6. Publish service agent sends completion message to Extension
7. Extension screen (listening throughout) receives notification → closes popup

### Dependencies

- **OAuth Handler** — hosted at `https://remix.app/remix/oauth_handler`
- **Publish service agent** — deployed in `remix` workspace at `https://agt.remixlabs.com`

### Library Asset

- [Extension third party oauth component](https://remix.app/remix/asset?source=https://agt.remixlabs.com/ws/remix_labs/ZNZOO7ORm64Fg2cxTzpUEFDXDrG4vAHf)
- Example: search for "third party oauth" → select "Extension third party oauth example"

---

## Mobile Widgets OAuth Flow

> Source: [Remix Widgets Third Party OAuth](https://www.notion.so/29c1d464528f808f8b86cf7a7d860239)

Uses the **device default browser** pattern, orchestrated by the Remix Container App.

### Flow

1. User installs app with widgets → widget shows "Tap to begin sign in"
2. User taps → redirected to Remix Container App
3. Container navigates to `oauth_signin` screen in `_rmx_widgets`
4. `oauth_signin` opens OAuth URL in device default browser
5. Container simultaneously navigates to home screen in `_rmx_home`
6. User authenticates in browser on 3rd-party platform
7. Browser redirects back to Container App → `confirmation` screen in `_rmx_widgets`
8. `confirmation` reloads all widgets (so they can use the new auth token)
9. `confirmation` navigates to home screen in `_rmx_home`
10. User navigates to widget screen → widgets now function with authenticated token

### Dependencies

`_rmx_widgets` must include two screens:
- **`oauth_signin`** — opens OAuth URL in device browser
- **`confirmation`** — reloads widgets + navigates to home

Screens location: `https://remix.remixlabs.com/e/edit/_rmx_widgets`

### Library Asset

- [Widget oauth url component](https://remix.app/remix/asset?source=https://agt.remixlabs.com/ws/remix_labs/Vde4BkQyPreZaM4k2wYEhcnk2pQTU2Uu)

---

## Flow Comparison

| Aspect | Desktop | Chrome Extension | Mobile Widgets |
|--------|---------|-----------------|----------------|
| Auth trigger | Button in artifact config | Button in extension screen | Tap on widget |
| Popup/browser | Popup window | Browser popup | Device default browser |
| Intermediate screen | `oauth_signin` in `_rmx_artifact` | None (direct to 3rd party) | `oauth_signin` in `_rmx_widgets` |
| Callback handler | `confirmation` in `_rmx_artifact` | `remix.app/remix/oauth_handler` → publish agent | `confirmation` in `_rmx_widgets` |
| Notification method | Publishes message directly | Service agent publishes message | Reloads all widgets |
| Container app | `_rmx_artifact` | N/A | `_rmx_widgets` + `_rmx_home` |
