# #desktop Slack Channel — Remix Labs

**Coverage:** Dec 20, 2025 – Feb 22, 2026
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

## Channel Tone & Patterns

- Primary participants: Benedikt (desktop backend), Tyler (Tauri/webcomps), Didier (sync/runtime), Arvind (product/design), Gerd (platform/compiler), Simon (builder frontend)
- Very technical; PRs linked inline, bugs cross-posted to #bugbash
- Tensions: browser-vs-native debate resurfaces frequently; Tauri limitations are a recurring source of friction
- Windows desktop support is manual/secondary (no auto-update, Fred distributes builds)