# #rmx-delivery-lumber Slack Channel — Remix Labs

**Coverage:** Oct 9, 2025 – Apr 18, 2026
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

**Feb 6:** First demo to Lumber. **Bug caught mid-demo:** table showing STATIC copied data, not live Snowflake data. Fix: base catalog component must always use LIVE querying.

**Feb 11:** 90-min session with Lumber engineers. **Risk:** Lumber building their own ETL/calculation layer (Java+SQL); may question need for Snowflake+Remix. Schema: payroll 38 tables, unions 19,
prevailing wages 15, company config 40, compensation 39.

**Feb 18:** Integration UX finalized. One `.remix` per report. Single new Lumber sidebar item → dashboard of available reports → two-panel view. White-label, no Remix branding. Integration spec sent
to Lumber engineers.

### Feb 24-28: Desktop + Facets + Crunch

**Desktop install [Feb 24-25]:** Fresh install 410 error fixed in v0.9920. Remaining: synced apps appear in Projects not Packages (appmeta pending). **Critical:** opening catalog at L0 auto-compiles +
corrupts exec-only app. Mar 1: Arvind checked in latest lumber home/catalog/reports to `iaEj4QYboi` repositories.

**Facet components [Feb 25-26] (Mark):**

1. `snowflake data source setter` — select table
2. `snowflake facet builder` — per-column facets (TEXT + DATE)
3. Copy facet component into screen
4. `snowflake facet → CTE query generator` — accumulates facets into SQL (conditions only applied when set)
5. Execute against Snowflake

**Table component [Feb 26] (Arvind):** Self-contained, configured by SQL statement string. CTE orchestrator overrides with filtered SQL when facets selected; returns default query before any
selection. Forward pattern: each paste-able catalog component has CTE orchestrator built in.

**Scope pushback (Vijay):** Lumber tried to add scope. Vijay told Manish "his guys were smoking something." Conditions: one owner + full scoping exercise, or no new scope. Oleg to own scoping.

### Mar 2–7: Requirements Call + Facet Pattern + Grid Descope

**Integration setup (Chris, Mar 2):** Checking with Reza/India team on who provided integration setup on Lumber's side. Test HTML page approach proposed to confirm integration.

**Facet config pattern decision (Arvind, Mar 4):** Facet selection driven **Data → out** (not by pre-selecting type). Selecting a field yields values + presumed type; may offer 2–3 relevant types (
e.g., menu + pill-button if N is small). Mark building runtime configurator screen → single Catalog item replacing "Generic Facet."

**Grid descope decision (Arvind, Mar 4):** Consensus: offer native table primitives (table/th/tr/td — Tyler adding) instead of finessing TUI Grid webcomp. Enables highly customised rich grids with
remix cards in cells. TUI Grid webcomp work paused for delivery. In-house configurable grid is weeks, not months. Agreed: define "good-enough" scope for current grid, accept throwaway work.

**Requirements call with Oleg + Greg (Reza, Mar 5):** 3:30pm Pacific. Oleg presenting scope of first release (not a Remix demo). Attendees: Vijay, John, Arvind, Reza. Oleg to record and share video.
Chris asked about deliverables timeline and whether to keep beta stable or revert to normal release cadence.

**Apps pushed to workspace (Arvind, Mar 7):** `_rmx_home_desktop`, `catalog`, and `reports` apps all pushed to Lumber workspace. Testing bulk paste.

### Mar 7–14: Table Perf + Grid Decision Reversal + Bugs

**High priority items (Arvind, Mar 9):** Activate in-house table; download as CSV/XLSX; finalise table+facets config flow.

**WIP in-house table perf (Arvind, Mar 11):** 80×40 grid sluggish in Studio/runtime. Key fixes: column-matrix (callables per-column) + flatten-transform-unflatten (~5x). pub/sub retained bug fixed (
turntable/pull/11810).

**DECISION REVERSAL — back to TUI webcomponent (Arvind, Mar 12):** After perf testing, Arvind decided to revert to TUI Grid webcomponent for Lumber delivery (perf reasons). In-house Remix primitive
grid continues as a longer-term goal.

**Lumber customer update (Vijay, Mar 12):** Manish said Lumber will be putting data into tables **this week by Friday (Mar 13)**. Also asked if reporting can be made **self-serve in the medium term
** (lots of reports needed). Vijay + Reza + Arvind held huddle to discuss.

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
