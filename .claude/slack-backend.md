# #backend Slack Channel — Remix Labs

**Coverage:** Jan 2 – Feb 28, 2026
**Channel ID:** CHTGC5BGQ
**Purpose:** server, compiler, and database stuff

**Key participants:** Chris (backend lead, mixer/auth/deploy), Gerd (compiler/stdlib/caching), Fred (DB internals, mixquery Rust), Simon (builder integration), Benedikt (desktop/Tauri/OPFS)

---

## Key Decisions & Technical Discussions

### Blob URL support in HTTP FFI [Jan 2-6]
Web components returning `blob:` URLs failed in API Connect. Decision: new `binary.fromBlobURL` — separate from `http` module. HTTP FFI dispatch: `$http_do_server` (mixcore), `$http_do_client` (JS fetch), `$http_do` (auto-dispatch). Ticket: mix-rs/issues/928.

### DB save FFI overhaul [Jan 13-28, 89 replies]
Bug mix-rs/issues/913: saving unchanged record threw FFI errors. Fred built new save FFIs returning case types: `db.saved(record)`, `db.rev_match_current(record)`, `db.rev_match_old(record)`. New FFIs: `$db_save_ar`, `$db_upsert_ar`. Old FFIs preserved for backward compat. Non-array save/upsert deprecated since 2022. Merged to mix-rs + amp; stdlib: protoquery/pull/2252.

### Wasm caching fix [Jan 26-27, 49 replies]
`mixrt.wasm` downloaded every page load (~8s for two loads per L2 open). Root cause: nginx gzip converts strong ETag to weak → no 304. Fix: upload pre-compressed with `gsutil cp -Z` → strong ETag preserved. PR: turntable/pull/11660. Also: mix-rs/issues/962 (mixer Cache-Control), mix-rs/issues/963.

### Build server SSE + string.contains_ci [Jan 30]
Build server now supports SSE notifications for progress (60s timeout no longer applies). New `string.contains_ci` for case-insensitive search.

### parseQueryString multi-clause [Feb 3]
Piped queries failed ("not exactly one clause"). Fix: protoquery/pull/2259 (Gerd, 2hr turnaround). REPL connect command: `mix_client -local-vm -env 'remoteDatabaseServer="http://localhost:2025/v1/ws/local/app"' -env 'token="$TOKEN"' -app queries`.

### QB 2.0 field discovery [Feb 3-13, 31 replies]
Simon needs field metadata for query tile UI (which fields are refs). Fred added type inference from index tokens — `"type": "ref"` when token conforms to ref format. Heuristic-based (tokens don't store types natively). Simon confirmed working Feb 13.

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

**Deletion / too-many-files-open [Feb 25]:** `join_all` in `find_recs_for_deletion` opened too many file handles. Fix: sequential `for` loop; `get_by_id` now IO-free. Also fixed `exact_match` over-optimization in `get_by_ref`. mix-rs/pull/1022, merged.

**High-priority flush patch [Feb 26]:** Critical bug in deletion/flush pipeline (via Benedikt). mix-rs/pull/1028, merged same day.

**`$binary_compress` FFI missing in CI [Feb 27]:** Gerd's compress/decompress PRs needed amp registration. Fix: amp/pull/2790, merged. Broader: need "builtin-with-fallback" FFI category between `builtin` and `FFI`.

**Amp dev blocked [Feb 27]:** System plugins path bug (double `path.Join`). Fix: mix-rs/pull/1036; resolved when CI completed.

**.DS_Store corrupts Desktop DB [Feb 26]:** Finder creates `.DS_Store` in workspace folder → API hosed. One-off cleanup: `find .../workspaces/local/ -name .DS_Store -print0 | xargs -0 rm`.

**JSON files served as binary [Feb 27]:** `.info` extension unrecognized → binary content-type. Open: mix-rs/issues/1034.

**ARM64 Linux mixer build [Feb 28]:** Lumber requested. mix-rs/pull/1026, merged.

**Desktop unresponsive: make-agent VM [Feb 28]:** VM not closed (Simon's side); worker error passback under investigation.

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

---

## Channel Patterns

- Deep technical; code snippets, REPL sessions, curl examples inline
- Go/Rust parity recurring theme — FFI behavior must match between amp and mix-rs
- Backward compat taken seriously: add new FFIs, don't change existing signatures
- Quick turnaround: urgent fixes often PRed within hours
- Simon surfaces builder integration issues; Gerd/Chris provide backend solutions