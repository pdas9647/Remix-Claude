# #announcements Slack Channel — Remix Labs

**Coverage:** Dec 20, 2025 – Apr 18, 2026
**Channel ID:** CK8S97CUV
**Purpose:** Important info everyone should read; also used for cross-team huddle coordination

---

## Key Events

### Jan 7, 2026 — WebAuthn / Passkey design for Funda

- Vijay, Chris, Gerd, John discussed passkey (WebAuthn) auth design for Funda
- **Gerd's design**: A passkey agent handles all checks; it fetches a platform token from `auth.remixlabs.com` using a securely stored private key; the auth DB stores a `pubkey` record with userId +
  claims that passkeys map to

### Jan 13–16, 2026 — Salesforce demo

- Team prepped retail + hospitality chat demos for a Salesforce meeting
- Scenarios: shopping cart assistance, store finder with appointment booking, product FAQs, Airbnb-style nearby amenities & operating instructions
- Demo workspace: https://remix.remixlabs.com/e/edit/service_desk/chat_corner
- Demo went well; Salesforce team was impressed

### Jan 20–22, 2026 — Desktop readiness status check

- Arvind called for a quick status check on readiness to onboard external users to Desktop
- Open questions: home app, catalog browsing/install flow, .remix v1 vs v2, catalog moving to workspaces
- Benedikt: tracking desktop issues at https://github.com/remixlabs/mix-rs/issues/937 — 2 blockers at the time
- Benedikt wanted a broader discussion on the role of the desktop app (separate announcement planned)

### Jan 27, 2026 — Lumber: Snowflake setup + catalog agents

- Snowflake Service Account Setup Guide: https://www.notion.so/Snowflake-Service-Account-Setup-Guide-2a81d464528f80de8febcf0b590be206
- Agents shared from `remix_labs` catalog: standard DDL agent, Cortex generate SQL, execute SQL, copy configured component, copy configured card

### Feb 2, 2026 — Lumber kickoff: scope alignment

- Full team meeting (Reza, Arvind, Mukund, Vijay, John, Chris, Wilber, Mark, Padmanabha)
- Agenda:
    1. Review job costing report actuals
    2. Align on PostgreSQL → Snowflake migration options
    3. Confirm Chris/Didier action items
    4. Assess desktop packaging readiness for Lumber delivery
- Lumber Snowflake schema: https://app.snowflake.com/oxfsvki/tw30727/#/data/databases/LUMBER_FI/schemas/JOB_COSTING
- Builder: https://remix-india-beta.remixlabs.com/e/edit/grid/snowflake

### Feb 11, 2026 — Lumber: auth + Snowflake

- Call: Vijay, John, Mukund, Didier, Arvind, Chris
- Focus: authentication — Chris examined Lumber JWT structure:
    - Claims include `tenants` with `COMPANY_ADMIN` role, `DS` auth
    - Issue tracked: https://github.com/remixlabs/mix-rs/issues/982

### Feb 13, 2026 — File uploading discussion

- Simon initiated cross-team chat on file uploading: Wilber, John, Mark, Gerd

### Feb 15, 2026 — Theme migration: delete old styles (@all)

- **Arvind (@all)**: Delete "Remix Style" and "Remix Styles" themes from Desktop → Themes tab → Trash → permanently delete
- Only keep `rmx_tailwind` (auto-updated with each desktop release)
- **Migration process** (Didier):
    1. Remove `_rmx_style` / `_rmx_styles` from the settings module (apps still render; styles marked deprecated)
    2. Open each screen; deprecated cards show an orange warning at L2; swap for equivalent new library cards
    3. No automated migration — no 1-to-1 mapping between old and new styles
- **Known bug**: some users see "database _rmx_style doesn't exist" error when deleting — no desktop rescue/cleanup tool exists yet (Chris: "Not something we've really figured out yet")

### Feb 17–18, 2026 — Lumber: auth + Snowflake + catalog (active)

- Work split assigned by Reza:
    - **Chris / Didier / Arvind** — authentication
    - **John / Mark / Vijay** — Snowflake and facets
    - **Arvind** — Catalog screen / home app / components in library
- Service agent deploy huddles ongoing (John)
- Simon working on storing saved values in service agents + supporting multiple stored instances

