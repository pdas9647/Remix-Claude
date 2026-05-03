# #backend Slack Channel — Remix Labs

**Coverage:** Jan 2 – May 2, 2026
**Channel ID:** CHTGC5BGQ
**Purpose:** server, compiler, and database stuff

**Key participants:** Chris (backend lead, mixer/auth/deploy), Gerd (compiler/stdlib/caching), Fred (DB internals, mixquery Rust), Simon (builder integration), Benedikt (desktop/Tauri/OPFS)

---

## Key Decisions & Technical Discussions

### DB save FFI overhaul [Jan 13-28, 89 replies]

Bug mix-rs/issues/913: saving unchanged record threw FFI errors. Fred built new save FFIs returning case types: `db.saved(record)`, `db.rev_match_current(record)`, `db.rev_match_old(record)`. New
FFIs: `$db_save_ar`, `$db_upsert_ar`. Old FFIs preserved for backward compat. Non-array save/upsert deprecated since 2022. Merged to mix-rs + amp; stdlib: protoquery/pull/2252.

### Wasm caching fix [Jan 26-27, 49 replies]

`mixrt.wasm` downloaded every page load (~8s for two loads per L2 open). Root cause: nginx gzip converts strong ETag to weak → no 304. Fix: upload pre-compressed with `gsutil cp -Z` → strong ETag
preserved. PR: turntable/pull/11660. Also: mix-rs/issues/962 (mixer Cache-Control), mix-rs/issues/963.

### QB 2.0 field discovery [Feb 3-13, 31 replies]

Simon needs field metadata for query tile UI (which fields are refs). Fred added type inference from index tokens — `"type": "ref"` when token conforms to ref format. Heuristic-based (tokens don't
store types natively). Simon confirmed working Feb 13.

### Delete FFI backward compat break [Feb 10-11, 27 replies]

`db.delete` changed to return case value → broke tests + auto-update. **Lesson:** never change existing FFI signatures; add new ones (`delete_ar`). Fix: mix-rs/pull/992.

### Non-ES256 JWT [Feb 17]

mix-rs/pull/1001 (Chris): supports RS256 etc. Lumber's Descope tokens now work with mixer. Not related to SAML.

### Desktop ↔ Mixer dev workflow [Feb 17-20]

`mixer serve --ws-dir <desktop-dir> --host localhost --port 2025` works but no R2 presigned URLs for `file.register`. `--base-url` flag exists; mix-rs/pull/948 unifies startup.

### File upload PUT semantics [Feb 18]

`file.register` returns R2 presigned URL; PUT overwrites by default (S3/R2 standard). Use `If-None-Match: *` to prevent overwrite. Not changeable server-side.

---

## Feb 25–28 Additions

**Deletion / too-many-files-open [Feb 25]:** `join_all` in `find_recs_for_deletion` opened too many file handles. Fix: sequential `for` loop; `get_by_id` now IO-free. Also fixed `exact_match`
over-optimization in `get_by_ref`. mix-rs/pull/1022, merged.

**High-priority flush patch [Feb 26]:** Critical bug in deletion/flush pipeline (via Benedikt). mix-rs/pull/1028, merged same day.

**`$binary_compress` FFI missing in CI [Feb 27]:** Gerd's compress/decompress PRs needed amp registration. Fix: amp/pull/2790, merged. Broader: need "builtin-with-fallback" FFI category between
`builtin` and `FFI`.

**Amp dev blocked [Feb 27]:** System plugins path bug (double `path.Join`). Fix: mix-rs/pull/1036; resolved when CI completed.

**.DS_Store corrupts Desktop DB [Feb 26]:** Finder creates `.DS_Store` in workspace folder → API hosed. One-off cleanup: `find .../workspaces/local/ -name .DS_Store -print0 | xargs -0 rm`.

**JSON files served as binary [Feb 27]:** `.info` extension unrecognized → binary content-type. Open: mix-rs/issues/1034.

**ARM64 Linux mixer build [Feb 28]:** Lumber requested. mix-rs/pull/1026, merged.


## Mar 2–6 Additions

**Live queries not supported over HTTP [Mar 2]:** Simon hit `messaging.subscribeToQuery: can only subscribe to remix database` in mixc worker on Desktop with mixer DB. Gerd: "all"-type live queries
not implemented for HTTP DB access. Known limitation.

