# #rmx-delivery-lumber Slack Channel — Remix Labs

**Coverage:** Oct 9, 2025 – Mar 21, 2026
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

## Phase 1 (Oct–Nov 2025): WFJ Benefits — DROPPED

Mobile + web app for WFJ benefits enrollment. **Dropped Oct 25** — WFJ POST API not exposed ("non-starter"), WFJ lacked bandwidth to document business rules.

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

**Jan 21:** Re-kickoff. **Jan 26:** Lumber kickoff call. Shreesha demanding first-week-Feb go-live — "pointlessly aggressive" (Reza).

**Jan 29:** Cortex agent only returns label/value (widget-oriented). Resolution (Feb 6): Padmanabha created `_v2` agents for table widget type.

**Feb 2-3:** Schema + data in Snowflake. Notion pages with Mermaid diagrams, SQL, CSV.

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

**Facet types in scope:** Date range, pill picker (month/quarter/year), month+year carousel, dropdown multi/single picker, keyword search.

**Table component [Feb 26] (Arvind):** Self-contained, configured by SQL statement string. CTE orchestrator overrides with filtered SQL when facets selected; returns default query before any
selection. Forward pattern: each paste-able catalog component has CTE orchestrator built in.

**Crunch mode [Feb 27] — target: Wednesday Mar 4:**

1. **Infra:** Chris + Didier — agent server install + integration
2. **Tools:** Arvind + John — home, catalog, report mgmt, Studio publish (all 4 essentially functional)
3. **Content + QA:** Reza + India team — catalog components, Snowflake schema, report content, QA

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

**High priority items announced (Arvind, Mar 9):**

1. Activate in-house table component (sortable, resizable, searchable, pinned rows/cols, grouped headers, paginated)
2. "Download as file" function — save CSV or XLSX to user filesystem
3. Finalise `table + facets` config flow (or limited version; full UX staged for next delivery)
   — cc'd: Reza, John, Mark

**Gerd + Tyler joined channel (Mar 11):** Gerd Stolpmann and Tyler Lentz added to #rmx-delivery-lumber.

**WIP in-house table component — perf issue (Arvind, Mar 11):** Remix-primitive table component (80 rows, 40 keys) is sluggish in runtime and painful in Studio. Multi-day thread with Arvind, Didier,
Vijay, Gerd, Simon exploring optimisations:

- Callables per-cell vs per-column (Gerd's column-oriented approach faster)
- Vijay's "flatten → transform → unflatten" pattern ~5x faster
- **Root cause identified:** `get state` nodes fire on every state change regardless of field — causes extra propagations; separate screen state fields not independently addressable
- **Bug found + fixed:** `pub/sub retained` flag broken in codegen — Gerd found root cause, Simon opened [turntable/pull/11810](https://github.com/remixlabs/turntable/pull/11810), merged Mar 13 (bug
  since July 2025)
- **Screenshot on save causing long save times** — Didier confirmed: screenshot library (~4 years unmaintained) walking DOM = slow; looking at alternatives. Vijay suggested using Rust renderer.
- **`onload` double-firing on query comp at L2** — Gerd: caused by state node propagation (tile 20 outputs scenegraph twice, 3MB each time); workaround: use pubsub view-scope for `query_status`; fix
  coming
- **L2 freeze on callable-heavy screen** — Gerd: "super slow" not fully frozen; 10-row limit suggested for dev. Still open.
- **`get state` per-field propagation issue** — Simon: can fix for state nodes via codegen, not for screen/app state (dynamic structure). Gerd proposed per-field var cells as workaround. Longer fix
  needed.
- protoquery/issues/2291 filed (Gerd, Mar 13) for retained pub/sub bug

**DECISION REVERSAL — back to TUI webcomponent (Arvind, Mar 12):** After perf testing, Arvind decided to revert to TUI Grid webcomponent for Lumber delivery (perf reasons). In-house Remix primitive
grid continues as a longer-term goal.

- **India team task:** Exercise TUI Grid config object against full TUI documentation; verify all features needed for Lumber's first 5 reports (grouped columns, colspan headers, etc.)
- **India team task:** Document minimum set of "rich" (non-plain-text) cell types — use case, data type, screenshot — so Tyler can add minimal rich renderers inside the webcomponent
- Tyler will work on supporting minimal styling via column config object (no external card injection)

**Lumber customer update (Vijay, Mar 12):** Manish said Lumber will be putting data into tables **this week by Friday (Mar 13)**. Also asked if reporting can be made **self-serve in the medium term
** (lots of reports needed). Vijay + Reza + Arvind held huddle to discuss.

**Release cadence clarification (Chris, Mar 12):** No beta promotion happened week of Mar 7 (no response to his earlier question). Chris's read: main deliverables to Lumber are **content demos** now;
full deployment likely weeks or months out. Plans to promote new beta (harmony/pull/367, beta 2674) on Mar 13.

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
- Issues found by Mark: (1) SQL textarea doesn't auto-populate on screen load — only populates after clicking "Update data"; (2) pagination size selector (25/50/100/500) broken
- Mark's fix plan: inject a default `SELECT * FROM X` from inside the CTE component as the initial SQL

**SQL base query architecture decision (Mark → Vijay/Arvind, Mar 21, approved):** Only the "base query" is shown and editable in the UI. The table component internally wraps it with facets +
pagination to build the full SQL. Flow: `base_sql` in → facets from state → pagination from nested component → full SQL built internally. User never sees/edits the full query. Handles both
natural-language AI queries and custom SQL uniformly. Approved by Vijay + Arvind. Arvind: will add a "view full final query" modal with copy button after Mark ships this.