### Feb 23, 2026 — Lumber: cross-team huddle (export widgets)

- Reza called a Lumber huddle with: Arvind, Vijay, John, Mark, Didier, Chris, Mukund
- John shared link during huddle: https://remix.remixlabs.com/e/edit/cloud_workspace/export_widgets
  — suggests discussion was about export_widgets screen/flow for Lumber

### Feb 28, 2026 — Lumber: reports building/packaging decision

- Arvind called a meeting at 11AM California: Lumber reports building/packaging
- **Required**: Reza, Vijay, John; **Optional**: Didier
- Short huddle; outcome: **"agreed with Vijay"** (Arvind) — specific agreement not recorded in thread
- Context: Lumber report packaging approach / how reports are assembled and delivered

### Mar 4, 2026 — Lumber: requirements walkthrough with Oleg + Greg

- Reza coordinated post-standup Lumber discussion with full team (Vijay, John, Mukund, Chris, Didier, Arvind, Padmanabha, Nivesh, Sirshendu, Mark, Wilber)
- Reza + Vijay had update from client call; requirements walkthrough call at 3:30pm Pacific with Oleg and Greg
- Arvind and John joined to hear requirements direct — walkthrough of actual reports and scope for first release go-live
- John shared Snowflake CTE docs post-call: https://docs.snowflake.com/en/user-guide/queries-cte

### Mar 5, 2026 — Arvind: zagjs + lit + slots recommendation for web components

- **Arvind**: Lumber is the forcing function for getting accessible, interactive components (combo box, menu, select) into the system quickly
- **Recommendation**: `zagjs + lit + slots for cards/comps everywhere`
- **Tyler**: Invoking web component methods with parameters is messy — current approach forces passing data through attributes (see `rmx-voice-chat-headless` manifest as example)
- **Simon**: Confirmed invoking parameterless methods works fine via `event` type in manifest; parameterized methods are the problem — "actions with payloads all over again"
- Discussion started but no final decision recorded; huddle held with Vijay

### Mar 6, 2026 — Lumber huddle + product strategy

- John started Lumber huddle
- Reza checking in with Arvind, John, Vijay for follow-up

### Mar 9, 2026 — Lumber: set client expectations on first phase

- Reza: need to go back to Oleg on Lumber channel to set expectations on when updated deliverables will be shown
- Proposed Thursday demo of first phase of delivery
- Arvind + Reza + Mark huddle to discuss outstanding delivery items and ownership

### Mar 11, 2026 — Lumber: first drop scope defined + table component work

- **Reza defined "first drop" scope**: (a) Snowflake instance set up by Remix, (b) schema covering 5 reports set up by Remix, (c) dummy/test data populated by Remix, (d) Remix agent server hosted by
  Remix + Desktop + mechanism to preview/publish reports (not yet integrated with Lumber app). No dependency on client for first drop.
- Reza: will take care of Snowflake schema + data population; asked Arvind to propose a delivery date
- Arvind: spending the day getting base table component working with primitives and large/varied sample data; owes Mark SQL overlay
- Mark: tested sorting/pagination on CTE-based query, ready to plug into updated table component using primitives
- Arvind: once table component takes shape, need to figure out communication between table config and sort/page/column-order (highly dependent on config)
- Arvind also noted: need to handle CSV download of object lists

### Mar 11, 2026 — Table huddle + L3 data tree issue

- Arvind called table huddle with Vijay, Gerd, Simon, Didier
- Simon referenced L3 data tree issue: turntable#11804
- Simon attempted repro but couldn't reproduce the huge-tree scrolling problem

### Mar 12–14, 2026 — Lumber: frequent sync huddles

- Mar 12: Reza + Arvind Lumber discussion (Vijay on customer call)
- Mar 14: Vijay called Lumber quick sync (Reza, Mark, John, Arvind, Didier); Arvind no major updates since prior sync

---

## Channel Patterns

- Used for cross-team coordination: calling huddles, tagging specific people
- Lumber is the dominant active project (multiple huddles/week Jan–Feb 2026)
- Customer calls coordinated here (Lumber, Bomisco, Salesforce demo)
- Technical decisions and architecture notes sometimes posted inline (JWT structure, passkey design)

### Mar 17, 2026 — Lumber: client report spec shared + huddle

