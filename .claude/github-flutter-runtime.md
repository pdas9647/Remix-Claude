# flutter-runtime

**Repository:** https://github.com/remixlabs/flutter-runtime
**Stack:** Flutter 3.32.7 / Dart 3.x, Kotlin (Android + widgets), Swift (iOS + WidgetKit + AppClip), Go (Ampmobile), Rust (MixRun/MixCore FFI), Fastlane, CircleCI
**Purpose:** Cross-platform mobile runtime — downloads/installs/runs .remix apps locally with native widgets, AppClip, push, IAP, offline Wasm execution
**Explored:** L1–L3 (~75 files)

## Build & Deploy

- Version: 1.1.2615+2615 | Bundle: `com.remixlabs.runtime` | iOS 13+ / Android SDK 30+
- RCM deps: `amp` (Go server), `mixcore` (Rust FFI), `mixrun` (Rust Wasm VM), `turntable` (rmx-runtime.js)
- CI: CircleCI — `build_ios` (macOS m4pro, Xcode 26.0.1), `build_android` (Docker). Flutter 3.32.7 pinned.
- Deploy: Fastlane → TestFlight / Play Store internal. Web assets → GCS `gs://rmx-static/flutter-assets/<ver>/web/`
- Native libs: MixRun/MixCore copied from RCM → Swift/Kotlin bindings. JNI .so for arm64-v8a/armeabi-v7a/x86_64. iOS xcframeworks.

## Architecture

```
main() → Config.loadFromPrefs() → MyApp
  → FirstScreen: validate JWT → start AMP (localhost:8000) → sync .remix via _rmx_sync → AppScreen
  → LoginScreen: WebView → remix.app/remix/signin → FlutterWebAuth2
```

**Config** (`lib/models/config.dart`) — Central god object. Auth tokens (JWT in Keychain/EncryptedSharedPrefs), routing (TopLevel enum), AMP/Mixer clients, AppLoader, StoreMgr, NotifMgr. Handles deep links, universal links, AppClip links, .remix files.

**AppScreen** — WebView loading turntable runtime. HTML served as string with `<base href>` to GCS assets. JS↔Native via `RmxPost` channel. On `start`, injects `initApp()` with app/screen/params/token/env.

**MixAction** — Dispatches ~30+ client actions from WebView: navigation (location-changed, logout), device (location, clipboard, share), mobile-specific (SMS, ESC/POS print, NFC, play-alert, drawer, reload-widgets, widget-debugger, share-render, donate-intent, add-to-siri), store (purchase, restore, current-plan), system (download-remix-file, track-error, save-token, scenegraph, ipc).

**Platform Channel** (`com.remixlabs.channel`) — iOS (AppDelegate.swift) + Android (MainActivity.kt): `ampmobile_start/stop/reset/wipedb`, `execute_agent` (MixRun Wasm), `reload_widgets`, `download_remix_file`. iOS-only: `send_sms`, `write_nfc`, `donate_intent`. Android-only: `enableAndroidTapJacking`.

## Home-Screen Widgets

**iOS (WidgetKit + SwiftUI):** 7 sizes. `IntentTimelineProvider` executes widget agent locally via MixRun Wasm → `DomNode` scene graph → native SwiftUI rendering (`NodeViewSwift`). Timeline entries for time-based updates. Config via `RemixConfig` intent.

**Android (WebView → Bitmap):** 3 sizes. WebView loads `widget.html`, executes agent via MixRun, renders in WebView, captures bitmap. Click-target buttons (`btn1-10`) → `PendingIntent` → URL. Config in SharedPreferences.

**Shared:** Both read email/token from secure storage, execute `_rmx_widgets` agent locally (no network after initial sync).

## Data Model

- **SecurePrefs:** JWT token, email, profile, org server/workspace, sync channel, downloads, purchases
- **RmxApps:** From `_rmx_sync/get_apps` — label, database, module, url. Sources: `DOT_REMIX_REMOTE` (GCS) or `DOT_REMIX_LOCAL` (bundled)
- **SyncChannel:** DRAFT / TEST / PUBLIC (Remix employees default DRAFT)
- **StoreMgr:** Basic / Monthly ($4.99) / Annual ($50). Hard limit: 3000 records (free). Receipt validation currently returns `true` always.
- **Conditional sync:** GCS If-Modified-Since headers. Background sync every 60 min on resume.

## Key Patterns

- AMP (Go) on localhost:8000 = full backend; MixRun (Rust/Wasm) = local agent/widget execution
- Flutter is shell; actual Remix UI is web-based in WebView
- All JS↔Native callbacks use `{status: "ok"/"error", data: ...}` envelope
- Widget rendering divergence: iOS = native SwiftUI from DomNode AST; Android = WebView bitmap capture

## Recent PRs

| PR   | Date   | What                                                      | Who    |
|------|--------|-----------------------------------------------------------|--------|
| #441 | Feb 20 | Fix deeplink path: `/open-remix-file` → `/run-remix-file` | Didier |
| #440 | Feb 6  | Back to mix-rs main branch                                | Didier |
| #439 | Feb 6  | Fixed widgets                                             | Didier |
| #438 | Feb 6  | Reformatting Kotlin files                                 | Didier |
| #437 | Jan 19 | Dynamic AppClip                                           | Didier |
