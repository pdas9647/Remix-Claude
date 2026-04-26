# #desktop Slack Channel — Remix Labs

**Coverage:** Dec 20, 2025 – Apr 25, 2026
**Channel ID:** C04T4HW06CC
**Bug tracker:** https://github.com/remixlabs/mix-rs/issues/937

---

## Architecture Debate: Native Tauri vs Browser Builder

**[Jan 30, 111 replies]** Option A: Tauri menubar + browser-hosted Studio. Option B: full native long-term.

- **Browser downsides (Tyler):** must re-implement tabs, bookmarks, `beforeunload`, zoom, clipboard, window title
- **Native advantages (Benedikt):** mixcore/FFIs, local compiler, file access, window mgmt, `_rmx_extension.core.runFlow`
- **Gerd:** most native advantages can be proxied via desktop backend even in browser mode
- **Resolution:** Ship Option A now; revisit after real external usage. Long-term, native is correct direction.

---

## Custom Home App & App Sync

**[Feb 12 + Feb 18]** Set `home` field in workspace sync group record. Same pattern as mobile. Desktop home button navigates to custom home app. No user-space tooling yet — must manually edit data
tile + invoke agent.

**Auto-sync:** Workspace sync group refreshes on every restart (not just first install).

**[Feb 18, 53 replies]** Fresh install issue: app meta not written correctly; `_rmx_tailwind` miscategorized. Fix: mix-rs/pull/991. Desktop home ≠ workspace home — need `navigate to my home` action.

---

## Key Bugs & Fixes

**Paste large module → 32KB limit [Jan 27]:** DELETE of ~2000 IDs hits Rocket 32KB form limit + O(n²) batch delete. Fixes: mix-rs/pull/980 (raise to 3.5MB), mix-rs/pull/983 (10min → 3s).

**Auth "illegal redirect" [Jan 22]:** `_rmx_auth` used deprecated `env.backendBaseURL` (empty in desktop). Fix: use `env.baseURL` — turntable/pull/11668. **Rule:** always use `env.baseURL`.

**"Open Link" broken [Feb 5]:** `window.open` doesn't work in Tauri. Fix: mix-rs/pull/977 — opens in default browser. remix.app incompatible with Tauri native webview.

**Timezone wrong [Feb 5-9]:** Runtime had invalid `America/San Francisco`. Fix: mix-rs/pull/979, desktop v9640. Service agents still ignore timezone: mix-rs/issues/987.

**CORS in preview [Dec 29]:** Preview VMs lack mixcore FFI → fall back to HTTP → CORS. Fix: turntable/pull/11620 (route through FFI). Basic auth unsupported in mixcore: mix-rs/issues/926.

---

## Architecture & Infrastructure

**Claude Desktop / MCP auto-launch [Feb 11]:** Remix Desktop auto-launched via MCP config. Decision: remove auto-start; user manually starts or setup flow writes config deliberately.

**`_rmx_admin` db [Feb 14]:** Hidden, implicitly created by mixer. Stores app meta for L0. Access: `remix://localhost/a/v1/ws/local/appmeta/`. D&D install bug: no meta written here.

**Release channel switcher [Feb 6]:** mix-rs/pull/976 — switch `dev`/`beta`/`release` in-app (no reinstall). Channels: `main`→`dev`; `beta`/`release` need branch protection + promotion workflow.

**Binaries on filesystem [Jan 14]:** turntable/pull/11636 — build binaries moved from DB to `/code` filesystem.

**Window title sync [Jan 12]:** `document.title` → Tauri window title via `on_document_title_changed` hook — mix-rs/pull/943.

**config.toml open questions [Feb 22]:** `apps_url` possibly outdated; `app_versions` accumulates old entries.

**Reload Studio window action needed [Feb 18]:** No mechanism to programmatically reload a window after installing .remix.

---

## Feb 22–28: Desktop Launch Push

**.remix v2 exec-only install broken [Feb 24]:** Packaged exec-only .remix installs as editable project with no assets. Auto-sync from `_rmx_sync` works; userspace install does not. Must converge both
flows.

**No dual-workspace DX [Feb 24]:** Can't run two Desktop instances for different workspaces. Workarounds: two laptops, two OS users, macOS fast user switching (once HTTP port configurable).
Configurable port: mix-rs/pull/948. Tracked: mix-rs/issues/1014.

**Window menu disambiguation [Feb 24]:** Can't distinguish Studio vs runtime windows. Tyler's rules: builder links → desktop builder window; runtime links → runtime window; external → browser. Window
title format `dbName / screen`: mix-rs/pull/1023. Client actions disabled in builder context (too many side effects).

