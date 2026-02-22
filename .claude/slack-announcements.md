# #announcements Slack Channel — Remix Labs

**Coverage:** Dec 20, 2025 – Feb 22, 2026
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

---

## Channel Patterns

- Used for cross-team coordination: calling huddles, tagging specific people
- Lumber is the dominant active project (multiple huddles/week Jan–Feb 2026)
- Customer calls coordinated here (Lumber, Bomisco, Salesforce demo)
- Technical decisions and architecture notes sometimes posted inline (JWT structure, passkey design)