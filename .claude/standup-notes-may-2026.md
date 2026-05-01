# Standup Notes — May 2026

**Coverage:** May 1, 2026
**Format:** Condensed bullets per standup. Google Doc link on each date for on-demand full transcript fetch.
**See also:** [standup-notes-apr-2026-3.md](./standup-notes-apr-2026-3.md) — Apr 27–30 | [standup-notes-apr-2026-2.md](./standup-notes-apr-2026-2.md) — Apr 13–24

---

## 2026-05-01 — [Google Doc](https://docs.google.com/document/d/1zgfdx1dE_Gh1LDDxGXXCk2f-4ffKc19ZhfZ6GKJvXuE)

- **Snowflake Summit (Jun 1-4, 4 weeks out)**: Pre-summit campaigns via LinkedIn / email / direct messaging. Jun 2 invitation-only happy hour near Moscone (partner co-hosted). Full attendee list
  acquired; segmenting for messaging tests. Suma awaiting logistics from Snowflake event managers.
- **Decision: homegrown tooling for Summit campaigns**: Use Vibecode landing-page component (Vijay: third-party tools look counter-intuitive when touting own capabilities). Off-the-shelf rejected.
- **RCS for Snowflake campaigns**: Unlikely for initial prospects (privacy + no phone numbers). Email + landing pages preferred. Separate RCS-oriented campaign still needed.
- **Twilio conference (next week)**: Mukund + Vijay + possibly John attending; learn Twilio's RCS pitch.
- **Lumber market angle**: Target mid-market companies with Lumber pattern from Summit attendee list. Lumber as case study.
- **Secondary sales angle**: Snowflake SMB sales group + solution engineers (CA SMB sales owner + SMB SE contact met at dinner). Reach out to AEs next week.
- **Dashboard assembly demo (John)**: New surface using Vibecode. Components communicate via app state + pub/sub. Accumulator tile manages facets + final SQL (service-agent configured, not visible in
  dashboard). Avoids screen-level state. Scope: small # facets + charts. Complex dashboards remain in builder. Validate against simpler 4 Lumber reports.
- **Chat client demo (Vijay)**: Rapid generation of chat client with embedded interactive vocabularies (cards, color pickers). Pure JS runtime → embeddable in Salesforce / Google Workspace / Zoom. No
  WASM, small size, good for Lightning Web Components.
- **Active chat-driven deals**: Starlight, King Power/Tequila, US SI bringing into shoe-retailer customer.
- **GTM strategy**: Partner-driven (Yelp/Twilio for SMB); avoid fragmented small-business market initially.
- **Domain focus**: Apartment management (Resides) — building out-of-the-box component vocabulary. India team to build domain variants → "content factory".
- **Didier's challenge**: Long history of impressive technical demos; need to convert to sales. Articulate clear value prop.
- **App auto-sync caveat**: Apps only auto-sync on desktop start (need restart). Documented in Studio how-to-guide plug-in (no images yet); content static-card-from-JSON.
