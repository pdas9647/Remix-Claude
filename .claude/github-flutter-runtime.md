# flutter-runtime

**Repository:** https://github.com/remixlabs/flutter-runtime
**Stack:** Flutter 3.32.7 / Dart 3.x (app), Kotlin (Android native + widgets), Swift (iOS native + WidgetKit + AppClip + extensions), Go (Ampmobile embedded server), Rust (MixRun/MixCore Wasm runtime
via FFI), Gradle/CocoaPods (build), Fastlane (deploy), CircleCI (CI/CD), RCM (dependency manager)
**Purpose:** Cross-platform mobile runtime for Remix apps — downloads, installs, and runs .remix applications locally on iOS and Android, with native home-screen widgets, AppClip support, push
notifications, in-app purchases, and offline-capable Wasm execution.
**Explored layers:** L1–L3 selective (~75 files). L4 (build scripts) and L5 (send_sms plugin) skipped.

## Build & Deploy

- **Version:** 1.1.2615+2615 (auto-incremented via `make increment-build-number`)
- **Bundle ID:** `com.remixlabs.runtime`, app group `group.com.remixlabs.runtime`
- **Targets:** iOS 13+ / Android SDK 30 (API 30), compileSdk 36
- **RCM deps** (`rcm.config.mjs`): `amp` (Go AMP server), `mixcore` (Rust FFI bridge), `mixrun` (Rust Wasm VM), `turntable` (JS runtime `rmx-runtime.js`)
- **CI:** CircleCI — `build_ios` (macOS m4pro, Xcode 26.0.1), `build_android` (Docker), `update_users` (post-merge to main). Flutter 3.32.7 pinned.
- **Deploy:** Fastlane → TestFlight (iOS) / Play Store internal track (Android). Web assets uploaded to GCS (`gs://rmx-static/flutter-assets/<version>/web/`).
- **Makefile targets:** `sync_all` (sync iOS+Android frameworks), `ios_build_and_upload_ci`, `android_build_and_upload_ci`, `android_sms_apk` (special SMS-permission build), `check` (16KB ELF
  alignment for Android 64-bit .so files)
- **Native lib sync:** MixRun/MixCore bindings copied from RCM install prefix → `MixRun.swift`/`MixCore.swift` (iOS), `MixRun.kt`/`MixCore.java` (Android). JNI .so files for
  arm64-v8a/armeabi-v7a/x86_64. iOS xcframeworks for mixrun/mixcore/Ampmobile.

## Architecture

### App Flow

```
main() → Config.loadFromPrefs() → MyApp (MaterialApp)
  → TopLevel.FIRST → FirstScreen
    - Validates JWT token (expiry check)
    - Starts local AMP server (Ampmobile Go lib on localhost:8000)
    - Downloads/syncs .remix apps via _rmx_sync/get_apps agent
    - → TopLevel.APP → AppScreen (or APPCLIP → AppClipScreen)
  → TopLevel.LOGIN → LoginScreen (WebView → remix.app/remix/signin → FlutterWebAuth2)
```

### Core Components

**Config** (`lib/models/config.dart`) — Central god object. Manages: auth tokens (JWT, stored in SecurePrefs/Keychain), screen routing (TopLevel enum), AMP/Mixer clients, AppLoader, StoreMgr,
LimitMgr, NotifMgr. Handles all incoming links (deep links, universal links, AppClip links, .remix files, QuickLook).

**AppScreen** (`lib/screens/app_screen.dart`) — WebView that loads turntable's runtime. HTML served as string with `<base href="${Config.assetsUrl}/">`, loading `common.js`, `app.js`, `rmx-runtime.js`
from GCS. Communicates via `RmxPost` JavaScriptChannel. On `start` message, injects `initApp()` with app name, screen, params, ampPrefix, token, sync channel, plan, env (backendBaseURL,
frontendBaseURL, auth_server, auth_workspace).

**MixAction** (`lib/network/mix_action.dart`) — Dispatches ~30+ client actions from the WebView runtime:

- Navigation: `remix/location-changed`, `remix/logout`, `remix/wipedb`, `remix/user-signin`
- Device: `remix/get-location`, `remix/copy-clipboard`, `remix/web/share`, `remix/bg-color-changed`
- Mobile-specific: `remix/mobile/send-sms`, `remix/mobile/esc-pos-print`, `remix/mobile/write-nfc`, `remix/mobile/play-alert`, `remix/mobile/open-drawer`, `remix/mobile/reload-widgets`,
  `remix/mobile/widget-debugger`, `remix/mobile/share-render`, `remix/mobile/share-widget-render`, `remix/mobile/donate-intent`, `remix/mobile/add-to-siri`
- Store: `remix/store/purchase`, `remix/store/restore-purchases`, `remix/store/current-plan`
- System: `remix/download-remix-file`, `remix/track-error`, `remix/fatal-error`, `remix/save-token`, `remix/scenegraph`, `remix/ipc`

**AmpConfig** (`lib/models/amp_config.dart`) — URL model for Remix apps. Key prefixes:

- `http://localhost:8000` — local AMP server
- `https://community.remixlabs.com/a` — community server
- `https://auth.remixlabs.com/a` — auth server
- `https://app.remixlabs.com` — universal link origin
- `com.remixlabs.runtime://` — custom URL scheme
- `https://remix.app/` / `https://dev.remix.app/` / `https://beta.remix.app/` — AppClip links
- Special DBs: `_rmx_home` (home app), `[rmx_appclip]` (fake AppClip DB), `_rmx_notifications` (background actions), `_rmx_widgets` (widget config)

