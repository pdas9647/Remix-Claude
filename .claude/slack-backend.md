# #backend Slack Channel — Remix Labs

**Coverage:** Jan 2 – Feb 19, 2026 (channel quiet Dec 20–31, 2025)
**Channel ID:** CHTGC5BGQ
**Purpose:** server, compiler, and database stuff

---

## Key Participants

- **Chris Vermilion** — backend lead; mixer API, auth, deployment, Go DB (amp/eleventwo)
- **Gerd Stolpmann** — compiler, build server, Mix stdlib, mixer caching, protoquery
- **Fred Im** — DB internals (mixquery Rust), Go/Rust save parity, async-ification
- **Simon** — frontend/builder; surfaces query/integration issues from builder perspective
- **Benedikt Becker** — desktop (Tauri/mixcore), OPFS, cloud subscriptions, iOS simulator

---

## Key Decisions & Technical Discussions

### Jan 2–6 — Blob URL support in HTTP FFI

- Web components returning `blob:` URLs failed in API Connect node (`unsupported protocol`)
- **Decision**: separate blob URLs from `http` module entirely; new `binary.fromBlobURL` function
- HTTP FFI cleanup plan agreed:
    - `$http_do_server`: mixcore implementation (desktop/agent)
    - `$http_do_client`: groovebox JS `fetch` (browser, supports blob/CORS)
    - `$http_do`: auto-dispatch (heuristics); falls back to server in desktop, client in browser
- Ticket: mix-rs/issues/928

### Jan 13–28 — DB save FFI overhaul (89-reply thread)

- **Bug**: mix-rs/issues/913 — saving an unchanged record threw FFI errors; no way for caller to handle
- Fred built new save FFIs returning case types instead of bare records:
    - `db.saved(record)` — new row created
    - `db.rev_match_current(record)` — no-op, record matches current revision
    - `db.rev_match_old(record)` — no-op, record matches historical revision (different rev but same content hash)
- **Key design decisions**:
    - New FFIs (`$db_save_ar`, `$db_upsert_ar`) return `result(string, array(saved_record))`
    - Old FFIs preserved for backward compat (transition via new HTTP endpoints)
    - `_rmx_ignored` field kept as string value (not case), e.g. `"rev_match_current"`
    - Non-array save/upsert FFIs deprecated since 2022 — dropped from new API
- PRs merged to mix-rs and amp by Jan 28; Gerd confirmed working in mixer
- Gerd also submitted protoquery/pull/2252 for stdlib integration

### Jan 26–27 — Wasm caching fix (49-reply thread)

- `mixrt.wasm` downloaded every page load (~4s each, twice per L2 open = ~8s wasted)
- **Root cause**: nginx adds gzip on-the-fly → converts strong ETag to weak (`W/`) → browser sends `If-None-Match: W/"..."` → server returns 200 instead of 304
- **Fix**: upload files to GCS pre-compressed with `gsutil cp -Z` (all file types) → nginx passes through GCS strong ETag → browser gets 304
- PR: turntable/pull/11660 (Gerd, approved by Chris)
- Also filed: mix-rs/issues/962 (mixer Cache-Control), mix-rs/issues/963 (other caching issues)
- **Impact**: browser also caches compiled wasm, so caching saves both download AND compilation time

### Jan 30 — Build server notifications + new stdlib functions

- Gerd: build server now supports SSE notifications for build progress; 60s timeout no longer applies
- New `string.contains_ci` function for case-insensitive search

### Feb 3 — Query pipeline / parseQueryString limitation

- `db.parseQueryString` only returned single AST clause; piped queries like `"group(\"_exists\") | count() | ._exists"` failed with "not exactly one clause"
- **Workaround**: break into separate `db.processPipeline` calls, one per clause
- **Fix**: Gerd submitted protoquery/pull/2259 within 2 hours
- Gerd also provided `mix_client` command to connect REPL to mixer DB:
  ```
  mix_client -local-vm -env 'remoteDatabaseServer="http://localhost:2025/v1/ws/local/app"' -env 'token="$TOKEN"' -app queries
  ```

### Feb 3–13 — QB 2.0 query field discovery (31-reply thread)

- Simon needs field metadata for query tile UI: which fields exist, which are expandable refs
- `+engineer` operator expands refs in queries; `keys()` returns field names
- **Problem**: `keys()` didn't indicate field types (especially refs vs. primitives)
- Fred added type inference from index tokens; `"type": "ref"` returned when token conforms to ref format
- **Caveat**: index tokens don't store types natively; inference is heuristic-based
- Simon confirmed new data visible (Feb 13); will integrate into codegen

