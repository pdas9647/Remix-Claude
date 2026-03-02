# #desktop Slack Channel — Remix Labs

**Coverage:** Dec 20, 2025 – Feb 28, 2026
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

**[Feb 12 + Feb 18]** Set `home` field in workspace sync group record. Same pattern as mobile. Desktop home button navigates to custom home app. No user-space tooling yet — must manually edit data tile + invoke agent.

**Auto-sync:** Workspace sync group refreshes on every restart (not just first install).

**[Feb 18, 53 replies]** Fresh install issue: app meta not written correctly; `_rmx_tailwind` miscategorized. Fix: mix-rs/pull/991. Desktop home ≠ workspace home — need `navigate to my home` action.

---

## Key Bugs & Fixes

**Paste large module → 32KB limit [Jan 27]:** DELETE of ~2000 IDs hits Rocket 32KB form limit + O(n²) batch delete. Fixes: mix-rs/pull/980 (raise to 3.5MB), mix-rs/pull/983 (10min → 3s).

**Auth "illegal redirect" [Jan 22]:** `_rmx_auth` used deprecated `env.backendBaseURL` (empty in desktop). Fix: use `env.baseURL` — turntable/pull/11668. **Rule:** always use `env.baseURL`.

**"Open Link" broken [Feb 5]:** `window.open` doesn't work in Tauri. Fix: mix-rs/pull/977 — opens in default browser. remix.app incompatible with Tauri native webview.

**Timezone wrong [Feb 5-9]:** Runtime had invalid `America/San Francisco`. Fix: mix-rs/pull/979, desktop v9640. Service agents still ignore timezone: mix-rs/issues/987.

**CORS in preview [Dec 29]:** Preview VMs lack mixcore FFI → fall back to HTTP → CORS. Fix: turntable/pull/11620 (route through FFI). Basic auth unsupported in mixcore: mix-rs/issues/926.

**Autocorrect corrupting JSON [Dec 29]:** macOS smart quotes in inputs. Fix: `spellcheck="false"` + `autocorrect="off"` — turntable/pull/11626, turntable/pull/11666.

**Arrow key tofu chars [Jan 31]:** Inserts garbage on first launch. Workaround: open DevTools. Tauri/webview bug, needs upstream repro. Still unfixed in prod as of Feb 28 (absent in dev builds — different Tauri/webview?).

**Syncing preferences error [Feb 2]:** Workspace shows "legacy". Fix: mix-rs/pull/965. Windows auto-update broken — Fred manually distributes.

**`.remix` always opens `home` [Feb 5]:** remix.app hardcoded to `home` screen. Workaround: Navigate tile on `home`'s onLoad.

**Downloads broken [Jan 5-8]:** Download webcomp not wired to Tauri. Partial fix: mix-rs/pull/954. Blob FFIs: groovebox/pull/457.

---

## Architecture & Infrastructure

**Claude Desktop / MCP auto-launch [Feb 11]:** Remix Desktop auto-launched via MCP config. Decision: remove auto-start; user manually starts or setup flow writes config deliberately.

**`_rmx_tailwind` invisible after D&D install [Feb 10]:** D&D install doesn't write `_rmx_admin` db entry. Regular installs work.

**`_rmx_admin` db [Feb 14]:** Hidden, implicitly created by mixer. Stores app meta for L0. Access: `remix://localhost/a/v1/ws/local/appmeta/`. D&D install bug: no meta written here.

**Release channel switcher [Feb 6]:** mix-rs/pull/976 — switch `dev`/`beta`/`release` in-app (no reinstall). Channels: `main`→`dev`; `beta`/`release` need branch protection + promotion workflow.

**Binaries on filesystem [Jan 14]:** turntable/pull/11636 — build binaries moved from DB to `/code` filesystem.

**Window title sync [Jan 12]:** `document.title` → Tauri window title via `on_document_title_changed` hook — mix-rs/pull/943.

**config.toml open questions [Feb 22]:** `apps_url` possibly outdated; `app_versions` accumulates old entries.

**Reload Studio window action needed [Feb 18]:** No mechanism to programmatically reload a window after installing .remix.

---

## Feb 22–28: Desktop Launch Push

**.remix v2 exec-only install broken [Feb 24]:** Packaged exec-only .remix installs as editable project with no assets. Auto-sync from `_rmx_sync` works; userspace install does not. Must converge both flows.

**No dual-workspace DX [Feb 24]:** Can't run two Desktop instances for different workspaces. Workarounds: two laptops, two OS users, macOS fast user switching (once HTTP port configurable). Configurable port: mix-rs/pull/948. Tracked: mix-rs/issues/1014.

**Window menu disambiguation [Feb 24]:** Can't distinguish Studio vs runtime windows. Tyler's rules: builder links → desktop builder window; runtime links → runtime window; external → browser. Window title format `dbName / screen`: mix-rs/pull/1023. Client actions disabled in builder context (too many side effects).

**Release serving [Feb 24]:** Mac via Tauri updater (binaries in R2/GCS). Windows: Fred manually uploads `.exe`. Long-term: automate Windows into same agent flow.

**Deeplink param fix [Feb 24]:** `remix_file_url` → `url` for consistency with Flutter/docs. Fix: mix-rs/pull/1013.

**Apple Silicon only [Feb 25]:** Intel Macs unsupported; macOS 26 (Tahoe) drops most Intel anyway. Safe to de-prioritize.

**"Same Window" for external links [Feb 25]:** Ignored — browser opens, desktop window stays. mix-rs/issues/1018.

**Windows bugs: `msg_perform_client_action` [Feb 26-27]:** Codegen change in `msg_open_link` requires rebuild+republish of all affected apps (including `auth.remix`). Fix: Didier republished; Fred posted v0.0.16–v0.0.18. Wipe path: `~/AppData/Local/com.remixlabs.desktop/workspaces/local`.

**Upgrade fails: `relative URL without a base` [Feb 27]:** Platform 1.46 bump broke `desktop_home_remix` (old GCS endpoint). Fix: Didier migrated version files. Home app is NOT built-in — comes from workspace sync group. Override: `REMIX_DESKTOP_WORKSPACE_APPS_PATH`.

**Pre-prod checklist [Feb 27] (Arvind):** (1) Arrow key tofu chars, (2) autocorrect/curly quotes, (3) MCP config cleanup, (4) solid sync, (5) default home.

**Sync model proposal (Didier):** Move all `.remix` apps to synced (mobile-like model); only `_rmx_desktop` stays in binary. Others (`_rmx_artifact`, `_rmx_search`, `_rmx_tailwind`, `_rmx_extension`) all sync.

**Auto-rebuild replaces "Constants not found" modal [Feb 27]:** Simon's silent auto-rebuild. Regression: `MODULE NOT FOUND` errors on library-synced screens — workaround: save+close+reopen. Direction: block UI + spinner during full rebuild (can take 45s).

---

## Channel Patterns

- Primary: Benedikt (desktop backend), Tyler (Tauri/webcomps), Didier (sync/runtime), Arvind (product), Gerd (platform), Simon (builder)
- Very technical; PRs linked inline, bugs cross-posted to #bugbash
- Browser-vs-native debate resurfaces frequently; Tauri limitations recurring friction
- Windows support manual/secondary (no auto-update)