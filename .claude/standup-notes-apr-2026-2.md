# Standup Notes — Apr 2026 (Part 2)

**Coverage:** Apr 13–17, 2026
**Format:** Condensed bullets per standup. Google Doc link on each date for on-demand full transcript fetch.
**See also:** [standup-notes-apr-2026.md](./standup-notes-apr-2026.md) — Apr 1–10

---

## 2026-04-17 — [Google Doc](https://docs.google.com/document/d/1QjMnRPMT4cZScG6GVZXHsrVO7CJ97kNM3y83_JYde5g)

- **Lumber auth clarified**: App not anonymous — use existing Dscope token from web portal. Backend verifies tenant ID in token against org ID from web component; tenant ID = remix token in mix env.
- **Desktop branch switch**: Switching dev → prod branch preserves all local data (DB + filesystem). Same applies for workspace switches.
- **Snowflake listing**: Converted to entirely paid model for direct consumption; completing today.
- **Snowflake Summit (Jun 1–4)**: Exhibitor in Startup Hub. Pre-Summit: outreach + LinkedIn ads + networking event. Lumber as case study candidate. NDA with SI partner "US Systems" — 3–4 potential
  client discussions; partner also interested in RCS.
- **RCS strategy (from Google partner engineering lead)**: 3 trends: (1) AI collapsing build costs → personalized journeys economically viable with Remix; (2) RCS channel readiness + Apple iOS support
  confirmed; (3) data infrastructure readiness → context from data lakes for personalized delivery.
- **RCS market**: Experience layer across channels (not exclusively RCS). Primary targets: B2C SMB + B2B mid-market. Large B2C brands already dominate current RCS noise. AI execution: embeddings match
  intent → correct experience without complex agents.
- **Lumber next steps**: Call needed — Chris + John/Arvin to clarify where to post remix docs + embed URL in Lumberfi web portal.

---

## 2026-04-16 — [Google Doc](https://docs.google.com/document/d/1AudXuJghmRZJF9ogVrdkL4SIAvcO7WcquYFzPqMW-E0)

- **Starlight workspace**: Set up rapidly with new tooling (Wilber). Scheduling flow demo: conversation history, faster queries, visual trimming of confirmation data. Bug: appointment time shown in
  24h format.
- **Starlight blocker**: Twilio RCS approval requires updated privacy policy + terms of service with opt-out instructions. Mukund to walk Starlight through process.
- **"Menu" experience picker (Vijay)**: Keyword triggers listing of all available experiences → scale testing without manual operator + simplified demo flow.
- **RCS→SMS fallback**: Confirmed working. Missing piece: URL shortener for high-volume SMS (options: Cloudflare worker or short prefix per workspace).
- **AI layer proposal**: Embeddings-based intent matching → route to correct experience. LLM autogeneration of typical intent questions. Implement while Twilio registration is pending. Vijay + John +
  Wilber to sync.
- **Campaign management (future)**: Journey-style view for cross-message activities; unified infra deploys same experience across portal, RCS, chat.

---

## 2026-04-15 — [Google Doc](https://docs.google.com/document/d/1AHHttU-9VWT9c8QFg3MEiJC2MvEh_-XUzqRSrI6C6LE)

- **Lumber client confusion**: Anand expected desktop handoff; client postponed download; broader team demo moved to Mon/Tue. Worker comp report shown (full job costing skipped — configurator issue).
- **Critical path risk**: Client extended timeline concerns → Remix removed from critical path. Needs concrete scope + committed delivery date to re-integrate.
- **Target**: Stable externally-ready Remix Desktop release by Friday (Apr 18).
- **Auth challenge**: Desktop needs native Dscope OAuth flow (not browser) to connect to client's self-hosted Mixer. Standard browser flow unavailable in Tauri. Also benefits Snowflake project (
  Chris).
- **Windows crash**: L2-only crash. Didier to investigate + fix.
- **Fred**: Subquery performance fix PR — many old versions in nested query causing slow execution.
- **Lumberfi infra**: API server (`qadmin.lumberfi.com/api`) over HTTPS port 443; no websockets on agent server.

---

## 2026-04-14 — [Google Doc](https://docs.google.com/document/d/1wMg6iaPObacyJvy8s6PYB_iHsI25flW8-3KR1-AQ6nk)

- **2-layer repo model (from Kurt's concept)**: Core repo (mix-rs + GroupBox + Turntable + Protoquery) + Product repo (mobile + desktop + Remix). Core independent at artifact level. Backport:
  cherry-pick on Core beta → auto-PR in Product. Chris to write concrete proposal.
- **Starlight priority**: Requires Mixer self-hosting + Dscope as auth server → driving self-host packaging + Mixer-as-auth-server work.
- **Web component workflow debate**: Registering web components + WASM too cumbersome (manual JSON manifest edit + re-upload after JS upload). Follow-up session planned. Smooth interim process first,
  then native builder improvement.
- **Export plugin clarification**: Old "export plugin" = .remix file only; "workspace tools plugin" = full export + file upload. Plan: remove old plugin, add direct download button for RMX packages.
- **Didier**: Extract-from-right node creation + direct manipulation L2 (drag facet onto drop zone to bind; dotted lines on hover only).
- **Simon**: Removing RMX runtime from builder (memory footprint); converting runtime code to TypeScript; L0 macro for repo plugin (create new app).
- **Lumber**: First session pushed to tomorrow using AG workspace + Snowflake instance. Arvin completing vertical bar chart configurator. Padma: file upload issue (possibly Cloudflare upload headers).

---

## 2026-04-13 — [Google Doc](https://docs.google.com/document/d/1wI73EF3Ug2pdWEKSg787ZE3uIBcqa592Xat4miO54lU)

- **Monorepo decision (incremental)**: Merge 2–3 repos as trial + build selective CI. Simon + Gerd opposed — prefer fixing circularities + reducing build times. Chris: simulating a monorepo builds
  complexity developers constantly hit.
- **Multi-repo pain**: 8 CI workflows to propagate core updates; E2E testing requires PRs across all component repos; backporting a 4-repo fix across beta branches is messy; partial cherry-pick state
  dangerous.
- **Benedikt**: Large PR consolidating mix core JS/WASM dynamic deployment. Created doc listing front-end surface use cases for testing.
- **Snowflake submission**: Blocked by open Snowflake support case re: org permissions. Chris sketching Mixer self-host docs for AWS.
- **Fred**: Bug fixes + Windows build + DB tool in mix-rs.
- **Oliver**: 250 tests for AST printers (expressions + phrases).
- **Reza**: Publishing issues for 4 Lumber reports. Question: deliver Remix Desktop via email DMG or hosted download?

---
