# #bugbash Slack Channel — Remix Labs

**Coverage:** Jan 20 – Feb 28, 2026
**Channel ID:** C862WHQMS
**Bug reporting guide (Feb 13):** https://www.notion.so/Bug-reporting-3061d464528f80cdacf7eed2612bad07

---

## Open / In-Progress Bugs

### 🔴 App meta URL — no portable `baseURL` across surfaces (Arvind, Feb 28)

- `remix://localhost/a/v1/ws/local/appmeta/{{app}}` stopped working on desktop; `http://localhost:2025/v1/ws/local/appmeta/{{app}}` works but is hardcoded to default port
- **Core problem**: no single `baseURL` primitive works across desktop, web runtime, and embed (web component); `baseURL` in Elm points to assets URL, not mixer URL
- **Benedikt's proposal**: host environments expose a typed struct to Mix (e.g. `{host: "desktop", localMixerUrl: "...", authWorkspace: ...}`); stdlib provides stdlib-level abstractions on top; user
  code handles surface-specific differences
- **Status**: no decision yet; Arvind needs a portable solution for Lumber delivery next week

### 🔴 System plugins can't run on desktop after flush (Gerd, Feb 28)

- After ~100 writes trigger a flush, system plugins (e.g. `_rmx_admin`-based) fail: `flat._rmx_id.off` path is wrong — a `path.Join` was copy-pasted inside another `path.Join`, double-nesting the path
- **Fix**: mix-rs/pull/1036 (Fred); approved by Gerd; Chris: fix in beta, "0.10100.0 should have the fix; will update beta across the board after CI"
- **Status**: fix in beta, not yet on prod

### 🔴 Snowflake-hosted runtime screens fail: `runtime.json` 410 (Chris, Feb 28)

- Chris fixed asset loading issues (`rmx-fullscreen.js/css`) but Snowflake-hosted runtime screens now fail with 410 for `runtime.json`
- Repro: https://mw7zsh-oxfsvki-remix.snowflakecomputing.app/e → new app → open home screen
- Simon tagged to investigate
- **Status**: unresolved

### 🔴 Desktop stopped responding: make-agent VM unclosed (Simon, Feb 28)

- Desktop stopped responding; make-agent VM not closed after use; worker passes the error back to the caller
- Gerd + Benedikt to investigate the worker-level error propagation
- **Status**: unresolved

### 🔴 Stdlib version bump → "project needs refreshing" modal (Arvind, Feb 17)

- After a desktop platform update, opening any L2 shows an error asking for a full rebuild
- **Root cause**: stdlib version bump invalidates pre-compiled code; builder doesn't detect this
- **Fix in progress**: Simon's turntable/pull/11731 — detects version mismatch at L1 and shows a modal ("This project needs to be refreshed") with OK + Cancel buttons; Cancel goes back to L0
- Reading `.mixexe.info` file via `file-manager` API to get stdlib version
- Workaround: do a full `make` at L1

### 🔴 Tailwind styles out of sync (preview ≠ runtime) (multiple reporters, ongoing)

- Tailwind styles visible in preview/builder but absent or wrong at runtime
- Specific cases: opacity `0.25` rendered as `25`; nav styles lost entirely
- **GitHub issue**: turntable/issues/11716
- **Workaround** (always works): open Styles module → save & make (even with no changes)
- **Root cause**: unclear; suspected when external styles reference is added without re-saving Styles module; no reliable repro yet — Didier investigating
- Note: `make` alone should regenerate styles but sometimes doesn't; full `rebuild` never needed

### 🟡 _rmx_tailwind Constants module error on dev (Arvind, Feb 25)

- Opening Styles module on remix-dev `_rmx_tailwind` throws Constants rebuild error; no "make" button available at L1
- Workaround used: rebuild on remix-beta, re-export as v2 .remix, install on desktop; leaves the bug on dev intentionally for Simon to debug
- **Root cause**: stdlib diff between dev and beta means "make" modal triggers on beta but not dev; Constants out of sync silently
- **Status**: Simon to investigate; preserved on remix-dev for repro

### 🟡 Tailwind compiler stack overflow (Gerd, Feb 26)

- Too many tailwind styles → compiler hits stack overflows at compile time; can't `make` tailwind on desktop
- **Fix**: Gerd fixing but "a full fix will not be available shortly"
- **Workaround**: compile on amp (server); or remove `[app_versions]` entry from `~/Library/Application Support/com.remixlabs.desktop/config.toml` and restart to force re-install
- **Status**: partial workaround; full fix in progress