**Agent server logging overhaul needed [Mar 6]:** Gerd: agent server logs are "pretty much useless" — no calling app/agent info, no call ID, `message` field truncated. stdlib logs (e.g., failed HTTP
requests) don't surface in server logs. Benedikt: tracing individual calls would be extremely helpful. Mix VM logs go to `mix-vm` target (enabled on cloud agents). Truncated message fix merged:
mix-rs/pull/1050.

## Mar 10–14 Additions

**Publish topics in service agents [Mar 10, 13 replies]:** Simon asked if publish node in a service agent can emit standard topics. Answer: view and session topics work (handled in pure mix). Gerd
shared a pattern using view topic + `co` wait/signal for serialized iteration in the build server — each action step publishes to a view topic to trigger the next. Example:
`remix-dev.remixlabs.com/e/edit/gerd-magic-stuff/Symbols/out_3`.

**FFI for command execution proposal [Mar 11]:** Benedikt shared Notion design doc for FFI to run host commands from Mix. Vijay approved. Gerd flagged biggest challenge: **3 different sandbox
solutions** needed (per platform). Benedikt: MVP = single "allow everything" permission; restrictive sandboxing deferred to later (only needed for third-party programs).
Doc: https://www.notion.so/FFI-for-command-execution-31f1d464528f80029d61c11aefc62b89

**Workspace-specific admin settings [Mar 13, 10 replies]:** Gerd: build server needs two per-workspace settings (debug object upload location, error report URL) — currently hardcoded. Important for
firewall customers (avoid info leakage). Arvind: `_rmx_prefs` is installed on each workspace. Gerd: these are admin settings, not preferences. Didier: synced prefs has a "default" level only admins
can set — `get-prefs` agent returns value to any caller, but regular users can see (not set) it. Notion: https://www.notion.so/Synced-preferences-2871d464528f80ae96dfeff9a738ca67

---

## PRs & Tickets Referenced

| Date   | PR / Issue           | What                                         | Who      |
|--------|----------------------|----------------------------------------------|----------|
| Jan 2  | mix-rs/issues/928    | Blob URL support in HTTP FFI                 | Gerd     |
| Jan 13 | mix-rs/issues/913    | Save unchanged record FFI error              | Gerd     |
| Jan 14 | mix-rs/pull/947      | Initial save FFI fix                         | Fred     |
| Jan 21 | mix-rs/pull/912      | Snowflake packaging                          | Chris    |
| Jan 21 | mix-rs/pull/918      | Index cleaner                                | Fred     |
| Jan 26 | turntable/pull/11660 | Wasm caching gsutil -Z                       | Gerd     |
| Jan 30 | protoquery/pull/2252 | Stdlib save case integration                 | Gerd     |
| Feb 3  | protoquery/pull/2259 | parseQueryString multi-clause                | Gerd     |
| Feb 11 | mix-rs/pull/992      | Revert delete FFI signature                  | Fred     |
| Feb 17 | mix-rs/pull/1001     | Non-ES256 JWT                                | Chris    |
| Feb 17 | mix-rs/pull/948      | Unify desktop/mixer startup                  | Benedikt |
| Feb 25 | mix-rs/pull/1022     | Fix too-many-files-open in deletion          | Fred     |
| Feb 26 | mix-rs/pull/1028     | Deletion/flush patch                         | Fred     |
| Feb 26 | mix-rs/pull/1036     | System-plugins path fix                      | Fred     |
| Feb 27 | amp/pull/2790        | FFI coverage for compress/decompress         | Gerd     |
| Feb 28 | mix-rs/pull/1026     | ARM64 Linux mixer build                      | Chris    |
| Mar 6  | mix-rs/pull/1050     | Fix truncated log messages                   | Benedikt |
| Apr 18 | mix-rs/pull/1086     | Mixcore JS bindings consolidation            | Benedikt |
| Apr 24 | mix-rs/pull/1100     | Blank query deleted records fix              | Fred     |
| Apr 29 | mix-rs/pull/1106     | Query optimization: short-circuit predicates | Fred     |
| Apr 29 | mix-rs/pull/1107     | mixer_wasi: non-trapping float in wasm-opt   | Benedikt |

---

## Channel Patterns

- Deep technical; code snippets, REPL sessions, curl examples inline
- Go/Rust parity recurring theme — FFI behavior must match between amp and mix-rs
- Backward compat taken seriously: add new FFIs, don't change existing signatures
- Quick turnaround: urgent fixes often PRed within hours
- Simon surfaces builder integration issues; Gerd/Chris provide backend solutions

## Mar 14–21 Additions

**OPFS landed [Mar 18, Benedikt]:** `dev.remix.app` now uses OPFS instead of IndexedDB; Desktop uses it for workspace form only.

