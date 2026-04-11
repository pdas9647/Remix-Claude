# #bugbash Slack Channel — Remix Labs

**Coverage:** Jan 20 – Apr 11, 2026
**Channel ID:** C862WHQMS
**Bug reporting guide:** https://www.notion.so/Bug-reporting-3061d464528f80cdacf7eed2612bad07

---

## Open / In-Progress Bugs

### No portable `baseURL` across surfaces (Arvind, Feb 28) 🔴

`remix://localhost/a/...` stopped working on desktop; `http://localhost:2025/...` hardcoded to default port. No single `baseURL` works across desktop, web, and embed. Benedikt proposed typed host-env
struct in Mix. **No decision yet; needed for Lumber delivery.**

### Snowflake runtime `runtime.json` 410 (Chris, Feb 28) 🔴

Asset loading fixed but runtime screens return 410 for `runtime.json`. Repro: `mw7zsh-oxfsvki-remix.snowflakecomputing.app/e`. Simon tagged. **Unresolved.**

### Desktop unresponsive: make-agent VM unclosed (Simon, Feb 28) 🔴

VM not closed after use; worker error propagation under investigation (Gerd/Benedikt). **Unresolved.**

### Stdlib version bump → "needs refreshing" modal (Arvind, Feb 17) 🔴

Platform update invalidates pre-compiled code. Fix in progress: turntable/pull/11731 (detects mismatch, shows modal). Workaround: full `make` at L1.

### Tailwind styles out of sync: preview ≠ runtime (ongoing) 🟡

Styles visible in builder but absent/wrong at runtime. turntable/issues/11716. Workaround: open Styles module → save & make. Root cause unclear; Didier investigating.

### `_rmx_tailwind` Constants error on dev (Arvind, Feb 25) 🟡

Styles module on remix-dev throws Constants rebuild error; no "make" button at L1. Preserved on remix-dev for Simon to debug.

### Tailwind compiler stack overflow (Gerd, Feb 26) 🟡

Too many styles → stack overflow at compile time. Workaround: compile on amp, or remove `[app_versions]` from config.toml and restart. Gerd fixing; full fix not imminent.

### Large action tile input overflows CSS (Gerd, Feb 25) 🟡

Large bound input overflows; EDIT button unreachable. Repro: `remix-dev.remixlabs.com/e/edit/overflow/home`. No fix.

### Propagation results not shown on first invoke (Gerd, Feb 25) 🟡

Results exist in messages but not displayed until second invocation. Intermittent. Filed: protoquery/issues/2279.

### Monaco exceptions on mouse click in Mix editor (Gerd, Feb 26) 🟡

JS exceptions on every click in Mix code editor when running desktop from browser. Environment-specific (local machines only).

### File browser / D&D broken in Firefox (Sirshendu, Feb 26) 🟡

Browse Files and drag-and-drop fail in Firefox; works in Chrome. Inconsistent repro.

### Deploy rmx-sync/prefs/search to customer workspaces (Didier, Feb 28) 🟡

Latest `rmx-sync` on prod fixes dedup. Need to deploy to every customer workspace. `catalog_list` tool deprecated (v1 only); Arvind migrating to v2 exports. Owner: Arvind + John/Wilber.

### L2 params `%20` → `+` mangling (Wilber, Feb 19) 🟡

`form_urlencoded` decodes space then re-encodes as `+`. Fix: mix-rs/pull/1003 (Benedikt), Simon reviewing.

### Sign-in fails first attempt in container app (Wilber, Feb 19) 🟡

Works on second attempt. amp/issues/2789; Chris investigating.

### Deleted agent still in first deployment (Wilber, Jan 21) 🟡

Compiled binaries not removed when agent deleted in builder. Workaround: manually delete `/code/<agent>.*` before deploying.

### rmx-tui-grid doesn't re-render after async data (Padmanabha, Jan 20) 🟡

Grid stays empty until connectors reconnected. Race condition suspected. Workaround: add Set Value node before grid. Tyler investigating.

### System/bootstrap modules deletable (Arvind, Feb 17) 🟡

turntable/issues/11725 — should not be deletable.

### Option key loses focus during L2 node rename (Padmanabha, Jan 29) 🟡

Pressing ⌥ while renaming loses focus + error toasts. Unanswered.

### Desktop memory consumption (Wilber, Jan 28) 🟡

