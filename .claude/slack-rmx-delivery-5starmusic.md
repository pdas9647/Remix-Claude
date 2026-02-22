# #rmx-delivery-5starmusic Slack Channel — Remix Labs

**Coverage:** Oct 30, 2025 – Feb 5, 2026
**Channel ID:** C09PP2CH97U
**Created by:** Reza Mohsin (Oct 30, 2025)
**Purpose:** Active delivery channel for 5 Star Music IVR + CRM build and go-live

---

## Key People

- **Mark** — 5 Star Music owner (customer)
- **Reza Mohsin** — CS/delivery lead
- **Mukund Ramaratnam** — Account manager / field visits to Mark's office
- **Suma N** — IVR recordings, Twilio setup, testing
- **Sirshendu (Riju)** — Built CRM app and DBP
- **Didier Prophete** — Platform support (Microsoft login, mobile bugs)
- **Wilber Claudio** — Android SMS relay device (outbound SMS gateway)

---

## Architecture Discussion (Oct 30)

Reza raised a critical design concern about the SMS delivery architecture:

- **Current state**: IVR calls Remix cloud agent → agent sends SMS *via Wilber's Android phone* (not Twilio)
- **Reza's concern**: This is not a production-grade architecture. Wants a proper design with Twilio SMS, retries, failure detection, monitoring logs, and potentially multiple Android devices for
  redundancy
- **Pattern required**: Either the customer workspace (`com_5starmusic`) or a central Remix outbound-SMS service connects to the IVR — with a clear upgrade path to swap in Twilio
- **Status**: Twilio SMS migration flagged as next step; short-term using Wilber's phone

---

## Build Progress (Oct 30 – Nov 10)

### [Oct 31] CRM App + DBP (Sirshendu)

- Built contact screen for IVR (non-overlay, link sent via SMS)
- CRM contact status model: **new** (default) → **read** (opened) → **closed** (manually marked by staff)
- Added search and filter for new/all contacts
- Added confirmation screen for deleting contacts
- **Removed HubSpot integration**
- Moved cloud DBs to prod workspace `Hafoe2ekMN` (John Dismore created it)
- Made Mark an admin in the workspace
- DBP uses `Top Level Scrolling Screen` card for mobile-optimized layout (width-constrained at wide screens)
- Links at go-live:
    - CRM: `business_profile_manager_5starmusic.remix`
    - DBP: https://remix.app/about/5starmusic
    - Builder (CRM): https://remix-india-beta.remixlabs.com/e/edit/business_profile_manager_5starmusic
    - Builder (DBP): https://remix-india-beta.remixlabs.com/e/edit/about_us_alt

### [Nov 4] Pre-launch Testing (Reza + Riju + Suma)

Action items assigned:

- **Sirshendu**: (1) display message time in local timezone, (2) validate email in contact form
- **Suma**: test IVR with US callers (IVR dropped international calls — no international SMS)
- **Reza**: video thumbnails intermittent (cleared by cache purge); mobile app fresh-install server error on some devices (iOS 10 XR, iOS 18.6.2) — transient, resolved after ~5 min, not pursued
- `share` button doesn't work on Firefox/Firefox forks (e.g. Zen browser) — confirmed working on Chrome/Safari

### [Nov 7] Microsoft Login Added (Didier)

- `_rmx_auth` updated with Microsoft login option (keeping Google) for 5 Star Music workspace `Hafoe2ekMN`
- Needed: Microsoft auth tested OK on Android; had initial errors that were investigated

---

## Go-Live (Nov 10–14, 2025)

### [Nov 10] Go-Live Check

- Reza asked platform team (John, Didier, Wilber) for blockers — none raised
- Didier: Microsoft login added; "can't think of any other issues"
- Twilio number: **(650) 855-2764**
- IVR greeting: static/noise at start of recording (Suma re-recorded; Twilio support unhelpful — noise appears to be introduced on the Twilio call, not in the source file)

### [Nov 12–13] Field Trip #2 (Mukund at Mark's office)

- Changed **default email app on Mark's Samsung** to Outlook (in-person fix; couldn't find setting remotely)
- Set **call forwarding** from main landline to 650 Twilio number — all calls now route to IVR
- Tested all 3 IVR options: Options 1 & 2 have disturbance at message start (Suma to re-record); Option 3 and main greeting sound fine
- Email sending from Remix app to contacts: working after Outlook set as default

### [Nov 14] Deemed Live

- Mukund: "Five Star Music is pretty much live"
- Mark's feature request: **push notification when contact form submitted** — Mukund told him it's possible, deferred to later

---

## Post-Launch Monitoring (Nov 17)

- Suma checked Twilio stats: **3 new inbound calls** since launch Friday; caller numbers visible in incoming log (useful for Mark)
- **No new contact form records** from those calls — uncertain if callers completed the form

---

## Post-Launch Updates (Jan–Feb 2026)

### [Jan 30] Customer Communication (Mukund → Suma + Reza)

- Mukund shared screenshots (content unknown — customer update or stats)

### [Feb 5] BPM .remix File Updated

- Mukund posted updated `business_profile_manager_5starmusic.remix`

---

## Channel Tone & Patterns

- Delivery-focused: field trips, in-person testing, tight customer coordination
- Mukund handles customer relationship; Suma handles IVR/Twilio; Riju builds
- Short tight messages; most detail in #bugbash cross-posts
- Reza sets architectural direction and go/no-go decisions