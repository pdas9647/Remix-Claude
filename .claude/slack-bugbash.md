# #bugbash Slack Channel — Remix Labs

**Coverage:** Jan 20 – Feb 22, 2026
**Channel ID:** C862WHQMS
**Bug reporting guide (Feb 13):** https://www.notion.so/Bug-reporting-3061d464528f80cdacf7eed2612bad07

---

## Open / In-Progress Bugs

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