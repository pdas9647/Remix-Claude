# #desktop Slack Channel — Remix Labs

**Coverage:** Dec 20, 2025 – Feb 28, 2026
**Channel ID:** C04T4HW06CC
**Bug tracker:** https://github.com/remixlabs/mix-rs/issues/937

---

## Architecture Debate: Native Tauri vs Browser Builder

### [Jan 30, 111 replies] Tauri native window vs. browser-hosted Studio (Arvind)

Major ongoing discussion. Two framings (Arvind):

- **Option A:** Tauri menubar + native widgets + **browser-hosted** Studio & runtimes
- **Option B:** Tauri menubar + native widgets + **full native** long-term investment

**Tyler's list of things Tauri has to re-implement that browsers give for free:**
tabs, bookmarks, `beforeunload`, zoom, clipboard issues, window title sync (fixed)

**Didier's whack-a-mole list already done:** copy/paste, capitalization, unsaved work detection, `window.open`, window title updates

**Benedikt for native:** native mixcore/FFIs, native compiler, local file access, window management (windows vs tabs, context menus, global shortcuts). Also: `_rmx_extension.core.runFlow` FFI only
possible natively.

**Gerd:** most native advantages can be proxied via the desktop backend even in browser mode.

**Vijay:** Use what we have now (Option A ~ready), revisit after real usage. Long-term, native is correct. Git for libraries is the right desktop model.

**Arvind conclusion:** For heavy professional daily-use tools, native is the right direction — psychological benefit + widgets + global shortcuts + offline-first + background operation. Option A is
what we ship now.

**Resolution:** Ship Option A (Tauri wrapping browser Studio); revisit properly after real external usage.

---

## Custom Home App & App Sync

### [Feb 12 + Feb 18] Custom workspace home app (Arvind + Didier)

**Mechanism:** Set a `home` field in the workspace sync group record (Org/System Setup notion doc). Same pattern as mobile. Desktop home button confirmed to navigate to the custom home app.

**Current pain:** No user-space tooling — must manually edit a data tile + invoke an agent. Need proper tooling integrated with the new publish/v2 .remix workflow.

**Auto-sync:** Workspace sync group refreshes on every desktop restart (not just first install).

### [Feb 18, 53 replies] Auto-install & sync of custom apps per workspace (Arvind + Didier)

**Mechanism confirmed:** Workspace-level sync group sets apps for all users in a workspace. Deeplinks work for opening .remix files (same as mobile).

**Fresh install issue:** No themes or projects appearing — app meta not being written correctly. `_rmx_tailwind` shows under "Packaged apps" instead of "Themes".

**Fix:** mix-rs/pull/991 (Arvind) — corrects tailwind category in builtin apps JSON.

**Desktop home ≠ workspace home:** The default `desktop_home_remix` is the system app (auto-opens on launch); workspace custom home must also be set and redirected to via the home button. Need a
`navigate to my home` action.

---

## Key Bugs & Fixes

### [Jan 27, 53 replies] Paste large module → 32KB Rocket limit (Arvind)

**Error:** `[AddNxMod] failed — size must not exceed 32KiB` when pasting full Styles module.

**Root cause:** DELETE of ~2000 IDs hits Rocket's 32KB form limit. Separately, batch delete was O(n²) — 10+ minute deletes due to no index in brute::Simple.

**Fixes (both merged):**

- mix-rs/pull/980: raise limit to ~3.5MB
- mix-rs/pull/983 (Fred): batch delete optimization — 10min → 3s

### [Jan 22, 53 replies] Auth "illegal redirect" error (Arvind + Benedikt)

**Error:** `illegal redirect: forbidden redirect scheme "" (only http(s) is allowed)` when signing in.

**Root cause:** `_rmx_auth` used `env.backendBaseURL` for redirect, which is `""` in appclip/desktop environments. Workaround: manually set token in `config.toml` (
`~/Library/Application Support/com.remixlabs.desktop/config.toml`).

**Fix:** Replace `backendBaseURL` with `env.baseURL` (always set). turntable/pull/11668 (Simon). Issue turntable/issues/11663.

