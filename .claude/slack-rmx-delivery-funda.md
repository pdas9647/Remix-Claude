# #rmx-delivery-funda Slack Channel — Remix Labs

**Coverage:** Sep 15, 2025 – Dec 2, 2025
**Channel ID:** C09G04S596C

---

## Customer: Funda

- **What it is:** WhatsApp-based founder networking community (~600–700 founders); not a revenue-generating customer
- **Admins:** Mahesh (runs the WhatsApp group), Vikrant (CEO of JobTwine, confirmed member count)
- **Current site:** funda.club — barely used (one blog post, 17 startups listed, no events)
- **Workspace:** `sEt2qxPydL`
- **Project names:** `funda` (main), `funda_clipper`, `funda_web`
- **Web app URL:** https://remix.app/about/funda
- **Notion page:** https://www.notion.so/Funda-26f1d464528f807f9782d5bffc5401a5
- **Remix team:** Vijay (design owner), Reza (CS lead), Mukund (GTM), Padmanabha + Arka + Sirshendu (build), Arvind (UI design), John (platform)

---

## Goals

Reza set three explicit goals for this engagement (Sep 15):

1. Deliver on Vijay's personal commitment to provide a working product to the Funda community
2. Use this as a test case for the Remix CS delivery process
3. Introduce Remix to 600+ founders → generate 5 trials/sales prospects

---

## Timeline

### Sep 15, 2025 — Channel created; first release scope defined

- **First release: member directory** (fully delivered by Remix CS, no user customization)
    - Web app on remix.app, responsive (mobile + laptop)
    - Auth: Google, Office365, Apple
    - Profile editing: name, title, company, LinkedIn, mobile, email (with show-to-members toggle), photo upload, "reach out to me" tags
    - Member listing: search by name, email, company, tag; directory listing with contact options

### Sep 18, 2025 — Extension tasks tracked

- John linked extension task tracking: https://remix-dev.remixlabs.com/e/edit/members/extension_tasks

### Oct 9, 2025 — Status check

- Mukund wants status; pushing to knock off simple items and have a working showcase

### Oct 14, 2025 — Design call with Vijay; rescheduled

- Reza scheduled a Zoom call for Vijay to walk through design thoughts
- Arka sick; Reza missed it; Padmanabha confirmed rescheduled post that night's Lumber call

### Nov 4, 2025 — Build complete; two blockers remain

- Padmanabha: "We are done with Funda" except:
    1. **UI** — waiting on Arvind's design
    2. **Snowflake Cortex search** — waiting on Mark
- Project URL: https://remix-india.remixlabs.com/e/edit/funda
- Ongoing bug fixes as found during testing

### Nov 12, 2025 — URL format clarified

- Arvind questioned why the app was being shared via direct `/run/<file>` URL instead of a clean vanity URL
- Vijay: "Sure, not an issue either way" → settled on using `remix.app/about/funda`

### Nov 19, 2025 — Significant update shipped; demo-ready

- Padmanabha shipped a major update with:
    - Async UI with loading states during AI calls
    - AI-generated person summaries on successful clipping
    - Edge case handling (validates person/company LinkedIn URLs only)
    - Admins can view/manage lists directly in the clipper
    - Admin functions for updating contact info
    - Case-insensitive search on People & Companies pages
    - Visitor stats collection via beacon-event
- Links:
    - Clipper: https://remix-india.remixlabs.com/e/edit/funda_clipper
    - Web app: https://remix-india.remixlabs.com/e/edit/funda_web
- **Bug:** New agent created didn't have anon access → search returned no results for Vijay → Padmanabha fixed immediately
- **Vijay feedback:**
    - "Looking really good — great work"
    - Wants to explore: "view full profile" as an intermediate modal (tagged Arvind)
    - Skills search is "real nice" — found Mark by searching "Haskell"
- Ready for call with Mahesh & Vikram (Funda admins)

### Nov 24, 2025 — Full flow demo call with Funda team

- Mukund: "Call with Funda team this afternoon — show entire flow, get feedback, finalize go-live plan"

### Nov 26, 2025 — Contact data shared post-call

- Reza shared an HTML file (likely LinkedIn profile output)
- Arvind shared `funda_ingest.json` — JSON array of contacts for Arka to use

### Dec 2, 2025 — LinkedIn sign-in button added to `_rmx_auth`

- John published a version of `_rmx_auth` with a LinkedIn sign-in button
    - Module: https://remix.remixlabs.com/e/edit/_rmx_auth/_rmx_auth_linkedin
    - .remix file pre-uploaded to Funda workspace (sEt2qxPydL) wip folder
- **To activate:** Chris must create the `linkedin` auth config; then replace the auth .remix via:
  https://remix.remixlabs.com/customers/deploy_rmx_auth?name=Funda&workspace=sEt2qxPydL
- Channel went quiet after this — pending Chris setting up LinkedIn OAuth

---

## Key Technical Notes

- **Auth:** Google, Office365, Apple live; LinkedIn being added (pending Chris auth config)
- **Snowflake Cortex search:** Still pending Mark as of Nov 4, 2025
- **Anon access bug pattern:** When creating a new agent for a workspace, remember to grant anon access if the app has public-facing screens
- **Visitor stats:** beacon-event added to the web app
- **Deploy auth .remix URL pattern:** `https://remix.remixlabs.com/customers/deploy_rmx_auth?name=<Name>&workspace=<wsId>`
