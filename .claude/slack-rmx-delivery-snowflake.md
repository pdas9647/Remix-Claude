# #rmx-delivery-snowflake Slack Channel — Remix Labs

**Coverage:** Mar 25, 2025 – May 2, 2026
**Channel ID:** C08JPJ6EES3
**Nature:** Snowflake GTM partnership, not a customer delivery

---

## Relationship: Snowflake Startup Accelerator

- **Remix joined the Snowflake Startup Accelerator:** May 29, 2025
- **Accelerator contact:** "Cam" (runs the program); bi-weekly check-in calls
- **Cam quote:** "out of all their accelerator teams we have by far the best demos"
- **Next step after graduating:** Snowflake AI Data Cloud Product Partner
- **Partner program slides:** https://docs.google.com/presentation/d/1Mx4M1yUqxsn2Nh-7kAH6Pt-SCwbqWhV4gLY5x-cBHoU/edit
- **Remix team:** Mukund + Fred Im (GTM), Vijay (product), Chris (platform), Didier, Shritesh
- **Snowflake contacts:** Cam (accelerator), various SEs

---

## Timeline

### Mar 25, 2025 — Channel created; first Snowflake call

- Fred Im arranged intro; Mukund/Vijay/John attended a 30-min call
- Snowflake contact (accelerator program) liked the value prop; brought in SE for follow-up

### Apr 8, 2025 — Technical deep dive

- 1-hour SE call; agenda: current + future state architecture, SPCS/Cortex/Native App plans, business value
- Canvas: https://figly.slack.com/docs/T02BD0B81/F08MF3AGUSV
- Chris's plan of attack for SPCS integration:
    1. Get mixer running on Snowpark Container Services (SPCS)
    2. Auth from mixer to Snowflake; run SQL queries
    3. Deploy amp + builder in SPCS
    4. (Stretch) Deploy as native applications

### Apr 9–10, 2025 — SPCS exploration

Switched GCP account to AWS (Snowpark requirement). Chris: mixer Docker image deployed in SPCS, agents invokable from CLI. SPCS pricing: ~$130/mo vs EC2 $30–43/mo.

### May 7–9, 2025 — Python/SQL bridge POC

No Rust Snowflake SDK → Chris wrote Python gateway between mixer and Snowflake; full SQL queries working through the bridge. Long-term plan: Go static library compiled into Rust mixer.

### May 29, 2025 — Officially in Snowflake Startup Accelerator

- Mukund: "we are now officially part of Snowflake's startup accelerator program"
- Kickoff call with Snowflake team; bi-weekly check-ins begin

### Jun 5, 2025 — Complete Remix runtime running in SPCS

- Chris: "complete remix runtime running in SPCS!" — major milestone
- Padmanabha built 2 Snowflake widgets (India team):
    - .remix file: https://storage.googleapis.com/rmx-content/draft/snowflake.remix
    - Builder project: https://remix-india.remixlabs.com/e/edit/snowflake
- Mukund thanks India team (Padmanabha, Sirshendu, Arka) after demos went well

### Jun 26–27, 2025 — Live programming in Snowflake; Cam check-in

- Chris: achieved **live programming 100% in Snowflake** (builder running inside SPCS)
- Issue: Snowflake's Content Security Policy blocks external fonts/icons in SPCS
    - Fix: bundle Google Fonts + MDI with builder (Didier), or deploy `_rmx_style` app from GCS
- Showed to Cam at the bi-weekly check-in → Cam confirmed best demos in the accelerator

### Jul 12, 2025 — Snowflake OAuth + Jul–Aug summary

OAuth-into-Snowpark working: Snowflake token → session token exchange → mixer in Snowpark. Demo: remix-dev.remixlabs.com/e/edit/agent-demo/oauth_home (direct REST + session-token styles). Jul–25:
Build Day demos submitted. Jul 30: GTM idea — free mobile widgets, charge for actionability. Aug 7–8: .remix upload to SPCS validated; **channel quiet after Aug 8 — resumed Feb 2026**.

### Feb 23–26, 2026 — Snowflake Marketplace submission push (Mukund)

Goal: submit listing by end of Feb 23 week. Bootstrap: mixer in Snowpark, libraries/styles, widget build flows, customer360 sample data, readme. Internal SPCS instance deployed at
`mw7zsh-oxfsvki-remix.snowflakecomputing.app/e`; admins: Chris/Mukund/John/Vijay/Wilber. Feb 26: Mukund asked John+Wilber to test widget build flow.