**Note:** `env.backendBaseURL` is deprecated; use `env.baseURL` in all new screens.

### [Feb 5, 19 replies] "Open Link" action broken in desktop runtime (Arvind + Tyler)

`window.open` doesn't work in Tauri native window. Fix: mix-rs/pull/977 (Tyler, merged) — "Open Link" now opens in default browser. Future: new client action to run a `.remix` file natively within
desktop (similar to `_rmx_extension.core.runFlow`).

**Note:** remix.app is not compatible with Tauri native webview — it's built for the browser.

### [Feb 5-9] Client timezone wrong in desktop runtime (John)

Builder had `America/Los_Angeles`, runtime had hardcoded `America/San Francisco` (invalid). Fix: mix-rs/pull/979 (Benedikt + Simon), desktop v9640. Remaining: service agents ignore timezone in
requests — mix-rs/issues/987.

### [Dec 29] CORS errors in desktop preview (Gerd)

Preview VMs (`client_builder-preview`, `embedded.XYZ`) don't have mixcore FFI — fall back to groovebox HTTP FFI, subject to browser CORS. Builder VM has mixcore (no CORS). Fix: turntable/pull/11620 —
route preview HTTP through FFI. Basic auth still unsupported in mixcore: mix-rs/issues/926.

### [Dec 29] Autocorrect / smart quotes corrupting JSON input (Gerd)

macOS autocorrects quotation marks in JSON input fields. Fix: `spellcheck="false"` + `autocorrect="off"` attributes. turntable/pull/11626 and turntable/pull/11666 (textarea DRY fix, Simon).

### [Jan 31] Right arrow key inserts tofu character in form fields (Arvind)

Happens on first launch of desktop. Workaround: open DevTools inspector (fixes it immediately). Tauri/webview bug — needs minimal repro to report upstream.

### [Feb 2] "Error loading syncing preferences" (Padmanabha)

Appears when workspace still shows as "legacy" in avatar menu. Fixed in mix-rs/pull/965. **Windows auto-update is not working** — Fred manually distributes installer.

### [Feb 5] `.remix` always opens `home` screen

remix.app only opens a screen named `home`. Workaround: add a Navigate tile on `home`'s onLoad to redirect to the desired landing screen.

### [Jan 5-8] Downloads broken in desktop (Gerd)

Download icon in `workspace_tools` cloud files does nothing. Root cause: download webcomp not wired to Tauri. Plan: `Content-Disposition: attachment` header from mixer's file-manager, or blob download
approach (Gerd: groovebox/pull/457 adds blob FFIs). mix-rs/pull/954 added partial download support.

---

## Architecture & Infrastructure

### [Feb 11, 17 replies] Claude Desktop dependency / MCP auto-launch (Padmanabha)

Remix Desktop auto-launches when Claude Desktop starts (MCP config written by desktop on startup). Confirmed: Claude Desktop not required to run Remix Desktop (config.toml permission error was the
real issue).

**Decision:** Remove auto-start. Options: (1) user manually starts Remix Desktop before using MCP tools, (2) setup flow in home app to write MCP config deliberately (Arvind + Benedikt agreed).

### [Feb 10, 15 replies] `_rmx_tailwind` invisible after D&D install (Gerd)

Installer logs success but app doesn't appear in L0. Root cause: D&D install doesn't write entry to `_rmx_admin` db — missing app meta. Regular apps install correctly. Logged in thread #1770744642.

### [Feb 14, 25 replies] `_rmx_admin` db on desktop (Arvind)

Hidden but exists (implicitly created by mixer). Stores all app meta for L0 display. Not surfaced in Studio UI. Access: `remix://localhost/a/v1/ws/local/appmeta/`. Wilber uses it for tooling → plan to
finish migration to workspace storage. D&D install bug: doesn't write app meta here.

### [Feb 6] Release channel switcher in userland (Benedikt)

mix-rs/pull/976 — users can now switch between `dev`/`beta`/`release` channels from within the app (no reinstall needed). Channels: `main`→`dev`; `beta` and `release` need branch protection rules +
formal promotion workflow.