**Release serving [Feb 24]:** Mac via Tauri updater (binaries in R2/GCS). Windows: Fred manually uploads `.exe`. Long-term: automate Windows into same agent flow.

**Deeplink param fix [Feb 24]:** `remix_file_url` → `url` for consistency with Flutter/docs. Fix: mix-rs/pull/1013.

**Apple Silicon only [Feb 25]:** Intel Macs unsupported; macOS 26 (Tahoe) drops most Intel anyway. Safe to de-prioritize.

**"Same Window" for external links [Feb 25]:** Ignored — browser opens, desktop window stays. mix-rs/issues/1018.

**Windows bugs: `msg_perform_client_action` [Feb 26-27]:** Codegen change in `msg_open_link` requires rebuild+republish of all affected apps (including `auth.remix`). Fix: Didier republished; Fred
posted v0.0.16–v0.0.18. Wipe path: `~/AppData/Local/com.remixlabs.desktop/workspaces/local`.

**Upgrade fails: `relative URL without a base` [Feb 27]:** Platform 1.46 bump broke `desktop_home_remix` (old GCS endpoint). Fix: Didier migrated version files. Home app is NOT built-in — comes from
workspace sync group. Override: `REMIX_DESKTOP_WORKSPACE_APPS_PATH`.

**Sync model proposal (Didier):** Move all `.remix` apps to synced (mobile-like model); only `_rmx_desktop` stays in binary. Others (`_rmx_artifact`, `_rmx_search`, `_rmx_tailwind`, `_rmx_extension`)
all sync.

**Auto-rebuild replaces "Constants not found" modal [Feb 27]:** Simon's silent auto-rebuild. Regression: `MODULE NOT FOUND` errors on library-synced screens — workaround: save+close+reopen. Direction:
block UI + spinner during full rebuild (can take 45s).

## Mar 2–7: Lumber RC + Windows Stability

**Mac beta 0.10180.0 — Lumber release candidate [Mar 4]:** Chris cut Lumber RC build. Build versioning: dev/beta/release are distinct version streams (not simple promotions of same artifact).
Benedikt: `desktop-release` CLI command exists for promoting builds; hot-fix branches remain an advantage of separate builds. Future: promote existing artifacts instead of rebuilding.

**Windows v0.0.19 + v0.0.20 [Mar 4]:** Fred cut v0.0.19. Platform version not displaying. Frequent crashes traced to `muda-0.17.1` (Tauri menu crate): `RefCell already mutably borrowed` +
`STATUS_STACK_BUFFER_OVERRUN` on Windows 11. Already on latest muda — upstream bug. Fred built v0.0.20 with updated dependencies, tentatively fixed. **Desktop needs a panic hook** (Benedikt). Chris:
motivation to get Windows release into CI.

**MCP bridge revamp PR [Mar 4]:** mix-rs/pull/1039. Available on release channel `bb-revamp-mcp`. Claude must still be started after Remix Desktop (confirmed by Vijay).

**Tofu char fix [Mar 5]:** mix-rs/pull/1047 (Benedikt). Addresses font rendering issue with missing glyph squares.

**App install improvement [Mar 2]:** mix-rs/pull/1038 (Benedikt). Avoids re-install for any URL type; adds "origin" to deployments.json. Different install procedures store own origin metadata.

**Mixcore FFI empty in browser-accessed mixer [Mar 2-3, 37 replies]:** Accessing desktop mixer via browser URL (`_rmx_host=http://localhost:2025`) doesn't load mixcore FFIs — `config.kind` undefined.
Root cause: front end only sets mixcore for Desktop and .remix contexts. Fix: mix-rs/pull/1041 (Benedikt), configures compiler WebSocket connection. Merged; unblocked Gerd.

**White screen of death [Mar 3]:** Gerd hit rare WSOD — desktop completely non-reactive, window unclosable.

**Windows buttons grayed out [Mar 2]:** Sirshendu: all buttons grayed, losing unsaved progress. Simon: happens when frontend loses backend heartbeat — should show "unfreeze" button after 1 min, but
didn't. Workaround: hidden unfreeze button in edit menu debug panel (Arvind).

**Make updates project timestamp [Mar 3]:** Arvind: simple make + exit updates last-modified timestamp for whole project. Shouldn't happen.

---

## Channel Patterns

