# Customer Projects — Orderly, 5 Star Music, Bomisco, Opt Health, Tapped In

> Hub: [Customer Success Projects](https://www.notion.so/26f1d464528f80f3a6fffc1fe4cf294e)

---

## Orderly

> Source: [Orderly](https://www.notion.so/2781d464528f80e6b282fa5ce3050372)

### Context

- Early-stage startup providing concierge services to hotels
- One hotel in Mountain View as initial customer
- PRD (copy): [Google Doc](https://docs.google.com/document/d/1j5AroLkCIbaJ9QFZYhYWuAE-MP5PrFUvL9-j2vizUzs/edit?usp=drive_link)

### Workspace

- **Workspace ID:** `G4dLBeuE4V`
- **Workspace URL:** https://agt.remixlabs.com/ws/G4dLBeuE4V

### Scope (Phase 1)

1. **Web scraper** — Scrapes WebRez (property management system, golden source for bookings/reservations/guest status), posts to Orderly API
2. **Hotel front desk flow** — Trigger Orderly concierge flows (checkin/checkout), send messages to guests

### Architecture

- Remix workspace holds **no data** from WebRez or Orderly (except logs)
- One named user per hotel property
- Future: widgets, AI agentic flows for hotel guests

### Deliverables

1. Remix Chrome Extension flow (scraping + triggering)
2. Remix Admin App with monitoring

---

## 5 Star Music (Twilio IVR)

> Source: [Twilio and Remix work steps: IVR 5 Star](https://www.notion.so/29a1d464528f8043a346debe4f572cde)
> Kanban: [5 Star Music Kanban](https://www.notion.so/26a1d464528f8095b9b6e072efec37cd)

### Context

- Bay Area music school, 50-60 students, 2-3 teachers
- ~10 inbound calls/day from prospective students

### Workspace

- **Workspace ID:** `Hafoe2ekMN`

### Key URLs

- Contact form (sent via IVR SMS): https://remix.app/about/5starmusic/ivr_contact_form
- Digital Business Profile: https://remix.app/about/5starmusic
- .remix file (home screen flow): https://storage.googleapis.com/rmx-content/draft/business_profile_manager_5starmusic.remix

### Architecture

1. Business landline → call forwarding (Xfinity `*72`) → **Twilio number**
2. Twilio IVR (Studio Flow) plays voice recordings, routes callers
3. Caller presses option → Twilio makes **HTTP request to Remix cloud agent**
4. Agent sends **SMS with app clip link** (contact form) to caller
5. Staff reviews inbound requests in Remix flow (mobile app + web app)

### Twilio Setup Notes

- Remix owns Twilio account, creates sub-accounts per customer (Remix pays, invoices back)
- Phone number must be registered (text-enabled, takes a few days)
- Voice recordings: M4A → convert to MP3 → upload to Twilio assets
- IVR built in Twilio Studio
- Call forwarding: landline carrier forwards unconditionally to Twilio number
- **Twilio number:** (650) 855-2764
- **SMS delivery (current):** outbound SMS sent via Wilber's Android phone (not Twilio); Twilio SMS migration flagged as next step
- **Call forwarding set:** Xfinity landline → Twilio 650 number (set in-person Nov 13)
- **Default email on Mark's Samsung must be Outlook** for the "send email to contact" flow to work

### CRM App (built by Sirshendu)

- Contact status: **new** (default) → **read** (auto on open) → **closed** (manually marked)
- Search and filter by new/all contacts; confirmation screen on delete
- HubSpot integration removed; cloud DBs on workspace `Hafoe2ekMN` (prod)
- Auth: Google + Microsoft login both enabled

### Delivery Status

- **Went live:** Nov 13–14, 2025 (Mukund field trip to Mark's office)
- **Pending feature request:** push notification when contact form submitted (Mark asked; deferred)
- **Post-launch:** 3 inbound calls in first week; caller numbers visible in Twilio log; low contact form completion rate initially

---

## Bomisco

> Source: [Bomisco](https://www.notion.so/2bc1d464528f806f94c9ce622bddacab)

### Context

- Uses Remix LinkedIn clipper to post data into HubSpot as contacts/companies
- Same pattern as "Remix own use" (Remix's internal LinkedIn-to-HubSpot flow)

### Workspace

- **Workspace ID:** `0ry4DirCMQ` (new)
- Old (corrupted): `ET8fjwrW2h`
- **Workspace URL:** https://agt.remixlabs.com/ws/0ry4DirCMQ

### Scope (Phase 1)

Clip information from LinkedIn, edit/correct, post to HubSpot as contact or company.

### Clipper Behaviour

- Page detection logic: shows "clip company", "clip contact", or "please navigate to a LinkedIn page" depending on current page
- Review and save flow before posting to HubSpot
- Duplicate detection needed (fields TBD)

### HubSpot Field Mappings

**Contact properties:**
First Name, Last Name, Company Name, Job Title, Email, Mobile Phone Number, City, State/Region, Country/Region, Lead Status, Contact Owner, LinkedIn Bio

**Company properties:**
Company Name, Company Domain Name, Industry, Phone Number, Type, City, State/Region, Country/Region, Time Zone, Description

### Open Questions (from notes)

- What unique field does clipper use for duplicate detection?
- Should we capture skills, experience, education?
- Should we save the markdown from LinkedIn?
- Research LinkedIn Sales Navigator integration

---

## Opt Health

> Source: [There is no failure. Only feedback.](https://www.notion.so/1b41d464528f80d9bc61e166f3db36af) (ATS 2025 field report)
> Contact: Chris Collins (Customer Success)
> Website: https://getopt.com/

### Context

- ~15-person startup providing personalized wellness programs for law enforcement, fire departments, military, and first-responders
- Attends events within the law enforcement / first-responder community to improve lead-capture and engagement
- Interested in Remix for lead-capture + DBP (Superapp framework)

### Field Trial — ATS 2025 (California Police Chiefs Association, Riverside CA, March 2025)

**Solution deployed:** Remix booth-hub flow + Digital Business Profile in Superapp framework.

**Booth flow:** Visitor → badge info captured (name etc.) → checked against pre-populated lead list → new lead added → customized Clip journey provided → notes captured → additional actions (sign
waiver for B12 shot, book blood draw) → push notification sent to schedule follow-up meeting

**Visitor flow:** Customized Clip journey → sign waiver form → B12 shot if needed → push notification for follow-up

#### The Good

1. Content prep (videos, questionnaires, lead list, blood draw calendar) solid
2. Opt Health was flexible and willing to experiment
3. Most visitors followed the DBP/Clip through despite UX friction
4. QR codes with custom journeys worked
5. Capture badge shortcut on iOS/iPadOS worked well (happy path: search known name in lead list)
6. Digital signature for waiver worked nicely — tested on iPad
7. 100% of visitors had iOS devices
8. Booth app sync flow worked well overall
9. Push notifications worked — some visitors responded to follow-up meeting requests and booked via the calendar screen pushed at end-of-day

#### The Bad

1. Audience mismatch: most were police chiefs evaluating agency adoption, not SWAT officers scheduling blood draws — pre-built blood-draw scheduling flow went unused
2. Booth staff not prepared — main contact had downloaded Remix, logged in, seen a demo, but could not run the flow independently. Too many steps, insufficient familiarity
3. **iPad >>> iPhone** for booth use: QR code scanning, waiver signing (screen size, keyboard). "Super weird to hand your phone to someone to sign"
4. **Caching bug**: first 2 QR code shares served an older xcaliber clip from a previous trial (container app had cached it). Had to delete + reinstall container app to get latest flows
5. **Search UX issues**: searching "Ryan K" required full last name; typing pre-selects entire search term so backspace deletes everything instead of last character
6. Waiver form issues: had to re-enter visitor name (should transfer from lead detail), date-picker required too many interactions (most users born 60s/70s/80s), accepted signature did not auto-save (
   some lost), form not iPad-optimised
7. App Clip experience too slow and choppy for booth context — 5–10 second load time; >1/3 of visitors pocketed phone before content loaded
8. Remix DXP banner on App Clip invoke: 80%+ of users ready to tap it; must be removed or bypassed
9. NFC tags not viable in busy B2B booth setting (too many steps, no cycles)
10. Add-lead form too long; saved lead did not appear in list until manual re-sync (Drawer → Update sync flows)

#### The Ugly

1. Booth app on iPhone crashed into unrecoverable state: "OnInitDataForVmServer Error when communicating with server" — Home button looped, had to delete flow; flow did not auto-resync to device,
   required John to send a `.remix` file to recover
2. Prep checklist (booth app, clip, onboarding, experience variables) too complex for typical event/booth staff persona; multiple conflicting QR codes from other collateral at booth

#### Key Learnings

- Must include an **incentive** for visitors to go through the DBP/Clip (raffle, waiver, coupon) — otherwise no motivation
- Full experience needs redesign for "wow factor": production → booth → visitor flow
- iPad is the right device for booth operations, not phone
- App Clip loading and Remix DXP banner are critical UX blockers for event use cases

**Quote from Chris Collins (Opt Health):** *"I think this has the potential to give our leads and customers a really cool and differentiated experience"* and *"Was a major help having you assist with
the Remix integration. Super excited to build this out further and refine the experience."*

---

## Tapped In (External Developer / Partner)

> Source: [Tapped In Interactions](https://www.notion.so/10c1d464528f804bb97fd9f061d7fbf2)
> Sub-page: [Push messaging](https://www.notion.so/19f1d464528f80a781c5cbaa4e2412d3)

### Context

- 3 students from University of Alabama built a student event platform using Remix Studio (summer 2024, ~3 months)
- Previously had a Wordpress prototype (~18 months prior)
- Business objective: all-in-one student platform, starting with events
- Revenue model: local business ads and deals targeted at students/groups
- Events scoped by audience: group-only, university-wide (.edu verification), or public

### Key Features Demonstrated

- Apple Wallet integration: save event QR code + user profile to Apple Wallet
- Admin view for event creation with audience targeting
- Complex cloud agent server screens in Remix Studio
- Tooling for pushing app to TestFlight

### Remix Suggestions (from demo review)

- Trigger Apple Wallet when approaching event venue (location-based)
- Google Maps integration (tap to open)
- App Clips for events
- Chrome Extension for scraping events from websites/university calendars
- Live Widgets
- Federated Search to pull newer library components

### Push Messaging (Platform Capability)

> Source: [Push messaging](https://www.notion.so/19f1d464528f80a781c5cbaa4e2412d3)
> GitHub: `remixlabs/turntable` repo — [webapp push messaging docs](https://github.com/remixlabs/turntable/tree/main/webapp#push-messaging--browser-notifications)

Browser-based push notifications using **Google Firebase**. Supports foreground + background messaging. Requires explicit user permission.

| Platform                       | Browser                              | Status                                         |
|--------------------------------|--------------------------------------|------------------------------------------------|
| iOS                            | Safari ("added to desktop" required) | Works (no pure background data notifications?) |
| Android, Windows, macOS, Linux | Chrome, Firefox                      | Works                                          |
| Others                         | —                                    | Untested                                       |

**Open TODOs:** parameterize Firebase config, test more OS/browser combos, other messaging platforms.

Flutter/Native app push: not yet documented.

---

## All Workspace IDs (Quick Reference)

| Customer         | Workspace ID |
|------------------|--------------|
| Funda            | `sEt2qxPydL` |
| Orderly          | `G4dLBeuE4V` |
| 5 Star Music     | `Hafoe2ekMN` |
| Bomisco          | `0ry4DirCMQ` |
| Lumber           | `iaEj4QYboi` |
| Remix (internal) | `6xX25jCW6I` |
