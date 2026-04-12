# Standup Notes — Apr 2026

**Coverage:** Apr 1–10, 2026
**Format:** Condensed bullets per standup. Google Doc link on each date for on-demand full transcript fetch.
**See also:** [standup-notes-mar-2026.md](./standup-notes-mar-2026.md) — Mar 2–31

---

## 2026-04-10 — [Google Doc](https://docs.google.com/document/d/1dCjakABS4veInsEXlk2GPUAmHWhHhKs2EFeaxlxcR2c)

- **Desktop builder performance (critical)**: Degrades on saves + screen adds; recurring data corruption + duplicate nodes on save. "Unfreeze blocked save tiles" flag considered dangerous. Decision:
  all builders must report desktop issues in Bug Bash — no working around them.
- **Database bloat (root cause)**: Workspace DB found to be 8.35 GB. Infinite version history kept for every save of every node. GC feature implemented but not enabled. Decision: Fred Im to enable
  GC + restore DB compaction endpoint for automated use. Benedikt to add regular DB size logging.
- **Lumber delivery**: Auto-sizing issue resolved. Arvind assembled table with updated tooling. One report + embed discussion with client. Assets to publish to Lumber catalog/library before delivery.
  Walk-through with Reinard Monday; deliver to Greg Tuesday.
- **Starlight workspaces**: Two set up (one on dev); Twilio connected for basic messages. Two active bugs: (1) "member of query" bug causing slow queries (Fred reviewing), (2) raw agent URL needed for
  Twilio webhook verification (Chris to address).
- **Starlight branding**: Figma brand file provided. Plan: AI pipeline (Claude) digests Figma → structured theme assets (colors, fonts, sizes, SVGs). Arvind to receive knowledge transfer
  post-delivery.
- **New beta deployment**: Chris to deploy new beta today — includes form-encoded request support fix.
- **Theming/paste improvements**: Revisit paste format to allow catalog address pasting (resolves components at paste time). Figma → theme conversion AI spike planned by Vijay + Arvind.

---

## 2026-04-09 — [Google Doc](https://docs.google.com/document/d/1lme41uy3-XM6nbBvLQbjQGh7Y3xCGex2h7Gf7zWX5vw)

- **John workload split**: (1) Starlight project initiation, (2) Snowflake generalized store submission solution using Lumber job costing report + facets as template.
- **Chart web component `auto` dimensions**: Stopped working with `height`/`width` = `auto`. Known issue with default container styles. Recommended version = `remix_labs`. John + Tyler to sync on
  correct version to use.
- **RCS messaging tool update (Wilber)**: Users can configure content + create messaging templates via content instance form. Variable injection (e.g., `first_name`) available at runtime. Uses
  Twilio-defined templates.
- **RCS deployment cleanup (Wilber)**: Splitting records + service agents from single Twilio project → cleaner arch, easier future Twilio migration.
- **Export app broken**: App export currently broken. Workaround: use Gird's plugin to export → upload .remix file via cloud files view. Drag-and-drop upload plugin restored. Chris: export error cause
  tracked down; uploads now using no-cache header.
- **Raw agent URL (Wilber/Chris)**: Needs raw URL + path/query params for Twilio webhook signature verification. Chris: will ensure this is addressed; not difficult to implement.
- **System project plugins blocker (Mark)**: Latest published plugin not showing in desktop builder (packaged apps hidden). Long-term fix: enrich app manifest to expose packaged app entry points as
  plugins (Didier). Short-term workaround: use Gird's plugin to export Table Builder + Facet Builder → upload via cloud files.

---

## 2026-04-08 — [Google Doc](https://docs.google.com/document/d/1lqJPooTmDobYWuXovXzGYiDylTJWv1GvzepqzEwio_4)

- **India team RCS website (Sirshendu + Padmanabha)**: Building RCS demo website/landing page with Customer 360 Snowflake demo in a polished UI. Contact form for RCS demo signups. Shows restaurant,
  e-commerce, B2B experiences.