### [Jan 14] Binaries now stored on filesystem (Gerd)

turntable/pull/11636 merged — build binaries moved from DB to `/code` filesystem. Speeds up compilation, reduces DB complexity. Prepares for screen relinking (follow-up PR planned).

### [Jan 12] Window title sync fixed (Benedikt)

`document.title` wasn't updating the native Tauri window title. Fix: `on_document_title_changed` Tauri hook → mix-rs/pull/943.

### [Feb 22] config.toml open questions (Arvind → Didier/Benedikt)

`apps_url` field appears outdated. `app_versions` list is an accumulation of old + current entries, not equivalent to `remix.remixlabs.com/_rmx_desktop/builtin_apps`. Under investigation.

### [Feb 18] Desktop ext action needed: reload Studio window (Arvind)

No current mechanism to programmatically target a window and reload its view. Needed for: home app installs a `.remix` → Studio window needs to refresh to see it.

---

## Feb 22-28, 2026 — Desktop Launch Push

### [Feb 24] .remix v2 (exec-only) install broken on desktop (Arvind/Wilber)

Installing a packaged exec-only v2 .remix file on desktop doesn't work — it installs as an editable project with no assets. Only installs with builder assets work today.

- Auto-sync from `_rmx_sync` at startup works; userspace install action does not
- Wilber filed bugbash thread with repro
- Arvind: these two flows must converge on the same infra/API

### [Feb 24] No way to simultaneously build-as-Remix and test-as-customer (Arvind)

**Blocker DX issue:** No way to run two Desktop instances for different workspaces on the same machine.

- Options: two laptops, or juggling two app directories on disk (Benedikt)
- macOS fast user switching is a workaround once HTTP port is configurable (Chris)
- DB/config directory is per OS user — two OS users on same Mac would work
- PR for configurable mixer port: mix-rs/pull/948
- Tracked: mix-rs/issues/1014

### [Feb 24] Window menu: Studio vs runtime window disambiguation (Didier/Simon/Benedikt/Arvind)

Window list menu can't distinguish Studio from runtime windows — both show raw document titles.

**Tyler's definitive behavior for links opened in desktop:**
- Builder links → open desktop builder window (same-window = navigate within)
- Runtime links → open desktop runtime window (same-window = navigate within)
- External links → always open in browser; original desktop window stays unchanged

**Window title work (mix-rs/pull/1023, Simon):** switched to reading browser `title` field; new format = `dbName / screen`. Arvind wants appMeta display name instead.

**Client actions in builder:** Currently ALL desktop client actions are disabled in builder context (too many side effects like logout, channel switch). Tracked to allow safe subset.

### [Feb 24] Desktop release serving + Windows builds (Fred/Benedikt)

- Mac desktop app served via Tauri updater; binaries stored in R2/GCS via an agent
- Release process: mix-rs/desktop/README.md#release-process
- Windows builds: Fred manually creates and uploads `.exe` installers (v0.0.15–v0.0.18 during this week)
- Long-term: automate Windows into the same agent flow; code for other platforms exists but untested

### [Feb 24] Deeplink param fixed: `remix_file_url` → `url` (Wilber/Benedikt)

Desktop deep link used `?remix_file_url=` while Flutter and Notion docs specify `?url=`. Historic discrepancy, not intentional. Fix: mix-rs/pull/1013.

### [Feb 25] Mac desktop: Apple Silicon only (Arvind/Didier/Chris)

**Confirmed:** Mac desktop currently only works on Apple Silicon (M1/M2+). Intel i7 Macs (2019 and older) do not work.

- macOS 26 (Tahoe) drops most Intel Mac support anyway
- Chris/Gerd: safe to de-prioritize Intel support (companies renew hardware every 3-5 years)

### [Feb 25] "Same Window" link behavior clarified for desktop (Benedikt/Tyler/Arvind)

See Tyler's rules above (Feb 24 window section). For external links: "Same Window" setting is ignored — browser tab opens and original desktop window remains intact. mix-rs/issues/1018 filed.

