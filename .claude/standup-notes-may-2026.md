# Standup Notes — May 2026

**Coverage:** May 1–8, 2026
**Format:** Condensed bullets per standup. Google Doc link on each date for on-demand full transcript fetch.
**See also:** [standup-notes-apr-2026-3.md](./standup-notes-apr-2026-3.md) — Apr 27–30 | [standup-notes-apr-2026-2.md](./standup-notes-apr-2026-2.md) — Apr 13–24

---

## 2026-05-08 — [Google Doc](https://docs.google.com/document/d/1kkvAoC9LKuHckFTgyjQ3HPbYB0bo7oVTisvfPVp0no0)

- **Starlight + Lumber**: Both progressing toward live; neither called live yet.
- **Snowflake site awareness campaign**: Short blog piece on "analytic apps" → LinkedIn → Summit campaigns for lead gen.
- **RCS market analysis (Twilio/Studio conference)**: Three branded channels: RCS, Apple Messages for Business, WhatsApp for Business. Team has strong story vs. competitors.
- **Lumber token (Reza + Didier)**: Didier provided React wrapper + full React example for token swapping; Lumber team found it helpful. Final piece: expose company ID/tenant ID from Lumber token →
  internal token. Decision: Lumber team implements React code in portal first (test performance + CSS leakage), then token modification.
- **Lumber competitive talking points (Reza → Manish)**: 4 points re Metabase: pricing, Snowflake/Cortex capabilities, Remix platform nature, work substantially complete.
- **Lumber one-shot report**: Build with semantic layer within next week (if evaluation continues).
- **Service agent tooling deep dive (Monday)**: Vijay + John + Arvin + Reza + Mark; critical for RBAC + base query setup for Snowflake customers.
- **Google outreach**: Vijay + Wilbur to structure material for Google submission.
- **Snowflake submission**: Zero overlap with Lumber service-agent work (Chris confirmed).

---

## 2026-05-07 — [Google Doc](https://docs.google.com/document/d/1upJh9GHZCeOx09BYjCG9Y-wGGoNdBUYyypu-kZuNIKE)

- **Starlight onboarding complete**: Nat (from Starlight) onboarded; set up on desktop; sent first RCS messages via Starlight Twilio account. Still proof-of-concept — Twilio live-RCS approval pending.
- **Dashboard components (John)**: Reconfigured for Vijay's new page builder. Built ROI calculator + chart component. Chart AI config = next step. Bug: chart doesn't refresh data on facet change.
- **Survey flow as questions component (Mark)**: Integrating into assembly layer; required extensive debugging.
- **DB corruption / crash (Simon + Mark)**: Shared .remix file caused skyrocketing memory + system crash; Mark can't push fixes (polling issue). Simon to flag to Chris + Fred. Makes code collaboration
  impossible.
- **Twilio approval process (John + Wilber)**: Biggest RCS hurdle — finicky wording, consistent ToS/privacy, approval from Google + 3rd-party business verifier. Wilber resubmitting after compliance
  feedback from Suma + Starlight.
- **Web component multi-instance crash fix**: Multiple web components on same page can't share same app/DB. Fix: use `rmx-id` attribute to override OPFS key per component (UID scoping). Workaround
  until fix: two separate apps side-by-side.

---

## 2026-05-06 — [Google Doc](https://docs.google.com/document/d/174cA85F0SsG1rUYjzuZcrH34i--Isz2Zr7Z9SO04f2Y)

- **Grocery catalog demo (Padma)**: Cragle dataset + Hiku AI semantic search. List + detail view with pagination. Target: US Snowflake partner showing to retailers at Amazon retailer conference + Asia
  AP Pack conference. Issue: remote data freezes builder screen -- using 500-record local DB workaround. Mix code needed for filter generation (query tile limitation). Mukund: create demo video.
- **Future expansion**: Chat experience (ask about products --> mini catalog).
- **Marketing campaign (Sirshendu)**: Videos + website updates in progress.
- **Lumber portal integration**: No red flags (Chris + Didier). Awaiting Lumber ETA for end-to-end test. Didier to build React component for Lumber team to prevent delays.
- **Lumber token work**: Final piece = pass company ID/tenant ID from Lumber token --> Remix token. Separate API call + custom auth handler may be needed. Decision: test generic report (no company ID)
  first, then add tenant ID.
- **Twilio conference**: Mukund + team attending.

---

## 2026-05-05 — [Google Doc](https://docs.google.com/document/d/16Y9HRjLY7S4NdpMhNigdJAaPARJR0EXlbuh1VwVGfq4)

- **RCM2 (Go binary)**: Introducing alongside existing RCM for gradual migration. Google auth fixed (missing Base64 decode). Dual config files (config.mjs + config.json) coexist during transition.
  Simple MJS auto-converts to JSON; complex MJS stays manual.
- **RCM required features**: (1) Test-only dep edge (simplifies dep graph); (2) Branch pointer set only after build fully complete (prevents pulling from partial builds). Atomic upload: separate
  artifact upload from publish step.
- **Repo consolidation next**: Merge RCM2; merge group box PRs --> pull group box into mix-rs; move desktop --> products repo.
- **Environment URL cleanup needed**: Three ambiguous "base URL" vars. Need clear API prefix vs asset prefix. Client-side vars must NOT auto-transfer to Mixer agents. Follow-up required before
  changes (risk: breaks mobile + login flow).
- **RMX Remix web component (Didier)**: Now handles unified login flow for token-required apps. Removed L2 tile preview persistence (less data to server, better memory; downside: Project tab search
  loses tile previews).
- **Fred**: DB query optimizations landed last week. Working on: limit groupings returned; flatten old DB versions function.
- **Simon**: Runtime polishing for embedded use (JS-centric start + HTML web component). Added anonymous/authenticated group-of-screens feature. Macros for live binding updates in progress.
- **Tyler**: Desktop shortcuts still broken (Tory team unresponsive). 3 shortcut types can't coexist: text editing (cmd+Z), Monaco, app-level (cmd+A); current config excludes text input shortcuts.
  cmd+C/V don't work in dev tools.
- **AI-in-builder vision (Vijay)**: Three AI-assisted tile types: function, web component, WASM. Requires LSP + decoupling from Monaco. Arvind: prioritize L2 conversational AI ("speak a function into
  existence") over L3. Vijay: must solve both. Immediate: Vijay researches native Monaco alternatives; Arvind articulates L2 AI vision for Thursday.
- **LSP (Gerd)**: Feasible; unmerged PR exists; compiler communicates via WebSocket port.
- **Builder --> Git-like tree (Vijay + Gerd)**: Map builder node hierarchy to Git tree for unified sync + clean AI contributions.

---

## 2026-05-04 — No transcript

*(Gemini transcription failed — no content produced.)*

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
