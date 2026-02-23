# #rmx-delivery-lumber Slack Channel — Remix Labs

**Coverage:** Oct 9, 2025 – Feb 20, 2026
**Channel ID:** C09KL3K7P0D

---

## Customer: Lumber

- **What it is:** Construction payroll/HR SaaS; 100+ customers
- **Partner:** Workforce Junction (WFJ) — employee benefits
- **Workspace:** `iaEj4QYboi`
- **Staging reports:** https://stage-admin.lumberfi.com/reports/
- **Lumber contacts:**
    - Oleg: overall product owner (US)
    - Manish: US-based (financial constraints)
    - Shreesha: aggressive on timelines
    - Anand (Hariharan): payroll engineering owner (India)
    - Lyncean (LINSAN) Patel: field/timesheets owner (India)
    - Trinadh Baranike: founding engineer, reporting (India)
    - Rajesh Kannan: reporting lead, first Remix user (India)
- **Remix team:** Mukund (GTM), Reza (CS lead), Vijay (product), Didier + Chris (platform/infra), Arvind (UX/build), Padmanabha (build), John (platform)
- **Notion pages:**
    - Lumber main: https://www.notion.so/Lumber-28c1d464528f801bac72d6c064d30db2
    - Postgres schema: https://www.notion.so/Schema-of-Lumber-Postgres-2f71d464528f8092b0dce634ef55bfb7
    - Job Costing report: https://www.notion.so/Job-Costing-Report-2fb1d464528f808b987cce2583b7fb8a

---

## Project History: Two Phases

### Phase 1 (Oct–Nov 2025): WFJ Benefits — DROPPED

- Original scope: mobile + web app for Workforce Junction benefits
    - Mobile app: contractors view/enroll in benefits plans
    - Web app: admin flow for benefits administrators
    - Mobile Embed SDK in Lumber's app
    - Self-host Remix in Lumber's private AWS cloud
- WFJ sent enrollment C# code and DB diagram; Figma mocks shared
- **Blockers:** WFJ POST side not exposed via API (Arvind: "basically a non-starter"); WFJ lacks bandwidth to document business rules
- **Oct 25:** Lumber/Oleg decided NOT to go forward with WFJ. Archived Nov 12.

### Phase 2 (Jan 2026 onward): Analytics Reporting — ACTIVE

**Un-archived Jan 21, 2026. This is the current project.**

- **New scope:**
    - Report builder for Lumber teams to configure/present tabular reports
    - Information widgets (data, charts)
    - Collaborative testing and Q&A
    - Remix self-hosted in Lumber's private cloud (AWS)

---

## Timeline (Phase 2)

### Jan 21, 2026 — Re-kickoff; scope set

- Mukund un-archivied the channel. Internal kickoff Jan 22; Lumber kickoff shifted to Jan 26.
- Staging env shared: https://stage-admin.lumberfi.com/reports/
- Solution architecture slides shared (Remix_Analytics_Lumber.pptx)

### Jan 26, 2026 — Lumber kickoff call; critical decisions

**Reza's post-call notes:**

1. **Timeline:** Shreesha demanding first-week-Feb go-live — "pointlessly aggressive." Reza managing expectation.
2. **Architecture:** Lumber data is multi-tenant Postgres on AWS → ETL to Snowflake (also on AWS) → Remix serves reports
3. **Auth:** Lumber portal passes customer identity/JWT token; Remix must validate token against Lumber API (`https://qa-platform.lumberfi.com/api/users/info`) to extract company ID
4. **Access model V1:** All users within a company see the same data (no row-level security for now)
5. **Scope constraint (critical):** Success = Lumber's Rajesh/Trinadh being able to build new reports themselves. Lumber *customers* building their own reports is **explicitly out of scope for V1**

**Architecture rationale (Vijay):** Postgres → Snowflake because Snowflake enables denormalized OLAP views across multiple future data sources (not just Postgres)

**Action items from kickoff:**

- Vijay/Chris/Didier: 2–3 slide solution architecture deck
- Reza: project plan with dates; access to Postgres QA DB + DDL
- Vijay: Snowflake cost estimate for Lumber

### Jan 29, 2026 — Cortex agent limitation found

- Padmanabha: `snowflake_cortex_generate_sql_service_acct` only returns `label` + `value` (designed for widget use case); useless for tabular reports
- John: agent was a starting point; extend it for new use cases
- **Resolution (Feb 6):** Padmanabha copied agents, renamed to `_v2`, modified the prompt for `widget_type = "table"` → https://remix-dev.remixlabs.com/e/edit/snowflake

### Feb 2–3, 2026 — Schema and data in place