---

## Key Technical Notes

- **SPCS stack:** mixer (Rust) in Docker on SPCS; Python bridge for Snowflake SQL (no Rust SDK)
- **Auth pattern:** Snowflake OAuth → session token exchange → short-lived token for mixer in Snowpark
- **CSP issue in SPCS:** fonts/icons need bundling or `_rmx_style` app deployed inside SPCS
- **SPCS pricing:** expensive vs raw EC2 (~3–4x); factor into customer pricing
- **Native app path:** packaging Remix as a Snowflake Native App would solve OAuth admin friction for self-serve GTM
- **Snowflake account:** `oxfsvki/tw30727` (used for Lumber Snowflake work)

---

## Key Links

- Snowflake widgets project: https://remix-india.remixlabs.com/e/edit/snowflake
- Snowflake widgets .remix: https://storage.googleapis.com/rmx-content/draft/snowflake.remix
- OAuth demo: https://remix-dev.remixlabs.com/e/edit/agent-demo/oauth_home
- Accelerator guidebook: https://docs.google.com/document/d/1USk3DKHLsFXnyNy-8XRFjWFYNB09kXb14tBhl7_dJRc/edit

### Mar 19, 2026 — Cross-application-boundary data access resolved (Chris)

The Snowflake Native App packaging had a blocker: app code can't access objects outside its own application database without explicit "references" created at install time. Chris got the generic
reference-setup working. README updated in the package (visible at: `https://app.snowflake.com/oxfsvki/remix/#/apps/application/REMIX_DXP/security/readme`). Notion doc updated with notes on
deployments, SPCS queries/inserts: https://www.notion.so/Remix-in-Snowflake-Snowpark-2841d464528f804ba6fbe2d8feb7a963

**Still open:** Decide on bootstrap content + document the setup process.

### Mar 21, 2026 — Lumber runtime flow for Marketplace submission (Mukund → Arvind)

Vijay suggested including the Lumber runtime flow in the Snowflake Marketplace submission. Mukund asked Arvind: (1) what's the latest version and is it accessible from Remix DT Studio? (2) Does this
mean providing the reviewer access to a workspace with Remix agents? No replies recorded in this period.

### Mar 24–27 — SPCS Active Build Begins

**Two SPCS environments established [Mar 24, Chris]:**

- **Dev (build target):** `https://mw7zsh-oxfsvki-remix.snowflakecomputing.app/e` — Mukund, Arvind, Padmanabha as workspace admins
- **Consumer test:** `https://iqlr4z-oxfsvki-remix-spcs-demo.snowflakecomputing.app/e` — empty; for verifying export+install flow
- Strategy: build/test in dev org, export+install into consumer org to confirm it works.

**SPCS plugins + mixcore still broken [Mar 27, Chris]:** mixcore not loading → most FFIs unavailable. Workaround: service agent for file assembly. Working export screen:
`.../snowflake_test/export_with_agent?app=<app_name>`.

### Mar 25 – Apr 3 — SPCS Active Build (Group channel C09GTR1TDC3)

**Core flow working [Mar 25, Padmanabha]:** `query_customer` agent → `REMIX_DXP.CORE.CUSTOMER_MASTER` (exposed as view). Customer360 list UI built end-to-end. Two initial blockers: tailwind not
loading + export plugin blank.

**Data access constraints:** External tables must be explicitly referenced. Chris exposed 7 `CUSTOMER_360.RAW_DATA` tables as views in `REMIX_DXP.SHARED_SCHEMA` (e.g. `CUSTOMER_MASTER_DEMO`) —
accessible from both remix + remix_spcs_demo accounts without changes on consumer side.

**Auth in SPCS:** No OAuth/RSA needed. Container provides session token at `/snowflake/session/token`. Agents must be built + deployed server-side (Make) before invocation from screens.

**GET_DDL not possible** on objects outside the app. Use `INFORMATION_SCHEMA.COLUMNS` for schema metadata. Padmanabha hardcoded DDL for 7 demo tables as workaround.

**Cortex:** `AI_COMPLETE` → use `SNOWFLAKE.CORTEX.COMPLETE`. Enable per Snowflake Native App docs. Default warehouse (`COMPUTE_WH`) set on service [Apr 2].

**Agent env URL bug [Apr 3]:** `client_frontendBaseURL` / `client_backendBaseURL` = `___URL_PLACEHOLDER____` when invoked server-side. Fix:
`env.getWithDefault(requestHeaders.origin) > withDefault(env.frontendBaseURL)`.

