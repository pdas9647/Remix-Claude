# #rmx-delivery-lumber Slack Channel — Remix Labs

**Coverage:** Oct 9, 2025 – Feb 28, 2026
**Channel ID:** C09KL3K7P0D

---

## Customer: Lumber

- **What:** Construction payroll/HR SaaS; 100+ customers
- **Partner:** Workforce Junction (WFJ) — employee benefits
- **Workspace:** `iaEj4QYboi` | Snowflake: `https://app.snowflake.com/oxfsvki/tw30727/`
- **Staging:** https://stage-admin.lumberfi.com/reports/
- **Contacts:** Oleg (product owner), Manish (US, finance), Shreesha (timelines), Anand (payroll eng, India), Lyncean Patel (field/timesheets, India), Trinadh Baranike (reporting, India), Rajesh Kannan (reporting lead, first Remix user)
- **Remix team:** Mukund (GTM), Reza (CS lead), Vijay (product), Didier+Chris (platform/infra), Arvind (UX/build), Padmanabha (build), John (platform)
- **Notion:** [Lumber main](https://www.notion.so/Lumber-28c1d464528f801bac72d6c064d30db2) | [Postgres schema](https://www.notion.so/Schema-of-Lumber-Postgres-2f71d464528f8092b0dce634ef55bfb7) | [Job Costing](https://www.notion.so/Job-Costing-Report-2fb1d464528f808b987cce2583b7fb8a)
- **Integration doc:** https://docs.google.com/document/d/1xHHOfZi94LdDfDbsktjrFc5IvBKJiyguDxXzafDv63s/edit

---

## Phase 1 (Oct–Nov 2025): WFJ Benefits — DROPPED

Mobile + web app for WFJ benefits enrollment. **Dropped Oct 25** — WFJ POST API not exposed ("non-starter"), WFJ lacked bandwidth to document business rules.

## Phase 2 (Jan 2026+): Analytics Reporting — ACTIVE

Report builder for Lumber teams to configure/present tabular reports over Snowflake data. Self-hosted in Lumber's AWS.

### Architecture
- Lumber multi-tenant Postgres (AWS) → ETL to Snowflake → Remix serves reports
- Tables imported as-is (already denormalized for OLAP); nightly sync
- **Auth:** Gateway agent accepts Lumber JWT, validates via Lumber API (`qa-platform.lumberfi.com/api/users/info`), extracts company ID, calls Snowflake agents internally. Auth notes: mix-rs/issues/982
- **Snowflake auth:** RSA service account (no end-user SF accounts). Keys in agents at `lumber_reporting/configurator`
- **Access model V1:** All users within a company see same data (no row-level security)
- **Success criteria:** Lumber's Rajesh/Trinadh can build new reports themselves. Customers building their own is **out of scope V1**.

### Timeline

**Jan 21:** Re-kickoff. **Jan 26:** Lumber kickoff call. Shreesha demanding first-week-Feb go-live — "pointlessly aggressive" (Reza).

**Jan 29:** Cortex agent only returns label/value (widget-oriented). Resolution (Feb 6): Padmanabha created `_v2` agents for table widget type.

**Feb 2-3:** Schema + data in Snowflake. Notion pages with Mermaid diagrams, SQL, CSV.

**Feb 6:** First demo to Lumber. **Bug caught mid-demo:** table showing STATIC copied data, not live Snowflake data. Fix: base catalog component must always use LIVE querying.

**Feb 11:** 90-min session with Lumber engineers. **Risk:** Lumber building their own ETL/calculation layer (Java+SQL); may question need for Snowflake+Remix. Schema: payroll 38 tables, unions 19, prevailing wages 15, company config 40, compensation 39.

**Feb 18:** Integration UX finalized. One `.remix` per report. Single new Lumber sidebar item → dashboard of available reports → two-panel view. White-label, no Remix branding. Integration spec sent to Lumber engineers.

### Feb 24-28: Desktop + Facets + Crunch

**Desktop install [Feb 24-25]:** Fresh install 410 error fixed in v0.9920. Remaining: synced apps appear in Projects not Packages (appmeta pending). **Critical:** opening catalog at L0 auto-compiles + corrupts exec-only app. Mar 1: Arvind checked in latest lumber home/catalog/reports to `iaEj4QYboi` repositories.

**Facet components [Feb 25-26] (Mark):**
1. `snowflake data source setter` — select table
2. `snowflake facet builder` — per-column facets (TEXT + DATE)
3. Copy facet component into screen
4. `snowflake facet → CTE query generator` — accumulates facets into SQL (conditions only applied when set)
5. Execute against Snowflake

**Facet types in scope:** Date range, pill picker (month/quarter/year), month+year carousel, dropdown multi/single picker, keyword search.

**Table component [Feb 26] (Arvind):** Self-contained, configured by SQL statement string. CTE orchestrator overrides with filtered SQL when facets selected; returns default query before any selection. Forward pattern: each paste-able catalog component has CTE orchestrator built in.

**Crunch mode [Feb 27] — target: Wednesday Mar 4:**
1. **Infra:** Chris + Didier — agent server install + integration
2. **Tools:** Arvind + John — home, catalog, report mgmt, Studio publish (all 4 essentially functional)
3. **Content + QA:** Reza + India team — catalog components, Snowflake schema, report content, QA

**Scope pushback (Vijay):** Lumber tried to add scope. Vijay told Manish "his guys were smoking something." Conditions: one owner + full scoping exercise, or no new scope. Oleg to own scoping.
