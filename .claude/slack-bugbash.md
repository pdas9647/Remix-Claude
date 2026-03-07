# #bugbash Slack Channel — Remix Labs

**Coverage:** Jan 20 – Mar 7, 2026
**Channel ID:** C862WHQMS
**Bug reporting guide:** https://www.notion.so/Bug-reporting-3061d464528f80cdacf7eed2612bad07

---

## Open / In-Progress Bugs

### No portable `baseURL` across surfaces (Arvind, Feb 28) 🔴
`remix://localhost/a/...` stopped working on desktop; `http://localhost:2025/...` hardcoded to default port. No single `baseURL` works across desktop, web, and embed. Benedikt proposed typed host-env struct in Mix. **No decision yet; needed for Lumber delivery.**

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

### State machine editor L3 hover styles broken (Didier, Mar 6) 🔴
Mouse-over styles for state machine editor at L3 no longer working — CSS animations move hover icons away from state tiles. Webcomp rendering is fine; high-level CSS broke it. **Unresolved.**

### Cannot remove connections in builder (Gerd, Mar 4) 🔴
"Remove invalid connection" button and backspace key fail to delete connections under certain unknown circumstances. Triggered by changing input type (URL→Binary). Simon could not repro. Gerd hit it again without error markers. **Unresolved.**

### Published .remix missing styles from desktop (Mark, Mar 5) 🟡
Created app on desktop with `_rmx_tailwind`, published via publish plugin, used as catalog item source → all styles missing. Workaround: open empty Styles module, Save & Make, republish. Didier suspects cross-instance export (prod→desktop) may cause it. No confirmed root cause.

### Desktop toasts cut off / mispositioned (Arvind, Mar 4) 🟡
Toast notifications sometimes show cut off, sometimes appear bottom-left. No repro steps available.

### Theme auto-make error referencing _rmx_auth (Arvind, Mar 3) 🟡
Opening theme on desktop triggers auto-make that errors referencing `_rmx_auth` screen. Themes should not need full make (only constants recompile). Proposed fix: add themes to "fake cases" list. **Not confirmed fixed.**

### File download in Desktop opens without auth (Benedikt, Mar 3) 🟡
Workspace Tools opens file URL in browser without auth (anonymous access error). Files from L1 opens URL inside studio instead of new tab. Simon: need desktop client action for "save as" flow. Benedikt proposed `Content-Disposition: attachment` header on mixer. **No fix yet.**

---

## Known Issues (no active fix)

- **Save & Make in Settings always errors** (Didier, Feb 11): known since Settings L2 introduced; plan to disable button (Simon: needs redesign)
- **Error page "send report" form errors** (Didier, Feb 9): `_rmx_errorFallback.mix` was stale; synced from prod (pq/pull/2264); needs productized error reporting UX