**cloud_workspace adapted for Snowflake [Apr 2]:** Installed in both accounts. Allows granting permissions beyond initial admin.

**Marketplace trial flow confirmed [Apr 1, Mukund]:** Install → SPCS container → Customer 360 sample + widget builder → widget preview → contact Remix.

**"Launch app" button [Apr 3]:** Setting default web endpoint shows Launch button on app page; `/` can redirect to a runtime screen.

**Status Apr 3:** widget_preview working in dev (mw7zsh). Padmanabha posted to demo env for review:
`iqlr4z-oxfsvki-remix-spcs-demo.snowflakecomputing.app/e/preview/customer360_spcs_test/widget_preview`.

### Apr 4–11, 2026 — Marketplace Listing Push + SPCS Build

**Widget preview polished (Padmanabha, Apr 6):** Updated aggregate card, clearer metadata, visible generated SQL. Deployed to iqlr4z consumer env. Mukund approved; has a landing page request.

**Platform tasks remaining (Chris, Apr 6):** Include initial .remix files in image + load on startup; support default web endpoint → demo screen redirect.

**Tailwind styles missing in SPCS (Apr 7):** Styles work on remix-india but missing in SPCS env. Fix: export `_rmx_tailwind` from remix-india, reinstall in SPCS account.

**Landing page review (Padmanabha → Mukund/Chris, Apr 7):** Marketplace landing page at `mw7zsh.../snowflake_widgets_analytics/home`. Chris: fix title/subheader typos, remove placeholder descriptions.
Mukund approved styling and edited content.

**App package v1.98 ready (Chris, Apr 8):** Package updated; ready to publish as public listing. Mukund: listing work not yet started.

**DB rename breaks widget preview (Padmanabha/Chris, Apr 10):** mw7zsh `widget_preview` broken — `REMIX_DXP` not found. Cause: Chris renamed underlying DB for public listing (now `remix_dxp`). Fix:
set `DATABASE` input to `REMIX_DXP_INTERNAL`. Tip: `SYS_CONTEXT('SNOWFLAKE$APPLICATION', 'NAME')` returns current app DB name dynamically.

**iqlr4z consumer env broken (Apr 10–11):** `REMIX_DXP_INTERNAL` not present in iqlr4z. Root cause: Chris accidentally deleted the internal listing when renaming for public submission — needs
recreation. Data access non-functional; Snowpark service still running.

### Apr 13, 2026 — Provider billing enabled; Stripe integration needed (Chris)

Snowflake Marketplace account now enabled for **provider billing**. Before publishing a paid listing, someone must set up Stripe integration for receiving payments.

## Apr 18–25, 2026 Additions

**SPCS consumer env down [Apr 23, Padmanabha]:** `iqlr4z-oxfsvki-remix-spcs-demo.snowflakecomputing.app/e` returning “Service MIXER not reachable: no service hosts found” — consumer SPCS instance
down. No resolution recorded in window.

**Snowflake Marketplace review feedback [Apr 23, Mukund]:** Reviewer “So Hyun” (Snowflake) returned two **required** fixes before listing approval:

1. **Install failure:** `REMIX_DXP.CORE.MIXER does not exist` in VERSION_SETUP phase — dependency ordering bug in setup_script.sql. Must test install from a **separate consumer account** (not provider
   account).
2. **In-product README required:** Must be readable inside Snowflake (not just on Notion). Must include: product description, config steps, stored procedures/UDFs, required privileges, example SQL
   commands.

### Apr 29–May 2, 2026 — Snowflake Summit GTM Strategy

**4-part GTM [Apr 29, Mukund]:** Snowflake Summit (Jun 1–4) strategy:

1. **Build on Desktop, deploy in Snowpark** — Marketplace listing + 2-week free trial; target SIs + GTM partners
2. **Embed analytics anywhere** — Desktop Studio + "Remix Canvas" (Vijay’s new build surface; placeholder name, to be renamed); embed reports/dashboards into any page
3. **Mobile widgets from Snowflake data** — new branded mobile app with Snowflake as data source
4. **Data lake → digital experiences** — Snowflake data → interactive RCS/web/chat end-user experiences

Note: "build in Snowflake" = build on Desktop Studio, deploy natively in Snowflake. Snowflake Intelligence app flagged as potential integration point. Discussion continued Apr 30.