### Platform Channel Bridge (`com.remixlabs.channel`)

Both iOS (`AppDelegate.swift`) and Android (`MainActivity.kt`) implement the same MethodChannel:

- **AMP lifecycle:** `ampmobile_start` (start Go server with user/token/dbDir/versionJSON), `ampmobile_stop`, `ampmobile_reset`, `ampmobile_wipedb`, `ampmobile_store_token`
- **Agent execution:** `execute_agent` — runs Mix agents locally via MixRun (Wasm)
- **Widgets:** `reload_widgets`, `widget_debugger`, `share_widget_render`, `share_render`
- **Files:** `download_remix_file`, `open_document`
- **iOS-only:** `send_sms` (MFMessageComposeViewController), `write_nfc`, `donate_intent`, `add_to_siri`, `scoped_resource_start/stop`, `download_with_coordinator`, `open_location` (embedded
  WKWebView)
- **Android-only:** `enableAndroidTapJacking`, `open_location` (EmbeddedWebViewActivity)

### Home-Screen Widgets

**iOS (WidgetKit + SwiftUI):** 7 sizes (small/medium/large/extraLarge/circular/rectangular + inline). Each backed by `IntentTimelineProvider`. Widget agent executed locally via `executeWidget()` (
MixRun Wasm). Returns `RemixAgentResponse` with `DomNode` scene graph → rendered natively via `NodeViewSwift` (SwiftUI). Supports timeline entries for time-based updates. Global styles fetched from
local DB. Images cached. Config stored per widget via `RemixConfig` intent.

**Android (WebView → Bitmap):** 3 sizes (Small/Medium/Large widget classes). Uses `WebView` to load `widget.html`, executes widget agent via `MixRun.instance.executeWidget()`, renders in WebView,
captures bitmap screenshot, overlays transparent click-target buttons (`btn1`–`btn10`). Click → `PendingIntent` → `ACTION_VIEW` opens URL. Widget config persisted via SharedPreferences.

**Shared pattern:** Both platforms read email/token from secure storage, execute the `_rmx_widgets` agent locally (no network needed after initial sync), and render the DomNode scene graph the agent
returns.

## Data Model

- **SecurePrefs** (iOS: Keychain + app group, Android: EncryptedSharedPreferences) — stores JWT token, email, profile, org server/workspace, sync channel, download info map, purchases, appclips map,
  home URL, limits
- **DownloadInfo** — per .remix app: URL + lastModified (for conditional GCS fetches via If-Modified-Since)
- **RmxApps** — from `_rmx_sync/get_apps` agent: list of RmxApp (label, database, module, url) + homes. Apps can be `DOT_REMIX_REMOTE` (GCS) or `DOT_REMIX_LOCAL` (bundled asset)
- **SyncChannel** — DRAFT / TEST / PUBLIC (Remix employees default to DRAFT, others to PUBLIC)
- **StoreMgr** — In-app purchases: Basic / Monthly ($4.99) / Annual ($50). iOS App Store + Android Play Store. Hard limit: 3000 records (free plan)

## Key Patterns & Conventions

- **Embedded local server:** AMP (Go) runs on localhost:8000, provides full backend (DB, agents, file management). MixRun (Rust/Wasm) executes Mix code locally for widgets and background agents.
- **WebView as runtime:** Main app UI is turntable's web runtime loaded in a platform WebView. Flutter is the shell; the actual Remix app UI is web-based.
- **JS↔Native bridge:** `RmxPost` channel (Dart side) and `com.remixlabs.channel` MethodChannel (native side). All callbacks use `{status: "ok"/"error", data: ...}` envelope.
- **Conditional sync:** .remix files downloaded from GCS with If-Modified-Since headers. Background sync every 60 minutes when app resumes.
- **App lifecycle management:** On resume, checks if AMP is still running. If not, restarts it and reloads the current screen. Sleep >4 min previously triggered full restart (now commented out).
- **Remote logging:** Optional remote log to `community.remixlabs.com/a/flutter-logger/agents/log` (enabled per-user or via `forceRemoteLog` flag).
- **Widget rendering divergence:** iOS renders natively (SwiftUI from DomNode AST), Android renders via hidden WebView bitmap capture. Both execute agents via MixRun locally.

## Open Questions

- **Dom.swift / NodeViewSwift.swift / ViewModifiers.swift / Patch.swift / Val.swift** — iOS widget SwiftUI rendering layer (DomNode AST → SwiftUI views). Large files (~36KB Dom.swift, ~17KB
  ViewModifiers.swift) not read. Would reveal the full iOS native rendering pipeline.
- **printer_utils.dart** (~47KB) — ESC/POS thermal printer command generation. Skipped due to size.
- **main_drawer.dart** (~27KB) — Side drawer with debug tools, channel switching, profile, etc. Skipped.
- **store_mgr.dart** — Read fully. Purchase verification currently returns `true` always (receipt validation commented out).
- **AppClip** — iOS AppClip uses a native `UiKitView` (`appclip-view`) registered via `FLAppClipViewFactory`. The AppClip web content is bundled as `appclip.zip` (html + rmx-runtime.js). Android falls
  back to external browser.
- **send_sms plugin** — Custom Flutter plugin at `send_sms/` for Android background SMS sending. Not explored.
- **MixCore vs MixRun** — Both are Rust FFI bindings. MixCore appears to handle DB-level operations (`getCodes`), while MixRun handles Wasm execution (`executeAgent`, `executeWidget`). Exact division
  of responsibility unclear without reading generated binding files.
