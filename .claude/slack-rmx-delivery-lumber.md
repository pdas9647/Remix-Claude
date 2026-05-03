# #rmx-delivery-lumber Slack Channel — Remix Labs

**Coverage:** Oct 9, 2025 – May 2, 2026
**Channel ID:** C09KL3K7P0D

---

## Customer: Lumber

- **What:** Construction payroll/HR SaaS; 100+ customers
- **Partner:** Workforce Junction (WFJ) — employee benefits
- **Workspace:** `iaEj4QYboi` | Snowflake: `https://app.snowflake.com/oxfsvki/tw30727/`
- **Staging:** [https://stage-admin.lumberfi.com/reports/](https://stage-admin.lumberfi.com/reports/)
- **Contacts:** Oleg (product owner), Manish (US, finance), Shreesha (timelines), Anand (payroll eng, India), Lyncean Patel (field/timesheets, India), Trinadh Baranike (reporting, India), Rajesh
  Kannan (reporting lead, first Remix user)
- **Remix team:** Mukund (GTM), Reza (CS lead), Vijay (product), Didier+Chris (platform/infra), Arvind (UX/build), Padmanabha (build), John (platform), Mark (facets), Tyler (webcomponents),
  Gerd+Simon (platform/codegen)
- **Notion:
  ** [Lumber main](https://www.notion.so/Lumber-28c1d464528f801bac72d6c064d30db2) | [Postgres schema](https://www.notion.so/Schema-of-Lumber-Postgres-2f71d464528f8092b0dce634ef55bfb7) | [Job Costing](https://www.notion.so/Job-Costing-Report-2fb1d464528f808b987cce2583b7fb8a)
- **Integration doc:** [https://docs.google.com/document/d/1xHHOfZi94LdDfDbsktjrFc5IvBKJiyguDxXzafDv63s/edit](https://docs.google.com/document/d/1xHHOfZi94LdDfDbsktjrFc5IvBKJiyguDxXzafDv63s/edit)

---

## Phase 2 (Jan 2026+): Analytics Reporting — ACTIVE

Report builder for Lumber teams to configure/present tabular reports over Snowflake data. Self-hosted in Lumber's AWS.

### Architecture

- Lumber multi-tenant Postgres (AWS) → ETL to Snowflake → Remix serves reports
- Tables imported as-is (already denormalized for OLAP); nightly sync
- **Auth:** Gateway agent accepts Lumber JWT, validates via Lumber API (`qa-platform.lumberfi.com/api/users/info`), extracts company ID, calls Snowflake agents internally. Auth notes:
  mix-rs/issues/982
- **Snowflake auth:** RSA service account (no end-user SF accounts). Keys in agents at `lumber_reporting/configurator`
- **Access model V1:** All users within a company see same data (no row-level security)
- **Success criteria:** Lumber's Rajesh/Trinadh can build new reports themselves. Customers building their own is **out of scope V1**.

### Timeline

**Feb 11:** 90-min session with Lumber engineers. Schema: payroll 38 tables, unions 19, prevailing wages 15, company config 40, compensation 39.

**Feb 18:** Integration UX finalized. One `.remix` per report. Single new Lumber sidebar item → dashboard of available reports → two-panel view. White-label, no Remix branding. Integration spec sent
to Lumber engineers.

### Feb 24–28: Facets + Table Architecture

Facet pipeline: Snowflake setter → facet builder (TEXT+DATE) → CTE query generator. Table: self-contained SQL-configured. Scope locked by Vijay; Oleg owns scoping.

### Mar 2–7: Facet Pattern + Grid Descope

Facet: data-driven out-params (not pre-typed); single Catalog item replaces “Generic Facet.” Grid: native primitives (table/th/tr/td) over TUI Grid webcomp — weeks not months. Requirements call Mar 5
with Oleg/Greg.

### Mar 7–14: Table Perf + Oscillation

In-house table sluggish (80×40); reverted to TUI Mar 12, reversed back Mar 16 after column matrix (callables per-column) → sub-2s incl. Snowflake. turntable/11810 pub/sub fix.

### Mar 14–21: In-House Table Back On + Table/Facet Design Sprint

**DECISION REVERSAL AGAIN — back to in-house Remix primitives (Arvind, Mar 16):** Gerd's column matrix transpose pattern (callables per-column not per-cell) brought the in-house table to sub-2s load
for 80×40 grid incl. Snowflake fetch; sort repaints ~1s. Reverses Mar 12 TUI Grid decision. Internal state (query_status, sort column) migrated to view-scope pub/sub; callable dispatch map moved to
direct binding; table config unified as `{ db, sql, columns }`.

**Snowflake schema SQL shared by Reza (Mar 18):** Trinadh Baranike sent two SQL files — `payroll-to-journalreport-to-reporting-tbls.sql` and `payroll-proportionated-comp-code-table.sql` — as the
starting point for building the 5 Lumber reports.

**Table component interface redesign (Arvind + Mark, Mar 18–19, 17-reply thread):** Arvind overhauled the component to be fully plug-and-play:

- Single `data` in_param: accepts one object with `{ db, sql, columns }` — no out_params, no event bindings
- Goal: paste → hook top-right into layout → live
- Pushed to repo (Table Builder project in Lumber workspace); Mark reviewing

**SQL base query architecture decision (Mark → Vijay/Arvind, Mar 21, approved):** Only the "base query" is shown and editable in the UI. The table component internally wraps it with facets +
pagination to build the full SQL. Flow: `base_sql` in → facets from state → pagination from nested component → full SQL built internally. User never sees/edits the full query. Handles both
natural-language AI queries and custom SQL uniformly. Approved by Vijay + Arvind. Arvind: will add a "view full final query" modal with copy button after Mark ships this.

### Mar 23–Apr 3: Charts + Demo Crisis + Report Builder Redesign

**TUI grid sort-by-column event [Mar 23, Arvind → Tyler]:** Column header click must fire a sort event for Snowflake SQL sort order to respond. cc: Mark, Reza.

**Pre-demo crisis [Mar 27, Reza, 3am]:** 5 open issues before Friday Lumber demo → moved to Monday:

1. CTE must apply to base table (not final dataset) for facets
2. Table shows source-table columns, not SQL GROUP BY result columns
3. Views not supported as data sources (tables only)
4. Demo can't be driven from Desktop catalog (publish/cache bug)
5. Numeric columns show excess decimals despite SQL rounding

**Mark pushed repo fixes [Mar 27]:** CTE fix, column/filtering improvements, view-datasource support. NOT published to catalog (cache issue).

**Scope: stacked-column chart only for Monday [Mar 27, Arvind]:** Job costing report only chart needed. John WIP: column+facet, h-bar+facet, scatter+facet, bubble.

**TUI built-in download [Mar 30, Arvind]:** TUI Grid has CSV/XLSX; charts have PDF/PNG. May be sufficient.

**Report builder redesign [Apr 1–3, Reza]:** Clean node types (data / filters / display) piped independently. JSON is isomorphic to Remix node graph → AI can one-shot an entire report definition.
Tested with Claude Code against Snowflake. Notion: https://www.notion.so/Report-Builder-3351d464528f800d8b5ce12f6517c93f |
Assets: https://www.notion.so/Claude-Code-generated-assets-3371d464528f809c9c5ed929c8709ecd

### Apr 4–11: Composable Architecture Delivery Sprint

**Status (Vijay/Arvind, Apr 6):** Vijay asked if deliverable ready for Lumber. Arvind: not today; Notion checklist to follow.

**Embedding in Lumber portal (Didier/Arvind, Apr 6–8):** Anon test .remix published in lumber workspace. Auth TBD: how Lumber user creds map to Remix token. Decision: webcomp for embedding into
existing page (one JS import); `rmx-app-record.js` if owning full page.

**Job costing defects (Reza, Apr 6–7):** Columns don’t refresh on SQL change without restart. Confirmed: job costing uses two views (`V_JOB_COSTING_TABLE` by cost code, `V_JOB_COSTING_CHART` by
date) — same CTE facets, separate Data Fetch nodes required. UX ask: column picker reorder.

**Data Fetch + Facets demo (Mark, Apr 9):** Screen showing independently configured Data Fetch + Facet per data source. `table_builder` + `facet_builder` repos updated — pull latest.

**TUI chart auto-sizing + regression (Apr 8–9):** `auto` w/h fix deployed; parent needs explicit height or chart grows unboundedly. Grid: `bodyHeight: "fitToParent"`. Chart then regressed (cache) →
Tyler published new `/v3`. **Action required: sync library assets + republish apps.**

**Report build demo (Mark, Apr 10):** Demo video: report built from scratch with multiple tables, each with independent data source/facets/pagination/sorting.

### Apr 13–18: Auth Design + Delivery Push

**Delivery push (Apr 14–16):** Dev→prod promoted; dedicated Lumber Desktop release channel created. Windows excluded (L2 crash). Mobile beta 2788 targeted.

**Lumber auth design (Apr 15, Chris):** Runtime works (mixer trusts Descope tokens). Studio needs OAuth/OIDC — Lumber must set up Descope app with `auth.remixlabs.com` as callback, no Remix code
changes. Self-contained mixer auth alternative requires code changes.

**Report flow v7 (Apr 17, Mark):** Paginated/all data copy variants; recursive Snowflake partition fetch; initial sort+facet values from UI; agent aliases; next event on Data Fetch; date format fixed.

**Data formatting (Apr 17, Arka):** Dates as day-numbers — add `DATE_OUTPUT_FORMAT: YYYY-MM-DD` param. Decimal drift — `::NUMBER(10,2)` cast; SQL prompt needs update.

**Embed (Apr 16–18, Didier/Chris/Arvind):** Google Doc drafted (pending: Desktop URL, rmx-remix.js URL, .remix params). Plan: co-publish rmx-remix.js with each desktop installer. Embed needs anon
agents — Arvind making workspace agents anon. Final integration passes Lumber portal token.

### Apr 18–25: QA Sprint + Oleg/Greg Demo Prep

**FLOAT → NUMBER(p,s) precision fix [Apr 20, Mark]:** Snowflake FLOAT columns return imprecise decimals (27.79 → 27.789999...). Root cause: FLOAT is approximate. Fix: (1) alter column type to NUMBER(
p,s), or (2) view with CAST(ROUND(col, 2) AS NUMBER(12,2)). Mark created `V_WORKER_COMPENSATION_NUMS` view on `V_WORKER_COMPENSATION_NEW` as demo.

**Channel switch to beta [Apr 20, Arvind]:** India team moved to beta — release v0.10557.0 blocked (Padmanabha: red toast on project creation; can’t create/import screen).

**Column order UX [Apr 21, Arvind]:** AI config handles complex header grouping, but drag/sort for column order also needed. Options: (1) tree editor respecting group boundaries; (2) TUI Grid native
drag + emit events to update config/variant. Open.

**Table config + QA milestone [Apr 22, Arvind]:** Pushed AI-based table config + column reorder to catalog. Asked team: have all 4 table-only reports been built and published on lumber-qa?

**Clean install test [Apr 22, Arvind]:** Requested full Desktop wipe + reinstall from scratch (as Lumber user would) + report build, then report back.

**DATA FETCH reload bug [Apr 23, Padmanabha]:** Right-click → Reload on DATA FETCH AND QUERIES screen errors: `RmxBase.attemptInit Missing: screenName`. cc: Vijay.

**Oleg/Greg demo + handoff target [Apr 23–24, Arvind]:** 9:30am PAC call Apr 24 demoing current state-of-the-art. Target: handoff call with Lumber tech team Mon Apr 27; delivery must reach lumber-qa
beforehand. Raised: channel switcher needed for Lumber users?

**Pagination bug in query_builder [Apr 24, Arvind]:** Pager visible with 12-row SQL result; clicking pages does nothing. Not blocking delivery.

### Apr 25–May 2: Auth + Embed Delivery

**Lumber auth token exchange [Apr 28, Didier/Arvind]:** Flow confirmed: `auth.remixlabs.com/a/x/auth/lumber_qa/token?token=<projectID>:<refreshJWT>` exchanges Lumber JWT for Remix token. Stage-admin
project ID: `P2Zne0vibP6VccnwmuPXAfOl1jz0`. Didier’s JS wrapper working May 2.

**rmx-remix unified login [May 1, Didier]:** Webcomp now handles auth itself — no more anon agents needed for embed. Sign-in broken on release (component mismatch); fixed in `0.11350.0` (promoted May
1). `rmx-remix.js`: `https://agt.files.remix.app/remix/desktop-releases/files/11350/js/rmx-remix.js`.

**Decision: release channel [May 2, Didier]:** Lumber delivery targets `release` channel. Vijay: Lumber eng team to set up embed; Reza handling Sunday night.

**Client status [Mukund]:** (1) reports built, (2) auth wired, (3) embed integration, (4) May 4 handoff call with Lumber tech team. Arvind added how-to doc to Desktop home app.

**Catalog UX issues [Padmanabha]:** Notion: https://www.notion.so/3501d464528f8099b21cf64bfc5b71c2