- Reza posted Job Costing zip, Snowflake DDL (generated via Claude Code)
- Snowflake instance: https://app.snowflake.com/oxfsvki/tw30727/
- Padmanabha's playground: https://remix-india-beta.remixlabs.com/e/edit/padmanabha-playground/snowflake
- Notion pages set up with Mermaid schema diagram, table descriptions, SQL, CSV data
- Vijay: put the report screen in a library; use from Desktop
- **Access issue:** Arvind couldn't access `iaEj4QYboi` service agents → Padmanabha granted all Remix people admin access

### Feb 4, 2026 — Denormalization strategy settled

- Decided: import Postgres tables **as-is** (already denormalized for OLAP)
- No need to re-normalize on import; SQL functions do the join/materialization at ingestion
- Didier: approach mirrors Salesforce/EdgeSpring edgemart model
- Nightly sync (full or incremental); minimal transforms on import

### Feb 5, 2026 — Demo prep; auth and access gaps

- Arvind: demo is Feb 6 at 7am Pacific
- Snowflake auth setup: RSA key pair generated by Padmanabha, hardcoded in agents 4, 5, 3 (`private_key_encoded`, `public_key_fingerprint`)
  at https://remix-india.remixlabs.com/e/edit/lumber_reporting/configurator
- End users do NOT have Snowflake accounts — service account used for all queries
- **V1 access model confirmed:** all users within a company see the same data
- Arvind: need to generalize so 3rd parties can onboard and auth themselves (future work)
- Didier: per-Lumber-company data isolation is confirmed in-scope; row-level access modeling deferred

### Feb 6, 2026 — First Lumber demo

- Slides posted: Lumber update.pptx
- **Table configurator bug caught by Arvind (mid-demo):** Component was showing STATIC copied data, not live Snowflake data. No one on the Lumber side noticed.
- **Fix required:** Base case in catalog must always use LIVE querying via service agents at runtime; freezing data is a secondary/optional use case
- John deployed Gerd's Repository DB to lumber workspace `iaEj4QYboi`

### Feb 11, 2026 — 90-min working session with Lumber engineers

- Reza had session with Anand, Rajesh, Trinadh on reporting complexity
- **Significant risk flagged:** Lumber engineering team is building their own "contextual data" ETL/calculation layer (Java code + SQL functions). Anand said they'll be done by end of Feb.
    - Risk: they may say "we already solved the problem — why add Snowflake + Remix complexity?"
    - Note: General Ledger report uses Java; majority of reports use SQL functions on Postgres
- **Schema complexity:** payroll 38 tables, unions 19, prevailing wages 15, company config 40, compensation 39
- **Report features needed (future phases):** pagination, filters, column selection, facets, data download, drill-down by row

### Feb 12, 2026 — Auth integration design

- Chris: auth notes at https://github.com/remixlabs/mix-rs/issues/982
- **Two auth approaches:**
    1. **(Now feasible):** Anonymously-runnable gateway agent accepts Lumber JWT, validates via Lumber API, calls Snowflake agents internally (those agents only accessible via gateway)
    2. **(Future):** Full token exchange: Lumber token → Remix token tied to company identity
- If embedded in Lumber's page: company ID is in the JWT token directly
- Vijay: why is Java code mixed with SQL functions? (complexity concern)

### Feb 18, 2026 — Integration doc + UX approach finalized

- **One .remix file per report** (not a single .remix passed a report name)
- **Integration UX (decided):** Single new menu item in Lumber's sidebar above a separator → one click loads Remix reports panel → Remix takes over the full reports UX (white-label, no "Remix"
  branding)
    - Lumber keeps their existing reports menu intact; Remix panel is additive
    - Landing screen: dashboard of available reports; then two-panel view with report dropdown
- Didier prepared Google Doc integration spec for Lumber engineers: https://docs.google.com/document/d/1xHHOfZi94LdDfDbsktjrFc5IvBKJiyguDxXzafDv63s/edit
    - Goal: "we have a solution, ball is in your court; if you have an alternative, bring it"
    - Chris confirmed auth/security section is accurate

### Feb 20, 2026 — Active

- Reza, Chris, Didier on call with Lumber at 7am Pacific (Chris confirmed attendance)

---

## Key Technical Notes

- **Snowflake auth:** RSA service account; no end-user Snowflake accounts. Keys in agents 3, 4, 5 at `lumber_reporting/configurator`
- **Cortex agents v2:** At https://remix-dev.remixlabs.com/e/edit/snowflake — modified for table widget type (returns all columns, not just label/value)
- **DDL also needs `views` inbinding** in `snowflake_generate_ddl_service_acct` (Padmanabha flagged; Wilber to fix)
- **Live vs static data:** table component configurator must default to LIVE SQL at runtime; static data copy is secondary
- **Integration doc:** https://docs.google.com/document/d/1xHHOfZi94LdDfDbsktjrFc5IvBKJiyguDxXzafDv63s/edit
- **Workspace access:** Use `iaEj4QYboi` on `agt` server. Add Remix people as admins via workspace settings.