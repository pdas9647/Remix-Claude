# #backend Slack Channel — Remix Labs

**Coverage:** Jan 2 – Mar 21, 2026
**Channel ID:** CHTGC5BGQ
**Purpose:** server, compiler, and database stuff

**Key participants:** Chris (backend lead, mixer/auth/deploy), Gerd (compiler/stdlib/caching), Fred (DB internals, mixquery Rust), Simon (builder integration), Benedikt (desktop/Tauri/OPFS)

---

## Key Decisions & Technical Discussions

### Blob URL support in HTTP FFI [Jan 2-6]

Web components returning `blob:` URLs failed in API Connect. Decision: new `binary.fromBlobURL` — separate from `http` module. HTTP FFI dispatch: `$http_do_server` (mixcore), `$http_do_client` (JS
fetch), `$http_do` (auto-dispatch). Ticket: mix-rs/issues/928.

### DB save FFI overhaul [Jan 13-28, 89 replies]

Bug mix-rs/issues/913: saving unchanged record threw FFI errors. Fred built new save FFIs returning case types: `db.saved(record)`, `db.rev_match_current(record)`, `db.rev_match_old(record)`. New
FFIs: `$db_save_ar`, `$db_upsert_ar`. Old FFIs preserved for backward compat. Non-array save/upsert deprecated since 2022. Merged to mix-rs + amp; stdlib: protoquery/pull/2252.

### Wasm caching fix [Jan 26-27, 49 replies]

`mixrt.wasm` downloaded every page load (~8s for two loads per L2 open). Root cause: nginx gzip converts strong ETag to weak → no 304. Fix: upload pre-compressed with `gsutil cp -Z` → strong ETag
preserved. PR: turntable/pull/11660. Also: mix-rs/issues/962 (mixer Cache-Control), mix-rs/issues/963.

### Build server SSE + string.contains_ci [Jan 30]

Build server now supports SSE notifications for progress (60s timeout no longer applies). New `string.contains_ci` for case-insensitive search.

### parseQueryString multi-clause [Feb 3]

Piped queries failed ("not exactly one clause"). Fix: protoquery/pull/2259 (Gerd, 2hr turnaround). REPL connect command:
`mix_client -local-vm -env 'remoteDatabaseServer="http://localhost:2025/v1/ws/local/app"' -env 'token="$TOKEN"' -app queries`.

### QB 2.0 field discovery [Feb 3-13, 31 replies]

Simon needs field metadata for query tile UI (which fields are refs). Fred added type inference from index tokens — `"type": "ref"` when token conforms to ref format. Heuristic-based (tokens don't
store types natively). Simon confirmed working Feb 13.

### Delete FFI backward compat break [Feb 10-11, 27 replies]

`db.delete` changed to return case value → broke tests + auto-update. **Lesson:** never change existing FFI signatures; add new ones (`delete_ar`). Fix: mix-rs/pull/992.

### Mixer v0 API prefix [Feb 12]

Simon merged builder PR using `/v0` before agt deployment → prefs errors. Chris deployed same day; resolved.

### Non-ES256 JWT [Feb 17]

mix-rs/pull/1001 (Chris): supports RS256 etc. Lumber's Descope tokens now work with mixer. Not related to SAML.

### Desktop ↔ Mixer dev workflow [Feb 17-20]

`mixer serve --ws-dir <desktop-dir> --host localhost --port 2025` works but no R2 presigned URLs for `file.register`. `--base-url` flag exists; mix-rs/pull/948 unifies startup.

### File upload PUT semantics [Feb 18]

`file.register` returns R2 presigned URL; PUT overwrites by default (S3/R2 standard). Use `If-None-Match: *` to prevent overwrite. Not changeable server-side.

### Builder compile loop [Feb 19]

Amp build returns 200 but no `_rmx-type==file` records → builder retries forever. Chris: amp-only, not worth fixing (moving away from amp); Simon must add builder-side check. amp/issues/2655.

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

**Desktop unresponsive: make-agent VM [Feb 28]:** VM not closed (Simon's side); worker error passback under investigation.

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

| Date   | PR / Issue           | What                                 | Who      |
|--------|----------------------|--------------------------------------|----------|
| Jan 2  | mix-rs/issues/928    | Blob URL support in HTTP FFI         | Gerd     |
| Jan 13 | mix-rs/issues/913    | Save unchanged record FFI error      | Gerd     |
| Jan 14 | mix-rs/pull/947      | Initial save FFI fix                 | Fred     |
| Jan 21 | mix-rs/pull/912      | Snowflake packaging                  | Chris    |
| Jan 21 | mix-rs/pull/918      | Index cleaner                        | Fred     |
| Jan 26 | turntable/pull/11660 | Wasm caching gsutil -Z               | Gerd     |
| Jan 30 | protoquery/pull/2252 | Stdlib save case integration         | Gerd     |
| Feb 3  | protoquery/pull/2259 | parseQueryString multi-clause        | Gerd     |
| Feb 11 | mix-rs/pull/992      | Revert delete FFI signature          | Fred     |
| Feb 17 | mix-rs/pull/1001     | Non-ES256 JWT                        | Chris    |
| Feb 17 | mix-rs/pull/948      | Unify desktop/mixer startup          | Benedikt |
| Feb 25 | mix-rs/pull/1022     | Fix too-many-files-open in deletion  | Fred     |
| Feb 26 | mix-rs/pull/1028     | Deletion/flush patch                 | Fred     |
| Feb 26 | mix-rs/pull/1036     | System-plugins path fix              | Fred     |
| Feb 27 | amp/pull/2790        | FFI coverage for compress/decompress | Gerd     |
| Feb 28 | mix-rs/pull/1026     | ARM64 Linux mixer build              | Chris    |
| Mar 6  | mix-rs/pull/1050     | Fix truncated log messages           | Benedikt |

---

## Channel Patterns

- Deep technical; code snippets, REPL sessions, curl examples inline
- Go/Rust parity recurring theme — FFI behavior must match between amp and mix-rs
- Backward compat taken seriously: add new FFIs, don't change existing signatures
- Quick turnaround: urgent fixes often PRed within hours
- Simon surfaces builder integration issues; Gerd/Chris provide backend solutions

## Mar 14–21 Additions

**OPFS landed [Mar 18, Benedikt]:** Origin Private File System shipped. New (sane) file system backend for the browser. On **Desktop**: used only for the workspace form. On **dev.remix.app**: replaces
IndexedDB as the whole local storage. Celebrated with 🎉. Arvind asked when it would reach desktop (dev) — answer: already used for workspace form.

**`.DS_Store` illegal path bug [Mar 18, Simon → Gerd]:** Mixer throws `"invalid path '/.DS_Store': illegal path segment .DS_Store"` when Finder creates `.DS_Store` near `.mixsrc` files — breaks the
builder entirely. Same root cause as Feb 26. Gerd: fix is to skip files with illegal paths during listing. Issue: mix-rs#1064. Simon committed turntable#11835 as interim debugging aid (Mar 21).

**Snowflake packaging PR ready to merge [Mar 19, Chris → Benedikt]:** mix-rs#912 (Snowflake packaging) is ready for review/merge once OPFS fixups are done. Open question: should the packaging config
get its own repo, since it's mostly independent of mix-rs and just references a particular image build.
