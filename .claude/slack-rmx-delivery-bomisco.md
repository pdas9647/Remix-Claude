# #rmx-delivery-bomisco Slack Channel — Remix Labs

**Coverage:** Oct 17, 2025 – Feb 11, 2026
**Channel ID:** C09MW6WCP6C

---

## Customer: Bomisco

- **Contact:** Mihir Mehta (mihir@bomisco.com / mihirkm@gmail.com), on Windows
- **Workspace:** `0ry4DirCMQ`
- **Project:** `bomisco_linkedin_hubspot_clipper`
    - URL: https://remix-india.remixlabs.com/e/edit/bomisco_linkedin_hubspot_clipper
- **Extension .remix file:** https://agt.files.remix.app/0ry4DirCMQ/_rmx_files/extension_remix_files/linkedin_hubspot_clipper.remix
- **Remix team leads:** Mukund (GTM), Reza (CS), Wilber + John (build), Tyler (extension)

---

## Timeline

### Oct 17, 2025 — Channel created; deal in final stages

- Mukund: Bomisco is a small company run by Mihir Mehta (a friend); use cases:
    - Widgets: HubSpot, Confluence
    - Chrome extension: LinkedIn → HubSpot clipper (person + company)
    - Warm leads flow from LinkedIn for a given ICP
- Wilber had already installed Remix Widgets connect app in their HubSpot instance
- Action items: create workspace, get LinkedIn clipper working
- Tone: Mukund driving; wants quick deal closure for the widgets use case

### Oct 23, 2025 — Deal signed

- Mukund: "Bomisco just signed off on the project"

### Nov 24, 2025 — Setup request

- Mukund asks Reza to set up Mihir with the clipper and workspace while Mihir is in India

### Dec 22, 2025 — First live attempt fails (HubSpot OAuth error)

- Reza, Arka, Padmanabha, Nivesh were on a Zoom with Mihir for go-live on LinkedIn clipper → HubSpot flow
- Chrome Extension install + login issues were resolved; main blocker: HubSpot OAuth threw an error when selecting the HubSpot account
- Reza asks: how to revoke HubSpot auth to re-trigger OAuth
- No follow-up in thread (unresolved at that point)

### Jan 29, 2026 — Token invalid error; Wilber tries V2 .remix

- Mihir pinged Mukund: "token is not valid" error while trying to clip contacts for an ongoing campaign
    - Vimeo recording: https://vimeo.com/1159432933/c6531c66a5
- Wilber rebuilt the app and switched to .remix file V2 (previously V1); extension still working on his end
- Mihir still had the same issue after the fix attempt

### Jan 30, 2026 — Inconsistent LinkedIn extraction

- John and Wilber ran tests using the extension clip tester + Bomisco contacts screen extract definition
- Same LinkedIn profile: sometimes worked, sometimes failed; working after a page refresh
- Hypothesis: LinkedIn is serving different HTML for the same profile at different times
- John: "not sure what the long term solution is here"

### Jan 31, 2026 — Mihir finally up and running

- Mukund thanks Wilber, John, Tyler for 2 days of effort
- **Key action item:** Mukund → Arvind: Copilot UX needs a full overhaul ASAP
    - "Mihir got super confused with our UI"
    - Scope: icons at top, sign-out flows, edit buttons — everything
    - (Arvind acknowledged with 👀)

### Feb 5, 2026 — Chrome extension public update (context)

- Chrome extension updated via the regular Thursday morning platform cycle (noted in #ops)
- Mihir had been using a manually-installed older version, not the Play Store version

### Feb 6, 2026 — App stopped working again

- Mihir reports: app not firing up at all since Feb 5; had to refresh/re-login repeatedly even on Feb 4
- Mukund did a Zoom call with Mihir; had him reinstall from Chrome Store (he was on old manual install)
    - Recordings: https://vimeo.com/1162416964/ and https://vimeo.com/1162416963/
- Error: "Contact was not extracted" with no further detail
- Chris/Tyler/Fred tested on Mac + Windows + Play Store + latest Chrome: **works for team but not Mihir**
- Fred (Windows) was able to clip further than Mihir got
- Hypothesis: HubSpot connection state on Mihir's end may interfere; no confirmed root cause
- Tyler: would need a specific repro; nothing actionable without one
- **Side note (Mukund):** Customers see internal junk workspaces when adding workspace in Copilot — Tyler says you can set a custom home app per workspace to avoid this

### Feb 8, 2026 — New error: "Lets Get Bomiscuous" error screen

- Mihir gets a new/different error; screenshot shows error title "Lets Get Bomiscuous" (Bomisco's own company tagline — not our copy)
- Vijay: "this is not good; get a repro and file a high-priority bug"
- Workaround: force quit Chrome (Wilber)
- Vijay later: "we need to treat customers with seriousness and professionalism" → until team confirms it's the customer's own tagline in the app

### Feb 10, 2026 — Invalid LinkedIn URL investigation

- Padmanabha: "Invalid LinkedIn URL" error is expected behavior if Mihir was on a non-profile/non-company page
- Regex for valid pages: https://remix-india.remixlabs.com/e/edit_l3/bomisco_linkedin_hubspot_clipper/home/out_16/out_13
    - Valid: `/in/{personId}` or `/company/{companyId}`
- **Vijay asks:** show the current page URL in the error message so user knows why it failed

### Feb 11, 2026 — URL shown in alert + V1/V2 .remix conflict fixed

- **Done:** Padmanabha redesigned "Invalid LinkedIn URL" alert to display the current page URL
- **New blocker found (blocking Bomisco):** workspace-deployed .remix file fails to load
    - Error: `cannot install a V1 .remix file over a V2 deployment`
    - Working: `https://storage.googleapis.com/rmx-content/draft/linkedin_hubspot_clipper.remix`
    - Not working: `https://agt.files.remix.app/0ry4DirCMQ/_rmx_files/extension_remix_files/linkedin_hubspot_clipper.remix`
- **Root cause (Gerd):** "Tools & Settings" export produces old V1 format; team should use `export_package` plugin or **workspace tools** to produce V2
- **Gerd:** "We should really take V1 exports out" (action item for Arvind)
- **Resolution:** Padmanabha used workspace tools → flow updated in Bomisco's workspace ✓

---

## Key Technical Notes

- **V1 vs V2 .remix:** Never export via "Tools & Settings" — it produces a V1 file that cannot be installed over a V2 deployment. Use `export_package` plugin or workspace tools.
- **LinkedIn extraction inconsistency:** LinkedIn sometimes serves different HTML for the same profile page; inconsistent clip results are not necessarily a Remix bug. No long-term fix decided.
- **Copilot UX overhaul:** Assigned to Arvind after Jan 31 post-mortem. Full scope: icons, sign-out, edit buttons, workspace setup UX.
- **Custom home app:** Workspaces can have a custom home app set so customers don't see Remix's internal workspace list.