### 🟡 Large action tile input overflows CSS, can't hit EDIT (Gerd, Feb 25)

- Custom action tile with large bound input (e.g. large JSON object) overflows visually; EDIT button unreachable
- Affects both desktop and remix-dev
- Repro: https://remix-dev.remixlabs.com/e/edit/overflow/home (large JSON → custom action tile)
- No issue filed; no fix yet
- **Status**: open

### 🟡 Builder doesn't show propagation results on first agent invoke (Gerd, Feb 25)

- Results exist in messages (`out_3`, `out_4` have data) but not displayed in builder until second invocation
- Also confirmed by Arvind and Simon (Simon: intermittent — gets cell values but no echo, or vice versa)
- Filed: protoquery/issues/2279
- **Status**: open issue

### 🟡 Monaco widget: exceptions on every mouse click in Mix editor (Gerd, Feb 26)

- Running desktop from browser with URL parameters causes JS exceptions every time mouse clicks into a Mix code editor
- Didier: happens on local machines; remix-dev etc. generally fine
- **Status**: environment-specific; no fix

### 🟡 File browser + drag-and-drop not working in Firefox (Sirshendu, Feb 26)

- `Browse Files` and drag-and-drop file inputs fail for Sirshendu in Firefox; works in Chrome; Padmanabha confirmed Chrome works
- Simon: could reproduce intermittently in Firefox
- **Status**: inconsistent repro; Firefox-specific

### 🟡 ACTION REQUIRED: Deploy rmx-sync/prefs/search to all customer workspaces (Didier, Feb 28)

- Latest `rmx-sync` on prod fixes dedup issue (last-one-wins when app redefined across sync groups)
- Need to deploy updated `rmx-sync`, `rmx-prefs`, `rmx-search` to every customer workspace where users auth
- Also: `catalog_list` tool deprecated (only exports v1 .remix); App Hub stays; Arvind migrating to v2 exports
- **Owner**: Arvind (taking over v2 exports, including App Hub entries); John/Wilber for workspace deployment
- **Status**: pending

### 🟡 Can't delete screen: "too many files open" error (Wilber, Feb 19)

- Deleting a screen (or saving) fails with OS file limit error
- **Root cause**: blob leak (`URL.createObjectURL` without `revokeObjectURL`); DELETE of ~300 records triggers parallel `get_by_id` calls each opening `db.idx`
- Issues: mix-rs/issues/1005, mix-rs/issues/789
- Benedikt: repro'd, discussing fix with Fred

### 🟡 L2 parameters not passed correctly into new tab (Wilber, Feb 19)

- `%20` (space) in URL params gets mangled to `+` when opening a new builder tab
- **Root cause**: `form_urlencoded` decodes `%20` → space, then re-encodes as `+` (HTML form standard, wrong for URLs)
- **Fix**: Benedikt's PR mix-rs/pull/1003 (avoid decode/re-encode entirely), Simon reviewing

### 🟡 Sign-in issue in container app (Wilber, Feb 19, v2589)

- Sign-in fails on first attempt; works on second attempt
- Issue filed: amp/issues/2789; Chris investigating logs

### 🟡 Deleted agent still appears in first deployment (Wilber, Jan 21)

- Deleted service agent still included when deploying project for the first time
- **Root cause**: deleting an agent in builder doesn't remove its compiled binaries; they get packed into deployment
- **Workaround**: manually delete `/code/<agent_name>.*` files before deploying
- Gerd: "The fix will need some time"

### 🟡 rmx-tui-grid doesn't re-render after async data arrival (Padmanabha, Jan 20)

- Grid web component stays empty until connectors are manually disconnected + reconnected
- Occurs with Snowflake API calls; doesn't repro with simple sync data
- **Root cause**: suspected race condition between attribute update and webcomp re-render (seen with other webcomps too)
- **Workaround**: add a "Set Value" node before the grid
- Tyler: added to investigation list; needs solid repro with accessible data

### 🟡 System/bootstrap modules can be deleted (Arvind, Feb 17)

- Filed: turntable/issues/11725 — system modules should not be deletable

### 🟡 Option key loses focus during L2 node rename (Padmanabha, Jan 29)