Two windows = 4+ GB; compiler runs once per window (~600MB baseline). No active fix planned.

### Settings module rebuild shows anomalous dialog on amp (Arvind, Feb 20) 🟡

Unexpected OS-style dialog when rebuilding Settings after editing Symbols. Simon investigating.

### Orderly: GCS carousel install skips Recents (Arvind, Feb 20) 🟡

Direct GCS install doesn't land in Recents.

### Cloud subscribe not updating in Studio L2 (Arvind, Mar 9, 60 replies) 🔴

Async cloud subscribe messages arrive at frontend but not rendered until next user interaction (echo). Adding any node forces propagation. Root cause: amp/wasm discrepancy — protoquery#2281 fix was
desktop-only, not amp. Gerd: "one of the reasons we want to drop amp." Simon confirmed works for wasm mixc. Simon + Gerd discussing solution; turntable#11795 adds unfreeze for amp. **In progress —
partial fix.**

### ~~Curly quotes in desktop Studio~~ (Arvind, Mar 13) ✅

PR mix-rs#1056 approved Mar 27; deployed.

### Cell propagation count shows 0 on screens (Gerd, Mar 12) 🟡

Works for components but not screens. Works in browser but not desktop. Simon has PR up for review.

### Get State value not passing into component boundary (Arvind, Mar 11) 🟡

Set state works, get state shows value, but value doesn't pass as in_param when editing the component. Repro asset shared. Simon reviewing.

### Read-only flag ineffective on checkboxes (Gerd, Mar 11) 🟡

Read-only attribute doesn't work for checkboxes, at least on desktop.

### Desktop crash from malformed claude_desktop_config.json (Didier, Mar 11) 🟡

"trailing characters at line 12 column 2" crashes desktop on startup. Root cause: malformed Claude config JSON. Fix: mix-rs#1039 (shouldn't be fatal). Workaround: re-save the JSON file.

### Desktop DMG missing Applications shortcut icon (Didier, Mar 12) 🟡

DMG installer shows no proper icon for Applications folder. Cosmetic.

### State machine editor L3 hover styles broken (Didier, Mar 6) 🔴

Mouse-over styles for state machine editor at L3 no longer working — CSS animations move hover icons away from state tiles. Webcomp rendering is fine; high-level CSS broke it. **Unresolved.**

### Cannot remove connections in builder (Gerd, Mar 4) 🔴

"Remove invalid connection" button and backspace key fail to delete connections under certain unknown circumstances. Triggered by changing input type (URL→Binary). Simon could not repro. Gerd hit it
again without error markers. **Unresolved.**

### Published .remix missing styles from desktop (Mark, Mar 5) 🟡

Created app on desktop with `_rmx_tailwind`, published via publish plugin, used as catalog item source → all styles missing. Workaround: open empty Styles module, Save & Make, republish. Didier
suspects cross-instance export (prod→desktop) may cause it. No confirmed root cause.

### Desktop toasts cut off / mispositioned (Arvind, Mar 4) 🟡

Toast notifications sometimes show cut off, sometimes appear bottom-left. No repro steps available.

### Theme auto-make error referencing _rmx_auth (Arvind, Mar 3) 🟡

Opening theme on desktop triggers auto-make that errors referencing `_rmx_auth` screen. Themes should not need full make (only constants recompile). Proposed fix: add themes to "fake cases" list. *
*Not confirmed fixed.**

### File download in Desktop opens without auth (Benedikt, Mar 3) 🟡

Workspace Tools opens file URL in browser without auth (anonymous access error). Files from L1 opens URL inside studio instead of new tab. Simon: need desktop client action for "save as" flow.
Benedikt proposed `Content-Disposition: attachment` header on mixer. **No fix yet.**

---

## Known Issues (no active fix)

- **Save & Make in Settings always errors** (Didier, Feb 11): known since Settings L2 introduced; plan to disable button (Simon: needs redesign)
- **Error page "send report" form errors** (Didier, Feb 9): `_rmx_errorFallback.mix` was stale; synced from prod (pq/pull/2264); needs productized error reporting UX

### Wasm component dep clash: crashes remix.app + Desktop plugins + workspace form (Wilber/Benedikt, Mar 19) 🔴