- **Collaboration workflow**: Team migrating all projects to desktop via repository mechanism during Mukund's leave. Currently in India beta for easier Arpita collaboration.
- **Snowflake SPCS load time**: Landing page loads slowly — runtime downloads all code one file at a time → many network requests (latency is network time to US West Coast, not warehouse size). Root
  cause tied to session agent downloading everything for any possible screen push. Fix assigned to Gerd; linked to session agent optimization.
- **Vijay feedback**: Users should explore demo freely; contact form blocks exploration. Lumber card rendering learnings applicable to SPCS iteration.

---

## 2026-04-07 — [Google Doc](https://docs.google.com/document/d/1DDen_IDQNZkAa5FSFSZEP6toMfAbHP8PwHbRtmOHq1w)

- **Monorepo decision deferred**: Kurt absent + opposed (reason: mixed PRs in list). Final decision waits for his return. Chris + Benedikt in favor; Kurt against; Tyler cautious.
- **Desktop component relocation**: Desktop is end-of-chain but lives in mix-rs (beginning-of-chain) — wrong placement. Options: move to turntable or own repo. Turntable would restructure into
  distinct folders: builder, runtime, desktop, extension, web components.
- **RMX2E grid server-side sorting PoC (Tyler)**: Data source URL drives server-side sort + pagination. Benefits: multi-sort, proper loading state, no table flash, no data through HTML attributes (
  less JSON in HTML → better performance). Agent must handle page/sort-column/sort-direction params.
- **Simon Hampton updates**: (1) "View stack embed" node codegen — idiomatic screen-in-screen within same .remix file; (2) L1 macro for "rebuild and make" — auto-runs after repo clone for seamless
  onboarding; (3) L0 macro for project creation — repo creates own app + drives user into it; (4) File node cloud integration (cloud subscribe + cloud agent support).
- **Web component fixes (Didier + Tyler)**: Default fonts not loading in web components — fixed. Toast breakage when embedding RMX app inside another without ShadowDOM — fixed. Notion page created
  with minimal embedding instructions.
- **Builder data extraction (Didier)**: Can now extract data from right side (was left only); simplifies data type discovery + binding creation.

---

## 2026-04-06 — [Google Doc](https://docs.google.com/document/d/19gLS2jtiqr4EvA6-pBb0h8pwxg5k8De0sQqu5MFrTxs)

- **Mix core JS cleanup (Benedikt)**: Consolidating mix core JS bindings from multiple repos into mix core repo. Fix: dynamically link both JS + WASM files (previously JS statically linked + WASM
  dynamic → version clashes). Dynamic loading enables browser caching + pre-compilation of WASM.
- **Asset hosting**: Static assets (web components, WASM) on `remix.app` for dev/beta. Behind-firewall deployments must host own copy. Snowflake sealed deployments: assets bundled at build time.
- **Release model design**: Rust-based 3-branch flow proposed — `main` (nightly), `beta` (long-lived, cherry-picked bug fixes), `release` (promoted from beta at end of cycle). Hot fixes target `beta`
  branch.
- **Monorepo discussion (Chris + Benedikt)**: Multi-repo makes hot fixes painful — single bug fix requires 4 PRs across mix core, turntable, protoquery, AMP. Monorepo simplifies branch protection +
  release flow. Write-up prepared for next day's group decision meeting.
- **RCM proposal**: Make propagating-change flow first-class — push to upstream branch → auto-updates downstream PRs, merges when checks pass.
- **Fred Im**: Fixed inquiry reordering bug (internal aggregation logic). Rewriting query tools (rebuild/check data files, reindex, validate records).

---

## 2026-04-03 — [Google Doc](https://docs.google.com/document/d/1C7KPRTLPpbeXdbjBcrZJMW2SsgN5XljJfSykFbueaoc)

- **Sales doubled → offsite back on agenda**: New engagement revenue ~$800/month (highest in ~5 cycles, comparable to Lumber). John: revenue less important than proving the RCS solution for future
  sales.