### [Feb 25] Plugin UI changes landed on desktop (Didier)

Small UI changes around plugins merged and deployed to desktop. (Elm CI flakiness required a retry.)

### [Feb 26–27] Windows desktop bugs: `Unrecognised action: msg_perform_client_action` (Sirshendu/Padmanabha/Didier)

After updating to v0.0.15+, Windows users (and Lumber workspace on Mac) saw:
- `Unrecognised action: msg_perform_client_action` on home screen load
- Auth deep link error: `Could not dispatch deep link: waiting for auth` when switching workspaces

**Root cause:** `msg_open_link` codegen changed ~1 month prior. Any app using it must be rebuilt and republished. Each workspace hosts its own `auth.remix` which also needs republishing when codegen changes.

**Fix:** Didier republished affected `.remix` files. Fred posted Windows v0.0.16/v0.0.17/v0.0.18. Workspace data dir to wipe if stuck: `~/AppData/Local/com.remixlabs.desktop/workspaces/local`.

### [Feb 27] Desktop upgrade fails: `relative URL without a base` (Gerd/Benedikt/Didier)

Platform bumped to 1.46 but `desktop_home_remix` (workspace sync group home app) still referenced `gcs/versioned` endpoint — unavailable at 1.46. Desktop exits on startup.

- Fix: Didier migrated all version files to 1.46
- **Architecture note:** Home app is NOT built-in — it comes from the workspace sync group. Override with `REMIX_DESKTOP_WORKSPACE_APPS_PATH` for local dev.
- Didier: wants to automate this so version bumps don't require manual migration steps

### [Feb 27] Arvind's pre-prod-build checklist

Before cutting a prod build (Arvind's list):
1. Arrow keys making buggy characters in text inputs
2. Auto-correct / auto-capitalization / curly quotes (Safari; `spellcheck="false"` should help — Benedikt)
3. Clean up MCP config — causes Claude errors for users with Claude Desktop installed
4. Solid sync experience — unify approach; consider mobile-like sync model (no apps embedded in DMG)
5. Default home page (non-customer)

**Sync model proposal (Didier):** Move to mobile-like model — only update desktop binary for real desktop changes; move all `.remix` apps to synced (not embedded); provide manual sync trigger. Only `_rmx_desktop` needs to stay in codebase (has core service agents). Others (`_rmx_artifact`, `_rmx_search`, `_rmx_tailwind`, `_rmx_extension`) all sync.

### [Feb 27] "Constants not found" modal replaced with auto-rebuild (Simon)

Simon merged improved rebuild detection — short-lived blocking modal removed; auto-detect + auto-rebuild now triggers silently.

**Regression reported by Arvind:** `COMPILER: LINE 2593. MODULE NOT FOUND: COMP_SYMBOLS_GET_DATA_USING_SQL` errors appear on screens after auto-compile, especially with library-synced components and Symbols. Workaround: save + close screen, reopen — error goes away. Unlinking the symbol also clears it.

**Resolution direction (Arvind + Didier):** Block UI + show spinner during full rebuild; don't go back to old modal. Didier: full rebuild can take 45s — only rebuild constants module if possible. Simon: false positive toast also present, will fix.

### [Feb 28] Arrow keys create tofu characters in text inputs (Simon/Didier/Tyler)

Still not fixed in production as of Feb 28. Present in prod, absent in dev.

- Didier confirmed: reproducible in L0 projects searchbox
- Simon: issue appears in prod but not dev environment (different Tauri/webview build?)
- Tyler: doesn't happen if DevTools console is open — "it's weird"

---

## Channel Tone & Patterns

- Primary participants: Benedikt (desktop backend), Tyler (Tauri/webcomps), Didier (sync/runtime), Arvind (product/design), Gerd (platform/compiler), Simon (builder frontend)
- Very technical; PRs linked inline, bugs cross-posted to #bugbash
- Tensions: browser-vs-native debate resurfaces frequently; Tauri limitations are a recurring source of friction
- Windows desktop support is manual/secondary (no auto-update, Fred distributes builds)