**`.DS_Store` illegal path [Mar 18, mix-rs#1064]:** Mixer throws error when Finder creates `.DS_Store` near `.mixsrc` files — fix: skip files with illegal paths; turntable#11835 as interim debug aid (
Mar 21).

**Snowflake packaging ready [Mar 19, Chris]:** mix-rs#912 ready once OPFS fixups done. Open: separate repo for packaging config (mostly independent of mix-rs).

## Mar 23 – Apr 3 Additions

**protoquery CI Docker fix [Mar 23, Chris]:** CircleCI stopped building Docker images — API mismatch between docker CLI and Circle-provided daemon. Fix: pin API version. protoquery/pull/2302. Chris:
auto-downgrade was supposed to handle this but didn't; workaround simple enough not worth digging into (docker/cli#2533).

**Mixer CLI local auth gaps [Mar 24, Simon → Gerd/Benedikt/Chris]:** Simon investigated running mixer from command line for CI integration. Three issues found:

1. `--ws-dir /amptest` → "Read-only file system" error — must use relative path `./amptest`. Resolved.
2. `POST /v0/ws` → "No matching routes" — Simon's rcm had a Nov 2025 mixer binary; `create workspace` is v0 only, not v1. Needed rcm update.
3. `mktoken`-issued token rejected as "not a workspace creator" — mixer doesn't default to locally signed tokens like amp does. Fix: `--workspace-creator user/email` flag or `WORKSPACE_CREATORS` env
   var. **No easy CI solve yet.** Simon filed mix-rs#1075; Chris: short-term option is to use Snowflake build (has local token support but lacks S3). Parked.

## Apr 4-11 Additions

**File-not-found returns 410 Gone [Apr 7, Simon → Chris]:** `file.get(null, "/conch.png")` returns "resource has been deleted" when file doesn’t exist. Chris confirmed: mixer maps file-not-found to
HTTP 410 (Gone) — misleading vocabulary. Also: many 410s in Snowflake runtime console are harmless (client probing for optional code files).

## Apr 11–18, 2026 Additions

**Desktop CI release [Apr 17, Chris]:** mix-rs#1093 — skip-if-no-changes prevented beta/release from releasing; fixed. New `release.CHANNEL.BUILD` tag for desktop-only CI; `rmx-remix.js` co-published
with installer.

**File link content-type [Apr 16, Simon/Gerd]:** `file-manager` is admin-only, returns `application/octet-stream`. Public files must use `/files/<path>` API. Simon: builder File node misleads users —
end users can't replicate. Gerd: use an agent.

**Flexible query projection [Apr 17, Fred]:** mix-rs#958 (draft): queries can now project arbitrary values beyond records. New: bitmap serialization return type — collect a predicate's bitmap, reuse
in a later query. Names under review.

## Apr 18–25, 2026 Additions

**Mixcore JS bindings consolidation [Apr 18, Benedikt]:** 5 coordinated PRs: mix-rs/1086, groovebox/481, turntable/11897, flutter-runtime/442, remix.app/213.

**`db_get_a` missing from mixcore [Apr 23, Gerd/Fred]:** `db.getArray` not in mixcore. Workaround: iterate `db.get_one` per ID — slightly more efficient than query (skips AST/bitmap overhead); nulls
for missing IDs need post-processing. No FFI needed.

**Blank query returns deleted records [Apr 23–24, Gerd/Fred]:** mix-rs/998 broke protoquery tests — blank query fallback to all-bitmap wasn't using pre-filtered all (returned deleted records when DB
shared across tests). Fix: mix-rs/1100 (Fred, Apr 24).

## Apr 25–May 2, 2026 Additions

**`mixer_wasi` CI failure fixed [Apr 29, Benedikt]:** `mixer_wasi` failing on `main` — root cause: rust/llvm now produces non-trapping float conversions that must be enabled in `wasm-opt`. Fix:
mix-rs/pull/1107.

**CI "only run on changes" masking bug [Apr 29, Chris]:** If a `main` job fails and no changes land before the next run, the failed job is skipped as a no-op → CI flips red→green without the fix. In
this case an unrelated `protoquery` merge triggered an `M` update that accidentally pulled in the stale `mixer` changes. No systematic fix yet.

**Query optimization sub-PR approved [Apr 29, Fred]:** mix-rs/1106 short-circuits predicate passes to reduce overhead. Approved by Chris — unblocks merge of main optimization PR (mix-rs/1094).