- **Reza** called a Lumber conversation at 10:00am Pacific
- Attendees: Vijay, John, Arvind, Didier, Mark, Wilber
- Reza shared **"Report spec for Remix.pdf"** — a document from client Oleg (Greg) defining report requirements
- Huddle started immediately after

### Mar 18, 2026 — Lumber huddle #1: India team + Vijay

- **Reza** called "Lumber huddle #1 - India team + Vijay Chakravarthy"
- Attendees: Sirshendu, Arka, Padmanabha, Nivesh, Arvind
- Reza included Arvind: *"would be good to stay in sync since you have a view of all the moving parts"*
- **Padmanabha** shared active workspace during huddle: `https://remix-india.remixlabs.com/e/edit/lumber_agents`

### Mar 19, 2026 — Huddle (no details)

- A huddle was started (Slackbot notification only; no message content)

### Mar 31, 2026 — Remix file HTML harness review

- **Arvind** called a cross-team huddle: "remix file html harness review"
- Attendees: Simon, John, Wilber, Mark, Didier, Vijay

### Mar 31, 2026 — Lumber: delivery scope and model discussion

- **Reza** called huddle with Vijay, Arvind, John: "Lumber delivery scope, what is left and delivery model"
- No substantive outcome recorded in thread; coordination only

### Apr 3, 2026 — Lumber: report builder Notion page + Claude-generated node graphs

- **Reza** shared Notion page with Claude-generated node graphs for **5 Lumber reports**: https://www.notion.so/Report-Builder-3351d464528f800d8b5ce12f6517c93f
- Each report has SQL statements in data fetch nodes + CTEs
- Source: PDF from client **Oleg** (Greg) with 5 report specs
- **Missing**: HRIS report — Anand has not provided HR data
- "Project labor overview" report added by Reza to show the bubble chart
- Attendees: Reza, Arvind, Mark, John, Vijay

## Apr 6–9: Lumber Delivery Push

**Delivery checklist review huddle [Apr 6, Arvind → Reza/Mark/John]:** Post-standup huddle to review outstanding Lumber delivery items. Checklist tracking in
Notion: https://www.notion.so/2ed1d464528f83f2808f0188ca86bde2

**Lumber architecture + urgency [Apr 7, Reza/Vijay/Arvind]:** 21-reply coordination thread:

- **Vijay:** "We really need to get back to them about when we can give them something" — customer timeline required urgently
- **Architecture confirmed:** Composable components — data fetch, display, and facets as separate, configurable pieces wired together in Studio (hardwiring where it makes sense, State only where it
  doesn't complicate component code). Arvind: 100% aligned.
- **Arvind's tasks by Apr 8 morning:** (1) Table config UX — select columns, sticky left column, grouped headers (TUI grid); (2) basic chart config UX
- **Errors** can live inside components
- **Colspan/grouped header config** is a pure config object — Arvind will work through it
- Reza shared Claude-generated report assets: https://www.notion.so/Claude-Code-generated-assets-3371d464528f809c9c5ed929c8709ecd

**Lumber huddle [Apr 8, Reza]:** Logistics only — no substantive decisions recorded.

## Apr 11–18, 2026 Additions

### Query perf: relationship subqueries [Apr 15, 43 replies]

Lumber queries slow (9s+): QB emits N separate `AND` subqueries for relationship fields instead of a merged subquery. Decision: **optimize in mix-rs engine** (not QB codegen; Simon confirmed no
codegen changes needed). Fred implementing: mix-rs#998 (9s→2.5s, approved Apr 16), mix-rs#1094 (optimizer PR, approved Apr 17), mix-rs#1095 (Chris follow-up suggestions).

### Webcomp publish + JS invoke [Apr 14–16]

- Simon (Apr 14): new macro Manifest → web comp node; JS external action via headers
- Apr 16: JS invoke huddle (Didier/Vijay/Simon/John)

### Lumber huddles [Apr 14–17]

- **Apr 14** — Desktop handoff (Reza + Didier; Arvind unavailable)
- **Apr 16** — Lumber sync (Arvind/Reza/Vijay/Didier/Chris/Mark); Lumber auth in scope
- **Apr 17** — Lumber call (Didier/team); Chris referenced mix-rs/issues/982 (auth token info)