- Pressing Option (⌥) while renaming a node loses focus; also causes error toasts
- Unanswered — Didier tagged but no response yet

### 🟡 Desktop builder blank-out (Benedikt, Feb 2) — tracked

- Studio becomes unresponsive (specific repro unknown from available data)
- Tracked: 📝 memo reaction, 2 replies

### 🟡 Desktop memory consumption (Wilber, Jan 28)

- Two Studio windows consuming 4+ GB; only partially reclaims on tab close
- Known architectural issue: compiler runs once per window (~600MB baseline + temp spikes)
- No active fix planned

### 🟡 Settings module rebuild shows anomalous dialog on amp (Arvind, Feb 20)

- Rebuilding Settings after editing/saving Symbols sheet shows unexpected OS-style dialog
- URL: https://remix.remixlabs.com/e/edit/_rmx_settings — Simon investigating, no resolution yet

### 🟡 Orderly: GCS install via carousel doesn't add to Recents (Arvind, Feb 20)

- Direct install from GCS versioned source (carousel path) skips "Open" screen, so flow doesn't land in Recents after auto-navigate
- Other Recents issues resolved; this specific gap remains

---

## Known Issues (no active fix)

- **save & make in Settings module always errors** (Didier, Feb 11): known since Settings L2 was introduced; plan to disable the "Save & Make" button in settings (Simon: needs redesign)
- **Error page "send error report" form errors** (Didier, Feb 9): `_rmx_errorFallback.mix` was stale; Didier synced from prod (protoquery/pull/2264); longer-term: needs productised error reporting UX

---

## Recently Fixed (since Jan 20)

| Date   | Bug                                                       | Fix                                    |
|--------|-----------------------------------------------------------|----------------------------------------|
| Feb 28 | System plugins can't run (path.Join double-nested)        | Fix in beta v0.10100.0 (Fred #1036)    |
| Feb 25 | Packaged app (v2 exec-only) installs as Project not Pkg   | Fixed v0.9948.0 (Gerd pq/pull/2277)    |
| Feb 24 | Can't logout on desktop (menu item confusion)             | Exists in Remix menu; standardized     |
| Feb 24 | Local agent calls failing in Desktop (macOS + Windows)    | Fixed (Gerd, PRs merged)               |
| Feb 23 | Default installed package flashing red in Packages tab    | App had error at publish time; fixed   |
| Feb 24 | Deleted widget node: can't rebuild project                | Fixed (Fred mix-rs/pull/1022)          |
| Feb 17 | remix-dev loading error                                   | Fixed (Chris)                          |
| Feb 17 | wasm/callstack error on desktop                           | Fixed                                  |
| Feb 16 | "Unsupported start_node" on Query tile                    | Fixed                                  |
| Feb 16 | workit/approve builder error                              | Fixed                                  |
| Feb 14 | Wrapping issue in builder                                 | Fixed                                  |
| Feb 13 | Upload File: private file 403 error                       | Fixed                                  |
| Feb 11 | iOS: installed .remix not in All Flows/Recents            | Fixed (partial — carousel gap remains) |
| Feb 11 | Desktop copy: JSON pasted as card not object              | Fixed                                  |
| Feb 10 | "invalid magic" loading remix_labs assets (disk incident) | Fixed (Chris restarted pod)            |
| Feb 2  | Component always marked * (recompile needed)              | Fixed                                  |
| Feb 2  | Wasm compiler crash, can't re-invoke                      | Fixed                                  |
| Jan 29 | Unable to close desktop builder windows                   | Fixed                                  |
| Jan 27 | Blocked/unresponsive desktop studio UI                    | Fixed                                  |
| Jan 27 | Tailwind styles corruption on desktop make                | Fixed                                  |
| Jan 27 | Auth workspace+server not passed back to builder          | Fixed                                  |
| Jan 27 | Fresh desktop: Google sign-in fails first time            | Fixed                                  |
| Jan 26 | File corruption in agent server (mix-rs/pull/961)         | Hotfixed                               |
| Jan 26 | poc.remixlabs.com workspace tools 403 for external user   | Fixed                                  |
| Jan 21 | Open in new tab navigation doesn't work                   | Fixed (Simon, mix-rs/pull/975)         |
| Jan 21 | Files section: uploads result in 0 bytes                  | Fixed (Simon, turntable/pull/11667)    |