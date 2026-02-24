# flutter-runtime — Exploration Plan

**Repository:** https://github.com/remixlabs/flutter-runtime
**Branch:** main
**Stack:** Flutter/Dart (app), Kotlin (Android native + widgets), Swift (iOS native + extensions), Gradle (Android build), Xcode/CocoaPods (iOS build), Fastlane (CI deploy), CircleCI (CI/CD), RCM (Remix Component Manager), shell scripts (build tooling)
**Structure style:** Mobile app monorepo — Flutter cross-platform app with heavy native platform layers (Android home-screen widgets in Kotlin, iOS WidgetKit + AppClip + Share/QuickLook/Intent extensions in Swift), plus an embedded Flutter plugin (`send_sms`)
**Total files:** 324 (439 items including directories)

## Layer 1 — Skeleton & Overview (must read)
Top-level manifests, CI/CD, build config, README — understand what the app is, how it's built, and what versions it targets.

- `README.md`
- `pubspec.yaml` (Flutter deps, app metadata)
- `Makefile`
- `.circleci/config.yml`
- `.circleci/ensure-gcloud-creds-figly`
- `.metadata`
- `.gitignore`
- `rcm.config.mjs` (Remix Component Manager config)
- `rcm-lock.json`
- `mixrs.version`
- `mixrun.version`
- `Gemfile` (Ruby deps for Fastlane)
- `android/build.gradle` (root Android build)
- `android/settings.gradle`
- `android/gradle.properties`
- `android/gradle/wrapper/gradle-wrapper.properties`
- `android/app/build.gradle` (app-level Android build)
- `ios/Podfile`
- `ios/Runner.xcodeproj/project.pbxproj` (Xcode project — skim structure, don't deep-read)
- `ios/Runner/Info.plist`
- `ios/Runner/Runner.entitlements`

**Estimated: ~21 files, ~2000 lines**

## Layer 2 — Entry Points & Core App Logic (high priority)
Flutter Dart code — the main app, screens, navigation, auth, networking, models. This is the heart of the app.

- `lib/main.dart` (app entry point)
- `lib/consts.dart` (constants/config)
- `lib/native.dart` (platform channel bridge)
- `lib/auth/login_utils.dart`
- `lib/screens/first_screen.dart`
- `lib/screens/login_screen.dart`
- `lib/screens/app_screen.dart`
- `lib/screens/appclip_screen.dart`
- `lib/models/config.dart`
- `lib/models/amp_config.dart`
- `lib/models/store_mgr.dart`
- `lib/models/notification_mgr.dart`
- `lib/models/limit_mgr.dart`
- `lib/network/amp.dart` (AMP server communication)
- `lib/network/mixer.dart` (mixer/agent server communication)
- `lib/network/app_loader.dart` (app download/loading)
- `lib/network/mix_action.dart` (action dispatch)
- `lib/data/rmx_apps.dart`
- `lib/data/sync_channel.dart`
- `lib/data/secure_prefs.dart`
- `lib/data/unsecure_prefs.dart`
- `lib/data/push_notification.dart`
- `lib/data/download_info.dart`
- `lib/data/record_summary.dart`
- `lib/data/appclip_config.dart`
- `lib/data/extensions.dart`
- `lib/data/track_error.dart`
- `lib/utils.dart`
- `lib/ui_utils.dart`
- `lib/debouncer.dart`
- `lib/printer_utils.dart`
- `lib/utils/display_exception.dart`
- `lib/widgets/main_drawer.dart`
- `lib/widgets/loading.dart`
- `lib/widgets/failure.dart`
- `lib/widgets/profile_icon.dart`
- `lib/widgets/rotating_widget.dart`

**Estimated: ~37 files, ~9500 lines** (config.dart 39KB, printer_utils.dart 47KB, main_drawer.dart 27KB, mix_action.dart 25KB are the largest)

## Layer 3 — Native Platform Code (medium-high priority)
Android Kotlin and iOS Swift native layers — widgets, activities, extensions. Critical for understanding platform-specific features.

### Android Native (Kotlin)
- `android/app/src/main/kotlin/com/remixlabs/runtime/MainActivity.kt`
- `android/app/src/main/kotlin/com/remixlabs/runtime/BaseWidget.kt`
- `android/app/src/main/kotlin/com/remixlabs/runtime/BaseWidgetConfigureActivity.kt`
- `android/app/src/main/kotlin/com/remixlabs/runtime/BaseJsInterface.kt`
- `android/app/src/main/kotlin/com/remixlabs/runtime/EmbeddedWebViewActivity.kt`
- `android/app/src/main/kotlin/com/remixlabs/runtime/ShareActivity.kt`
- `android/app/src/main/kotlin/com/remixlabs/runtime/Utils.kt`
- `android/app/src/main/kotlin/com/remixlabs/runtime/WidgetUtils.kt`
- `android/app/src/main/kotlin/com/remixlabs/runtime/SecurePrefrences.kt`
- `android/app/src/main/kotlin/com/remixlabs/runtime/ClickInfo.kt`
- `android/app/src/main/kotlin/com/remixlabs/runtime/data/WidgetConfig.kt`
- `android/app/src/main/kotlin/com/remixlabs/runtime/data/WidgetInfo.kt`
- `android/app/src/main/kotlin/com/remixlabs/runtime/data/WidgetRecord.kt`
- `android/app/src/main/kotlin/com/remixlabs/runtime/data/WidgetResponse.kt`
- `android/app/src/main/kotlin/com/remixlabs/runtime/data/WidgetSize.kt`
- `android/app/src/main/kotlin/com/remixlabs/runtime/data/Dim.kt`
- `android/app/src/main/AndroidManifest.xml`

### iOS Native (Swift) — Main App
- `ios/Runner/AppDelegate.swift`
- `ios/Runner/FLAppClipView.swift`
- `ios/Common.swift`
- `ios/CommonMixCore.swift`
- `ios/CommonWebkit.swift`
- `ios/Logger.swift`
- `ios/QuickLoopHelpers.swift`
- `ios/Runner/RmxNFCWriter.swift`
- `ios/String+extension.swift`
- `ios/Digest+extension.swift`
- `ios/Runner/Runner-Bridging-Header.h`

### iOS WidgetKit Extension
- `ios/RemixWidget/RemixWidget.swift`
- `ios/RemixWidget/WidgetAgent.swift`
- `ios/RemixWidget/WidgetAgentUtils.swift`
- `ios/RemixWidget/Dom.swift`
- `ios/RemixWidget/NodeViewSwift.swift`
- `ios/RemixWidget/Val.swift`
- `ios/RemixWidget/Patch.swift`
- `ios/RemixWidget/AppMeta.swift`
- `ios/RemixWidget/NetworkImage.swift`
- `ios/RemixWidget/LengthWithUnit.swift`
- `ios/RemixWidget/Utils.swift`
- `ios/RemixWidget/ViewModifiers.swift`
- `ios/RemixWidget/View+extension.swift`
- `ios/RemixWidget/WidgetConfiguration+extension.swift`
- `ios/RemixWidget/WidgetDebugger.swift`
- `ios/RemixWidget/Info.plist`
- `ios/RemixWidget/RemixWidgetExtension.entitlements`

### iOS AppClip Extension
- `ios/RemixAppClip/RemixAppClipApp.swift`
- `ios/RemixAppClip/AppClipView.swift`
- `ios/RemixAppClip/AppClipWebView.swift`
- `ios/RemixAppClip/EmbeddedWebView.swift`
- `ios/RemixAppClip/Info.plist`
- `ios/RemixAppClip/RemixAppClip.entitlements`

### iOS Other Extensions
- `ios/RemixShareExtension/ShareViewController.swift`
- `ios/RemixShareExtension/Info.plist`
- `ios/RemixQuickLook/PreviewViewController.swift`
- `ios/RemixQuickLook/Info.plist`
- `ios/RemixAgentIntentExtension/IntentHandler.swift`
- `ios/RemixAgentIntentExtension/RemixIntents.intentdefinition`
- `ios/RemixAgentIntentExtension/Info.plist`

**Estimated: ~58 files, ~10000 lines** (Dom.swift 36KB, AppDelegate.swift 24KB, AppClipWebView.swift 19KB, BaseWidget.kt 19KB, ViewModifiers.swift 17KB are the largest)

## Layer 4 — Build & Deploy Tooling (medium priority)
Fastlane configs, shell scripts, web assets, build-assets.

- `android/fastlane/Fastfile`
- `android/fastlane/Appfile`
- `ios/fastlane/Fastfile`
- `ios/fastlane/Appfile`
- `ios/fastlane/Matchfile`
- `bin/setup.sh`
- `bin/cp_apk.sh`
- `bin/cp_ipa.sh`
- `bin/cp_sms_apk.sh`
- `bin/upload_apk_to_gcloud.sh`
- `bin/upload_sms_apk_to_gcloud.sh`
- `bin/upload_web_assets.sh`
- `bin/upload_packaged_folder.sh`
- `bin/upload_rmx_fonts.sh`
- `bin/download_packaged_folder.sh`
- `bin/download_rmx_fonts_assets.sh`
- `bin/package_rmx_fonts.sh`
- `bin/sync_rmx_core_assets_from_builder.sh`
- `bin/create_version_folder.sh`
- `bin/generate_versions_json.sh`
- `bin/format_date.sh`
- `bin/check_elf_alignment.sh`
- `assets/web/js/app.js`
- `assets/web/js/common.js`
- `assets/web/css/init.css`
- `android/app/src/main/assets/widget.html`
- `android/app/src/main/assets/sharesheet.html`
- `ios/web/appclip.html`
- `ios/web/quicklook.html`
- `ios/web/sharesheet.html`

**Estimated: ~30 files, ~2000 lines**

## Layer 5 — send_sms Plugin & Tests (low priority, skip if budget tight)
Embedded Flutter plugin for SMS and test files.

- `send_sms/pubspec.yaml`
- `send_sms/lib/send_sms.dart`
- `send_sms/lib/send_sms_method_channel.dart`
- `send_sms/lib/send_sms_platform_interface.dart`
- `send_sms/android/src/main/kotlin/com/remixlabs/send_sms/SendSmsPlugin.kt`
- `send_sms/android/build.gradle`
- `send_sms/android/settings.gradle`
- `send_sms/android/src/main/AndroidManifest.xml`
- `send_sms/example/lib/main.dart`
- `test/amp_config_test.dart`
- `test/utils_test.dart`

**Estimated: ~11 files, ~800 lines**

## Excluded (do not read)
- `pubspec.lock`, `Gemfile.lock`, `ios/Podfile.lock` — lock files
- `android/key.properties`, `android/settings_aar.gradle` — signing/build variants
- `android/.editorconfig`, scattered `.gitignore` files (root, `android/`, `android/app/src/main/assets/`, `ios/`, `ios/web/`, `ios/RemixAppClip/`, `send_sms/`, `send_sms/example/`) — trivial config
- `android/app/google-services.json`, `ios/Runner/GoogleService-Info.plist`, `ios/RemixAppClip/GoogleService-Info.plist` — Firebase config
- `build-assets/` — binary/credential artifacts (`.mp`, `.p8`, `.json` key)
- `android/Ampmobile/build.gradle` — legacy AMP module
- `ios/Runner.xcodeproj/project.pbxproj` — auto-generated Xcode project (143 KB, too large)
- `ios/Runner.xcworkspace/` — workspace metadata
- `ios/Runner.xcodeproj/xcshareddata/xcschemes/` — Xcode schemes
- `ios/Flutter/` — Flutter Xcode configs (auto-generated)
- `ios/Frameworks/` — empty placeholder (`.git_keep` only)
- `ios/PrivacyInfo.xcprivacy` — Apple privacy manifest
- `ios/Remix - Smartphone Superpower.storekit` — StoreKit config
- `ios/Assets.xcassets/` — all icon image assets (PNG files + Contents.json)
- `ios/Runner/Assets.xcassets` — 18-byte placeholder
- `ios/Runner/Base.lproj/LaunchScreen.storyboard`, `ios/Runner/Base.lproj/Main.storyboard` — storyboards
- `assets/icon/`, `assets/images/`, `assets/widgets/` — image assets
- `android/app/src/main/res/drawable*`, `android/app/src/main/res/mipmap*` — Android image resources
- `android/app/src/main/res/color/`, `android/app/src/main/res/values*` — Android XML resources
- `android/app/src/main/res/layout/` — Android layout XML (read only if investigating widget UI)
- `android/app/src/main/res/xml/` — Android XML config (widget metadata)
- `android/app/src/debug/`, `android/app/src/profile/` — variant manifests
- `android/app/src/main/java/` — empty Java placeholder (`.gitkeep` only)
- `ios/RemixWidget/RemixWidgetExtension-Bridging-Header.h` — bridging header
- `ios/RemixAppClip/RemixAppClip-Bridging-Header.h`, `ios/RemixAppClip/AppClipDebug.xcconfig`, `ios/RemixAppClip/AppClipRelease.xcconfig` — Xcode config
- `ios/RemixShareExtension/Base.lproj/`, `ios/RemixQuickLook/Base.lproj/` — storyboards
- `ios/RemixShareExtension/RemixShareExtension-Bridging-Header.h`, `ios/RemixShareExtension/RemixShareExtension.entitlements`
- `ios/RemixQuickLook/RemixQuickLookExtension-Bridging-Header.h`, `ios/RemixQuickLook/RemixQuickLook.entitlements`
- `ios/RemixAgentIntentExtension/RemixAgentIntentExtension-Bridging-Header.h`, `ios/RemixAgentIntentExtension/RemixAgentIntentExtension.entitlements`
- `send_sms/example/android/` — example app boilerplate (includes res/, gradle, manifests)
- `send_sms/example/README.md`, `send_sms/CHANGELOG.md`, `send_sms/LICENSE`, `send_sms/README.md` — docs
- `send_sms/analysis_options.yaml`, `send_sms/example/analysis_options.yaml` — lint config
- `send_sms/pubspec.lock`, `send_sms/.metadata`, `send_sms/.gitignore`, `send_sms/example/.gitignore`
- `android/fastlane/README.md`, `ios/fastlane/README.md`, `ios/Assets.xcassets/LaunchImage.imageset/README.md` — Fastlane/asset READMEs

## Reading Budget Estimate
| Layer | Files | Est. Lines | Est. KB |
|-------|-------|------------|---------|
| L1    | ~21   | ~2000      | ~70     |
| L2    | ~37   | ~9500      | ~319    |
| L3    | ~58   | ~10000     | ~337    |
| L4    | ~30   | ~2000      | ~60     |
| L5    | ~11   | ~800       | ~25     |
| **Total** | **~157** | **~24300** | **~811** |

**Recommended approach:** L1 + L2 full (~58 files, ~11500 lines). L2 has several very large files (config.dart 40KB, printer_utils.dart 47KB, main_drawer.dart 27KB, mix_action.dart 26KB, store_mgr.dart 22KB) that dominate the budget. L3 selective: iOS WidgetKit core (Dom.swift 37KB, RemixWidget.swift 16KB, ViewModifiers.swift 17KB) + Android BaseWidget.kt 19KB + AppDelegate.swift 24KB — ~25 files, ~6000 lines. L4 skim Fastlane only. Budget ~85 files / ~18000 lines.