- Primary: Benedikt (desktop backend), Tyler (Tauri/webcomps), Didier (sync/runtime), Arvind (product), Gerd (platform), Simon (builder)
- Very technical; PRs linked inline, bugs cross-posted to #bugbash
- Browser-vs-native debate resurfaces frequently; Tauri limitations recurring friction
- Windows support manual/secondary (no auto-update)

## Mar 14–21: OPFS Regressions (resolved Mar 23)

**OPFS regressions [Mar 18–21, Benedikt]:** Two bugs: (1) sign-in logo missing in Tauri — embedded blob images broken (fix: mix-rs#1065); (2) system plugins crash “Name is invalid” — .remix URL
slashes not URL-encoded in OPFS path (fix: turntable#11824). Hold issued Mar 19 (build 10464 safe); lifted Mar 23 with v0.10644.

**Feature request: “disable a release” in release agent [Mar 19, Arvind → Benedikt]:** Mark a release disabled to block unsafe updates. Could add to `desktop-releases` app (release JSON served from
fixed URL per channel, created by `new_release` agent in CI). Open.

## Mar 23–Apr 3: OPFS Recovery + Export FFI

**OPFS restored [Mar 23, v0.10644]:** Plugins + workspace form fixed; update hold lifted.
**Plugin cache [Mar 23, Gerd]:** `Build_Tools` no Cache-Control (same root cause as .remix files).
**Windows v0.0.23 [Mar 24, Fred]:** Turntable updates.
**rmx_artifact v1.46 [Mar 27]:** mix-rs/pull/1080 approved.
**`make setup dev` hangs [Mar 28, Didier]:** Post-OAuth "waiting for auth"; unresolved.
**export_package FFI [Mar 31, Simon]:** mix-rs/pull/1084 (bypasses tauri-plugin-http); CORS concern pending.
**Restore windows on update [Apr 1, Arvind]:** Feature request; open.

## Apr 11–18 Additions

**Windows v0.0.24 + v0.0.25 [Apr 14–16, Fred]:** Fred cut v0.0.24 (stability fixes) and v0.0.25 (turntable updates).

**Windows crashes at L2 — unresolved [Apr 14–15, Sirshendu → team]:** Desktop on Windows crashes when navigating to L2. Sirshendu reproduced consistently; Simon couldn’t on Mac. No fix yet.

**Wasm tiles confirmed [Apr 15, Simon → Sirshendu]:** Wasm function tiles work correctly in Desktop/mixer context.

**Cmd+A / Cmd+Z broken in text inputs [Apr 16, Arvind → Benedikt]:** Select-all and undo non-functional in Desktop text inputs. Root cause: Tauri/WKWebView does not dispatch Cmd+A or Cmd+Z to DOM.
Known upstream Tauri limitation. Unresolved.

**Function tile library tab empty [Apr 16, Sirshendu → Simon]:** Library tab in function tile editor shows no items. Repro confirmed. Open.

**Workspace tools: arbitrary backend servers [Apr 17, Arvind]:** remix-issues#166 — workspace tools should support arbitrary backend servers, not just localhost. Needed for self-hosted deployments.

## Apr 18–25 Additions

**Login page scrollable / Lumber build channel [Apr 20, Padmanabha/Arvind/Chris]:** v0.10557.0 login page vertically scrollable; workspace field off-screen on mobile (SE2). Didier: debug experiment,
fixed. Lumber Desktop build: **lumber-qa** channel. Padmanabha couldn’t sign in to lumber-qa — Arvind: try **beta channel** as working deliverable.

**New project creation error [Apr 20, Padmanabha]:** Error immediately after creating new project. Open.

**Release channel install error [Apr 20, Arka]:** Error after clean uninstall + reinstall of release channel. Open.

**Desktop can’t switch channel [Apr 21, Padmanabha]:** No channel-switch UI in Desktop. Open.

**Windows build for lumber-qa [Apr 22, Sirshendu]:** Request for Windows release on lumber-qa channel. Open.

**Sync overwrite corrupts binding [Apr 22, Padmanabha]:** “Sync nodes overwriting local changes” makes card out-binding invalid (lumber-qa, Generate CSV). Inconsistent repro. Open.

**Navbar: wrong open window count [Apr 23, Simon]:** Desktop navbar shows fewer windows than actually open. Open.

**`_rmx_tailwind` styles missing in Desktop [Apr 23, Padmanabha]:** Theme not rendering in remix-desktop. Open.

**Desktop fails to start offline [Apr 25, Simon]:** Offline — DB rejects all requests during app sync, blocking Desktop and dev-environment builder. Open.
