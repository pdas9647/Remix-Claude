# #rmx-delivery-snowflake Slack Channel — Remix Labs

**Coverage:** Mar 25, 2025 – Feb 28, 2026
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

- Initial Snowflake account (Shritesh) was on GCP; Snowpark required AWS/Azure → switched
- Chris: got mixer Docker image deployed in SPCS; can invoke agents from command line
- SPCS pricing concern: smallest instance $130/mo vs EC2 equivalent at $30–43/mo
- Snowflake shared DataOps.live approach for SPCS deployment

### May 7–9, 2025 — Python/SQL bridge POC in SPCS

- No Rust Snowflake SDK → Chris wrote tiny Python gateway server between mixer and Snowflake
- Got full SQL queries working from agent via the bridge:
  `SELECT * from emp_basic` returning results through mixer
- Vijay shared pyremix/pyo3 work — Python-callable Remix workspace:
  `from pyremix import Workspace; ws.invoke('pyremix', 'demo', ...)`
- Chris: long-term better approach = Go static library compiled into Rust mixer (Python is a nicer POC)
- Accelerator guidebook: https://docs.google.com/document/d/1USk3DKHLsFXnyNy-8XRFjWFYNB09kXb14tBhl7_dJRc/edit

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

### Jul 12, 2025 — Snowflake OAuth into Snowpark working

- Chris: OAuth-into-Snowpark flow working (forked Snowflake Go library; traced Python + Go connectors)
- Mechanism: Snowflake OAuth token → exchange for short-lived session token → used against mixer in Snowpark
- Demo screen (both OAuth styles): https://remix-dev.remixlabs.com/e/edit/agent-demo/oauth_home
    - Style 1: direct REST API calls with Snowflake token
    - Style 2: token exchange → session token for mixer service in Snowpark

### Jul 24–25, 2025 — Snowflake Build Day

- Submitted to Snowflake Build Day demos; Snowflake Cortex feature reference shared

### Jul 30, 2025 — GTM strategy discussion: free widgets for Snowflake users

- Mukund/Vijay idea: "Free-forever mobile widgets (Android, iOS) for all Snowflake users"
    - Download Remix app → pick widget template → Snowflake OAuth → widgets installed; no flows/actions unless paid
- **Vijay's framing:** give away enterprise widgets across data sources (incl. Parquet); charge for *actionability* (tap-through flows)
- **Technical friction:** Snowflake OAuth setup requires a high-level admin per account; can't be fully self-serve without native app install
- Chris: if Remix listed as native app in Snowflake Marketplace, OAuth config could be packaged in
- Vijay mentioned Cisco interest in a "digital twin for every employee" initiative

### Aug 7–8, 2025 — .remix upload to SPCS; last known Snowflake call

- Chris: direct .remix file upload from local builder to SPCS working (concept validated for desktop too)
- Aug 8: bi-weekly call with Snowflake ("Remix TV show"); Reza showed Alteryx/Snowflake demo
- Snowflake predictably asked: "is it using Cortex?" → Vijay: "tell them to enable Cortex for us for free"
- **Channel went quiet after Aug 8, 2025 — resumed Feb 2026 with Marketplace push**

### Feb 23–26, 2026 — Snowflake Marketplace submission push (Mukund)

**Goal:** Submit Remix listing to Snowflake Marketplace for approval/review by end of Feb 23 week; initiate marketing campaign the following week.

**Bootstrap experience checklist (Mukund's recap):**
- Chris working on mixer (mixr) running inside Snowpark with auth
- Initial libraries + styles + demo workspace to set up
- Widget building flows in web: pick data source, preview widgets
- Sample Snowflake tables with flows (e.g. customer360 dataset)
- Instructions / readme
- Mobile app experience: separate step, optional; web-side auth comes standard with Snowflake

**Deployed internal SPCS instance (Chris):**
- Builder URL: `mw7zsh-oxfsvki-remix.snowflakecomputing.app/e`
- Default workspace available at that URL; admins: Chris, Mukund, John, Vijay, Wilber
- `snowflake_widgets` copied over; `cloud_workspace` tool present but not fully editable via builder
- Runtime screens broken (assets moved to wrong place in image — needs fixing)
- Demo of run-inside-SPCS auth: `.../e/edit/run_in_spcs`
  - `run_sql` agent runs server-side, calls Snowflake API with local token
  - `use_run_sql` screen demonstrates calling it from the browser
- **Auth note:** Snowflake account has two URL forms (`yub45648.snowflakecomputing.com` and `oxfsvki-remix.snowflakecomputing.com`); same account, different locator formats. MFA codes work for both once the correct form is used. Mukund needed a role grant from Chris.

**Feb 26:** Mukund asked John + Wilber to test the widget build flow in the SPCS env — wants to submit the marketplace listing ASAP.

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