- **RCS / federal credit unions (30-day trial)**: RCS solution in 30-day trial with federal credit unions for marketing campaign conversion, outreach, and engagement. Demos complete; setting up
  experience for client. Team positioned to become default interface for these interactions.
- **Intercom integration plan**: Allow clients to send RCS from Intercom. Short-term: client uses Remix conversation view. Vijay: significant future revenue via AI-automated RCS experiences.
- **RCS submission to Twilio complete**: Submitted; Sumal credited for helping with required details. Campaign + landing page + value story being built for RCS marketing.
- **Snowflake marketplace submission**: Very close (Chris: platform in good state). Remaining: SPCS environment quirks, packaging demo data, onboarding flow design. Streamlit app discussed as
  front-end for permissions/data setup — not blocking submission.
- **Sales pipeline**:
    - Starlight USD (SI with Snowflake experience): expected to close soon.
    - Zendesk partner discussions progressing; next meeting next week.
    - Meeting with lead of Google's RCS team secured in ~10–12 days.
- **Priorities (Vijay)**: Push hard on pipeline. Urgent: resolve Lumber desktop delivery ASAP. Demo of Vijay's work (Arvind to present) deferred to Monday.

---

## 2026-04-02 — [Google Doc](https://docs.google.com/document/d/1Zwo02MIMhrldH_TT8-oA392tEOvG1vYN7qLxaBAliQo)

- **RCS flow status (Wilber Claudio)**: Service agent app for calendar picking mostly complete; scheduling flow nearly done. John to update cards post-standup.
- **Twilio agent tooling**: New tooling to accelerate Twilio agent setup — covers all Remix edge cases, auto-deploys agents + creates aliases from Twilio info. Some manual Twilio console config still
  required.
- **System plugin versioning decision**: Cache-control header conflict between dev/beta and prod Cloudflare environments — no single file upload solution covers all environments. Decision: version
  system plugins (separate versions for dev, beta, prod). Two tasks: (1) short-term fix, (2) longer-term move system plugin registration to preferences for workspace admins.
- **TUI charting components (John Dismore)**: New components: bubble scatter, column stack, column, bar — all with numeric filter. No loading state yet (all refresh at once). All in library; Lumber
  library asset updated.
- **Chart architecture**: Config-driven — base SQL uses standard field names; chart config maps X/Y/label. Three parts: aggregator (base query + facets), SQL runner, charting component (reshapes
  data). Max height calculated client-side; stack sum calculated by chart.
- **Lumber (Mark Loevenstein)**: Bug fixing + table cleanup; supporting various facet types.
- **Post-standup lumber huddle**: Reza, Mark, John, Vijay, Arvind to review Reza's lumber proposal + timeline + task assignments.
- **RCS factory enablement**: Vijay: focus on completing "first of a kind" experience before enabling factory flow. John: RCS component is independent of experience — only sends message + link to
  agnostic .remix file.

---

## 2026-04-01 — [Google Doc](https://docs.google.com/document/d/1pnKfuBBbLreaCMxPwlSKw3aCo9c-QWxVe4G69Z4nzyw)

- **Lumber (India team)**: No new updates; Padmanabha Das confirmed nothing to report.
- **Lumber SQL/CTE rethink (Reza Mohsin)**: Current approach needs rethinking for better report functionality. Reza to work through examples first, then bring in Vijay, John, Mark.
- **SPCS Snowflake Snowpark widget preview flow (Padmanabha Das)**: Goal — user selects table, enters human prompt → Cortex generates + runs SQL from table metadata → widget preview in app. Blocked:
  cannot obtain table DDL in consumer-side application. Chris investigating workaround. Fallback: user provides DDL manually.
- **JSX/embedding follow-up (Vijay)**: Smaller group meeting with Simon, Didier, John on JSX-side work. John to schedule.
- **Personal**: Nivesh Mishra getting married Apr 25; relocating to Bangalore.