`__wasm_bindgen_func_elem_2405 is not a function` on dev.remix.app when loading remix files. Root cause: transitive dependency clash in rcm components. Affects **remix.app**, **Desktop plugins**, and
**Desktop workspace form**. Repro: https://dev.remix.app/about/remix. Benedikt investigating; discussion in #C027XQTJG6P. **Unresolved / Critical.**

### File download button in Desktop freezes app (Didier → Benedikt/Simon, Mar 18) 🔴

Downloading a file from the files tile throws an error with no way to close the window — must quit and restart Desktop. Simon: missed applying same URL check to the download button (applied to get_url
only). Needs a tauri-specific solution or general file-access-by-URL approach. Talk scheduled between Simon + Benedikt for Monday Mar 23. Related to the existing "File download opens without auth"
issue (Mar 3). **Unresolved.**

### Paste callable comp → "unknown target" build error (Didier/Simon, Mar 17–18) 🟡

Pasting a screen containing callable comps triggers "unknown target" make error on Desktop (client VM). Works on amp. Root cause: 2 distinct bugs. Simon PR: turntable#11823; Didier parallel fix:
turntable#11820. Arvind proposed bidirectional review to pick the less risky one. **PRs under review.**

### L0 collaborator list not updated on L1 paste/delete (Arvind, Mar 20) 🟡

Pasting modules, deleting modules, or doing make at L1 doesn't add user to the `collaborators` list, even though `lastModified` updates correctly. Only L2 node moves trigger a collaborator update.
Simon's fix turntable#11830 adds collaborator on paste. Broader open question: should `make` update `lastModified` at all? **Partial fix in PR; design question open.**

### Repo writer comma format locks creator out (Mark/Gerd, Mar 20) 🟡

Adding multiple writers to a Repository project with comma-separated emails (e.g. `a@b.com, c@d.com`) instead of space-separated corrupts the writers list and locks the creator out. Workaround:
manually edit `projects/<Name>.json` in the Repository cloud files (the `writers` field). Root fix needed: UI validation + creator should always retain permission. **No fix yet.**

---

## Mar 21 – Apr 3 New Open Bugs

### IndexDB → OPFS migration: all plugins broken (Vijay, Mar 28) 🔴

OPFS migration fails on upgrade ("OPFS already exists"); all plugins dead. Debug build v10809 (`bb-debug-indexdb-migration`) didn't fix. Root: `createWritable` undefined. **Unresolved.**

### member_of + entity filter empty in mixer (Wilber, Mar 31) 🔴

Combined `member_of` + `entity` filter returns no results on desktop/mixer; works on amp. Fix: mix-rs/pull/1085 — awaiting review.

### Node titles expand unboundedly in builder (Didier/Arvind, Apr 2) 🔴

Flex layout change broke node width bounding — long names stretch nodes. PR turntable/pull/11888 partial; broader CSS fix pending. **Must land before beta/prod.**

### Type select styling broken in Safari/desktop (John, Apr 3) 🟡

Styled HTML `<select>` unsupported in WebKit (desktop). Arvind to push CSS fix.

### Published .remix stale in catalog (Mark, Mar 25) 🟡

Republish shows old version. Root: missing `Cache-Control: no-cache` on uploads; needs code change. Workaround: change filename per publish.

### New web components + file upload button broken in desktop (Vijay, Apr 1) 🟡

Net-new web components blank on desktop (work on remix/remix-dev). "Choose files" button dead in plugins; drag/drop works. Under investigation.

### App Clips "file code refs not allowed" on dev (Wilber, Mar 25) 🟡

Old Wasm-executable .remix files fail on dev.remix.app; work on beta/prod. Benedikt adding v2 support. Workaround: re-export as bytecode.

### Deploying to local workspace converts app to executable (Simon/Gerd, Mar 27) 🟡

Deploying agent to `localhost/local` marks appmeta `binary` — no recovery. Gerd: disallow deploys to local. **No fix yet.**

## Apr 4-11 New Open Bugs

### Desktop keyboard shortcuts broken (Mark, Apr 10) 🟡

`Cmd+A`/`Cmd+K+C` don’t work in function nodes; `Cmd+A` fails to select all at L1/L2. Benedikt will look.

### Component drill-in/out wrong level (Mark, Apr 10) 🟡

Builder ends at wrong level after drilling in/out. Didier added footer safety code (turntable/pull/11917); root cause not repro’d.