### Feb 10–11 — Delete FFI backward compat break (27-reply thread)

- `db.delete` was changed to return `db.deleted(record)` case value → broke protoquery test 413 + auto-update flow
- **Decision**: be conservative with FFIs; old `db.delete` signature must not change (breaks existing .remix files)
- Fix: mix-rs/pull/992 (Fred) — reverted delete to old signature; new `delete_ar` added for case-returning version
- Broader lesson: add new FFIs rather than changing signatures of existing ones

### Feb 12 — Mixer v0 API prefix deployment timing

- Simon merged builder PR using `/v0` prefix before it was deployed to agt.remixlabs.com → user prefs errors
- Chris deployed same day; resolved

### Feb 17 — Non-ES256 JWT support in mixer

- mix-rs/pull/1001 (Chris): supports RS256 and other signing algorithms
- **Impact**: Lumber's Descope auth tokens can now be used directly with mixer
- Not related to SAML support (SAML would require much more protocol work)

### Feb 17–20 — Desktop ↔ Mixer dev workflow

- Simon: switching between Desktop and mixer for testing is inefficient
- `mixer serve --ws-dir <desktop-workspaces-dir> --host localhost --port 2025` works but doesn't create "magic URLs" for `file.register` (no R2 storage locally)
- Benedikt: `--base-url` flag exists; PR mix-rs/pull/948 to unify desktop/mixer startup
- Desktop behaves like cloud server except for file registration (no presigned R2 URLs)

### Feb 18 — File upload PUT semantics

- `file.register` returns R2 presigned URL; PUT overwrites by default (standard S3/R2)
- Use `If-None-Match: *` header to prevent overwrite
- Cloudflare R2 behavior; not changeable server-side

### Feb 19 — Builder compile loop + more caching issues

- **Library without code bug**: amp build returns 200 success but no `_rmx-type==file` records → builder assumes Constants compiled → imports fail → builder retries → infinite loop
- Chris: amp-only issue, not worth fixing (moving away from amp); Simon must add builder-side check
- GitHub: amp/issues/2655 (known issue)
- **agt-dev.files.remix.app caching**: weak ETag issue again (same root cause as Jan 26 mixrt.wasm fix)

---

## PRs & Tickets Referenced

| Date   | PR / Issue           | What                                            | Who      |
|--------|----------------------|-------------------------------------------------|----------|
| Jan 2  | mix-rs/issues/928    | Blob URL support in HTTP FFI                    | Gerd     |
| Jan 13 | mix-rs/issues/913    | Save unchanged record throws FFI error          | Gerd     |
| Jan 14 | mix-rs/pull/947      | Initial save FFI fix                            | Fred     |
| Jan 21 | mix-rs/pull/912      | Snowflake packaging                             | Chris    |
| Jan 21 | mix-rs/pull/918      | Index cleaner                                   | Fred     |
| Jan 26 | turntable/pull/11660 | Wasm caching: gsutil -Z for all files           | Gerd     |
| Jan 26 | mix-rs/issues/962    | Mixer Cache-Control                             | Gerd     |
| Jan 26 | mix-rs/issues/963    | Other Cache-Control issues                      | Gerd     |
| Jan 30 | protoquery/pull/2252 | Stdlib save case integration                    | Gerd     |
| Feb 3  | protoquery/pull/2259 | parseQueryString multi-clause support           | Gerd     |
| Feb 11 | mix-rs/pull/992      | Revert delete FFI to old signature              | Fred     |
| Feb 17 | mix-rs/pull/1001     | Non-ES256 JWT support                           | Chris    |
| Feb 17 | mix-rs/pull/948      | Unify desktop/mixer startup                     | Benedikt |
| Feb 19 | amp/pull/2756        | Library-without-code warning (reverted to warn) | Gerd     |

---

## Channel Tone & Patterns

- Deep technical discussions; code snippets, REPL sessions, and curl examples shared inline
- PR review requests tag specific reviewers (Chris, Benedikt, Fred, Gerd)
- Go/Rust parity is a recurring theme — FFI behavior must match between amp (Go) and mix-rs (Rust)
- Backward compatibility is taken seriously; new FFIs added rather than changing existing signatures
- Design decisions often emerge from long threads (50–89 replies); participants iterate in real time
- Simon surfaces integration issues from the builder/frontend perspective; Gerd and Chris provide backend solutions
- Quick turnaround: urgent fixes often submitted as PRs within hours of being reported
- Low formality; direct and technical; no status reports or standups in this channel