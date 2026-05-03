# #bugbash Slack Channel — Remix Labs

**Coverage:** Jan 20 – May 2, 2026
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

### Propagation results not shown on first invoke (Gerd, Feb 25) 🟡

Results exist in messages but not displayed until second invocation. Intermittent. Filed: protoquery/issues/2279.

### Deploy rmx-sync/prefs/search to customer workspaces (Didier, Feb 28) 🟡

Latest `rmx-sync` on prod fixes dedup. Need to deploy to every customer workspace. `catalog_list` tool deprecated (v1 only); Arvind migrating to v2 exports. Owner: Arvind + John/Wilber.

### L2 params `%20` → `+` mangling (Wilber, Feb 19) 🟡

`form_urlencoded` decodes space then re-encodes as `+`. Fix: mix-rs/pull/1003 (Benedikt), Simon reviewing.

### Deleted agent still in first deployment (Wilber, Jan 21) 🟡

Compiled binaries not removed when agent deleted in builder. Workaround: manually delete `/code/<agent>.*` before deploying.

### Cloud subscribe not updating in Studio L2 (Arvind, Mar 9, 60 replies) 🔴

Async cloud subscribe messages arrive at frontend but not rendered until next user interaction (echo). Adding any node forces propagation. Root cause: amp/wasm discrepancy — protoquery#2281 fix was
desktop-only, not amp. Gerd: "one of the reasons we want to drop amp." Simon confirmed works for wasm mixc. Simon + Gerd discussing solution; turntable#11795 adds unfreeze for amp. **In progress —
partial fix.**

### Cell propagation count shows 0 on screens (Gerd, Mar 12) 🟡

Works for components but not screens. Works in browser but not desktop. Simon has PR up for review.

### Get State value not passing into component boundary (Arvind, Mar 11) 🟡

Set state works, get state shows value, but value doesn't pass as in_param when editing the component. Repro asset shared. Simon reviewing.

### Read-only flag ineffective on checkboxes (Gerd, Mar 11) 🟡

Read-only attribute doesn't work for checkboxes, at least on desktop.

### Desktop crash from malformed claude_desktop_config.json (Didier, Mar 11) 🟡

"trailing characters at line 12 column 2" crashes desktop on startup. Root cause: malformed Claude config JSON. Fix: mix-rs#1039 (shouldn't be fatal). Workaround: re-save the JSON file.


### State machine editor L3 hover styles broken (Didier, Mar 6) 🔴

Mouse-over styles for state machine editor at L3 no longer working — CSS animations move hover icons away from state tiles. Webcomp rendering is fine; high-level CSS broke it. **Unresolved.**

### Cannot remove connections in builder (Gerd, Mar 4) 🔴

"Remove invalid connection" button and backspace key fail to delete connections under certain unknown circumstances. Triggered by changing input type (URL→Binary). Simon could not repro. Gerd hit it
again without error markers. **Unresolved.**

### Published .remix missing styles from desktop (Mark, Mar 5) 🟡

Created app on desktop with `_rmx_tailwind`, published via publish plugin, used as catalog item source → all styles missing. Workaround: open empty Styles module, Save & Make, republish. Didier
suspects cross-instance export (prod→desktop) may cause it. No confirmed root cause.


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

## Apr 11–18, 2026 New Open Bugs

### D&D .remix (auth required) in anon mode fails (Didier, Apr 16) 🟡

Dragging a `.remix` app requiring auth into Desktop then running in anon mode returns an error. Repro: create+export any prod app, drag into Desktop, run in anon mode. Simon couldn't repro using an
anon-allowed app. **Unresolved.**

## Apr 18–25, 2026 New Open Bugs

### http.get timeout param crashes mixrt (Gerd, Apr 23) 🔴

`http.get(..., timeout: 50)` panics mixrt with "invalid type: number, expected null or option variant some" deserialization error. **New regression; unresolved.**

### Workspace tooling inaccessible on desktop dev (Arvind, Apr 23) 🔴

Query returns "Database cloud_workspace does not exist" — all workspace tooling dead on desktop dev. Showstopper. cc: Benedikt, Fred. **Unresolved.**

### Styling broken on remix.remixlabs.com/remix_app/home (Simon, Apr 24) 🟡

Production home URL has broken styles. Screenshot shared with Arvind + Mark. **Unresolved.**

### Token corrupted on remixlabs.com/experience_chat.html (Mukund, Apr 24) 🟡

Token corrupted error visible on the public experience_chat page. **Unresolved.**

### snowflake_snowflake_demo fails to build: item_7 not found (Padmanabha, Apr 23) 🟡

`snowflake_snowflake_demo` on remix-india fails — `variable not found: item_7` across 19 modules. Was working 4–5 months ago. **Unresolved.**

### Web component error in customers page (Arpita, Apr 20) 🟡

Web component throwing console errors on customers page and other pages with web components. Sirshendu unable to repro. **Unresolved.**

## Apr 28–May 2, 2026 Unresolved Bugs

**Agent add slow: "Creating N-1 modules" [Apr 28, Arvind/Gerd]:** Adding a single agent triggers full rebuild popup and is slow. Root cause: frontend compiler + backend file storage → many small
round-trips (< 10KB, 3–5ms each). No fix yet.

**File-manager crash on stale path [Apr 30, Benedikt]:** Obsolete `channel/<ch>/version/<v>/` R2 file brings down `file-manager/content` listing + cloud workspace plugin deploy. Bad files deleted (
Chris); systemic fix open: mix-rs#1064.

**`rmx-remix` webcomp issues [Apr 30, Didier/Benedikt]:** (1) Double embed bug → turntable/11979 (Didier, in progress); (2) Mixcore `"deprecated parameters"` init warning in all webcomps on dev (
Benedikt investigating). Symptom: Snowflake landing page web comp sections intermittently invisible.
