# Standup Notes — Mar 2026

**Coverage:** Mar 2–31, 2026
**Format:** Condensed bullets per standup. Google Doc on each date for full transcript.

---

## 2026-03-31 — [Google Doc](https://docs.google.com/document/d/15PkqA5khj_bQE0kHwnGI3Kh8-fO19C-uSSu9wxlcbZQ)

- **Release branching decision (major)**: Stable branch = bug fixes only; dev branch = new features. Solves rolling-release instability. Benedikt drafts monorepo plan, Chris drafts multi-repo plan;
  compare next week.
- **Monthly versioning**: Minor version bump per month; separate from compiler semantic versioning (ABI changes). Weekly dev/beta cadence continues.
- **`file.register` + .remix HTML runtime (Simon)**: `file.register` returns full file object. New HTML page runs .remix URL via `remix.app` runtime; recommended embedding vs iframe.
- **Extension CI (Tyler)**: CI auth flow with Chrome Web Store; extension back on release cycle. Didier: artifact viewer menu hidden; state machine CSS fixed.
- **Lumber delivery debate**: Table gaps: column pinning, grouping, sort, totals row. Reza: 50–75% of product is onboarding/ACL/lifecycle — platform alone is not a delivery. Arvin: onboarding risk
  high.

---

## 2026-03-30 — [Google Doc](https://docs.google.com/document/d/17wyn1YJ7I2v4WxcfPRZnsvFrry-LtcSi3ki5mDSTRz0)

- **OPFS fix (Benedikt)**: Safari failure resolved (sync interface in worker). Decision: remove IndexDB→OPFS migration — wiped sticky settings easily recreatable.
- **Platform freeze (Vijay)**: Frozen ~3 months (through ~Jun 2026), bug fixes only. Exceptions: fix amplification + migrate O server → Mixer.
- **Infra**: GoDaddy billing caused `remixabs.com` outage → migrating to Squarespace. `mixcore.wasm` blocked in Snowflake SPCS (data URL) → browser-side FFI broken, server-side fine.
- **Lumber demo (Mar 30)**: Greg, Oleg, Prinad, Manish attended; supportive. Next: commit delivery date; iframe integration in Lumber test env. Lumber desktop = candidate for Release 1.0.
- **Mix Center docs viewer (Gerd)**: Machine-readable JSON stdlib plugin; types auto-generated, content handwritten. Replaces Notion stdlib docs; AI can generate Mix examples from it.

---

## 2026-03-27 — [Google Doc](https://docs.google.com/document/d/1K6utfutblQ20YU-VYzNjYjps55GBoKKAjqSDKRdCzoo)

- **Starite kickoff**: Upcoming. Bug: screen navigates back on pub/sub in non-top-of-stack screen (Gerd fixing). Mixer also needs raw URL with exact path/query params for service agent calls.
- **Lumber blockers**: Catalog publish/refresh, CTE pattern, view support, SQL column config, bubble/bar chart, CSV download. Catalog caching bug: R2 missing cache-control headers — Wilbur to fix in
  Mixer upload code.
- **GTM maturity framework (Mukund)**: 4 buckets — early exploration, product candidate, early GTM, scale GTM. Criteria: ICP clarity, value story, assets, pricing, CS readiness.
- **Platform positioning**: "Context engine for every customer interaction" — AI infrastructure, AI apps, Remix Studio. Lumber ICP gap: LoB vs IT buyer unclear; full-scale push blocked.

---

## 2026-03-26 — [Google Doc](https://docs.google.com/document/d/1qPLY3aMFLKWhUCR5ejwVB3ax2b1dcWz9sDoAbpBt8O8)

- **Preferences (John)**: Three-tier access — user, workspace admin, workspace-wide defaults. Custom experience via .remix file.
- **System plugin rethink**: Move registration from Elm → sign-in workspace level (internal vs customer-facing). Didier: sync app whose plugins surface as regular local plugins.
- **Lumber table sorting (Tyler)**: Options — (1) data source → server-side sort via agent; (2) `reset data`, manage state externally; (3) external facet picker modifies SQL (used for demo). Steel
  thread confirmed.
- **RCS admin app (Wilbur)**: Conversation list + CRM hydration, live scheduling from DB, events logged. Android confirmed. Bug: Mixer no POST for form URL encoded.

---

## 2026-03-25 — [Google Doc](https://docs.google.com/document/d/1PtQ7AYZGw5awQd9vkMagwLfEZr1F0e3ifZvXr9QXMoA)

- **Lumber India demo (Mar 26)**: Hand off Remix Desktop using dummy Snowflake data. SPCS builder blocked — plugins not working in SPCS env (Chris investigating).
- **Customer 360 generalization (Sirshendu)**: Lumber work generalized for Customer 360 + Remix internal data; hardcoded workspace names/auth → constants.

---

## 2026-03-24 — [Google Doc](https://docs.google.com/document/d/1tSlFgdANisQFqqsUxugZAY4rp1idcVJJhs_jpyl3r_4)

- **Monthly release cadence (Chris)**: Every 4th beta-to-prod = public release (1.0, 1.1…); accumulates release notes. Incremental repo consolidation: mix-rs + group-box first (looping dep).
- **Web component updates (Tyler)**: `data-RMX-C-ID` attribute enables anchors in cards. Attribute data passing → `arguments` manifest syntax (fixes race condition).
- **File node + FFI (Simon / Benedikt)**: File node returns JSON directly. FFI command execution assigned to Benedikt — restricted host commands from Mix, user confirmation required.

---

## 2026-03-19 — [Google Doc](https://docs.google.com/document/d/1iBzIjYrRmrjGOL1Obn54VSpa1yTSpdjRW3PofN473iw)

- **Data source + alias mgmt (John)**: Source persistence works at L2, inconsistent at L1; user-centric storage preferred. Alias flow shows all DB aliases; delete component published to library.
- **RCS focus shift**: Calendar booking for Starlight (embed + RCS). Demo hacks being cleaned up. FB Messenger + WhatsApp planned post-scheduling. Starlight RCS proposal finalizing (Mukund).

---

## 2026-03-18 — [Google Doc](https://docs.google.com/document/d/1K5g3NwkOdGzsN82i34v-xoMjlwIMAhr-RcrJzbo5FYU)

- **India team delivers clean Lumber tables (Reza)**: Hourly-populated reporting tables in prod. Snowflake schema call Fri with Anand/Trinad on payroll structure.
- **Bugs**: Compiler critical fix nearly done. Screen save with table ~5 sec — DB likely source.

---

## 2026-03-17 (Afternoon) — [Google Doc](https://docs.google.com/document/d/1C6AQ6JLLn35ZiCSQwgXouTUaOPFTgXE6HPLCeAVn2fE)

- **Screenshot + style bugs (Didier / Benedikt)**: Screenshot library replaced (ms-level). Style sync bug: styles disappear on "rebuild and make."
- **Table component (Arvind)**: Nearing Lumber library readiness — sticky totals row, sticky left cols, grouped headers in progress. Requirements: grouped cols, sticky totals, sort, CSV/Excel/PDF. TUI
  lacks event dispatching for persistence.

---

## 2026-03-16 — [Google Doc](https://docs.google.com/document/d/1LDofaFupgzEcJoiJql2NVvYzFoveQFCTO6Y7j-T6p10)

- **Command FFI (Benedikt)**: Started — restricted host commands from Mix. OPFS PR ready (may break DBs, file install, caching).
- **CI + Snowflake native app (Chris)**: CI design for extension done. Snowflake native app: customer data access working.
- **Agent tooling (Gerd + John)**: Deployment plugin v1 (deploy library agents to workspace). Alias plugin — select base agent + fixed params. Merging into standard toolkit.

---

## 2026-03-13 — [Google Doc](https://docs.google.com/document/d/1s7gZWM2_uN3PoRPTbHRqfQtPlz7nFAK0F99VTi712qA)

- **Sales (Mukund)**: Starlight + US services company progressing. Zendesk partnership moving. Snowflake submission goal next week. Deal tracking: quadrant chart in HubSpot, auto-Slacked from ~Mar 20.
- **RCS + positioning**: Scheduling flow praised by USG Global, Starlight, Zendesk. Website refresh: "context-aware experiences." Lumber arch: AI → Snowflake queries → reports → builder → .remix →
  portal.

---

## 2026-03-12 — [Google Doc](https://docs.google.com/document/d/19I6hu6Iykg80W2ZkjkrWefctkmzsH4OXmMkd1_BeO2o)

- **USG Global demo (John)**: Multi-billion $ consulting SI; Thai duty-free store use case.
- **"Experiences" architecture (Vijay + John)**: AI dispatches to registered .remix experiences across channels (chat, RCS, landing pages). Simple dispatch model sufficient.
- **Twilio RCS (Wilbur)**: Web view in SMS — e-commerce + meeting booking vs Google Calendar. WhatsApp parity. Twilio UX gap = Remix opportunity.

---

## 2026-03-11 — [Google Doc](https://docs.google.com/document/d/1f7n5z9qnfsWK0UqU3ScqqPfq26k8J23gR6D_RhbErbc)

- **Lumber 5-report scope**: Oleg documented 5 proof-point reports: facets, downloading, single facet → bar chart + table, user-friendly headers. Charts: bar + scatter.
- **Client expectations**: Client owns Snowflake schema + ETL (~1–2 months). Team to deliver phased scope + hard dates. Positioning: "report composition + publishing product."

---

## 2026-03-10 — [Google Doc](https://docs.google.com/document/d/1oAgvK2ZhSiWqoEVM6Eq26PZEgSZDhbknB6DYxk7N9-U)

- **Docs overhaul (Gerd)**: Notion docs obsolete. Custom processor → JSON → Markdown; enables LSP + AI training. Doc viewer broken — sprint priority.
- **Engineering process**: GitHub issue required for new work. "Sprint zero/one" labels. "Ready to pick up" = prioritized queue; backlog → "icebox."
- **Platform updates**: Web component PRs: method args + file node content binding. Table primitives merged (Tyler). Builder: node swapping at L1; duplicate dialog; synced prefs (Didier).

---

## 2026-03-05 — [Google Doc](https://docs.google.com/document/d/1NQONGS79R6Uy0vcWyUpByEX71fG9J-VLrJfmUAtSNpw)

- **Preferences UI (John + Didier)**: JSON name-value prefs auto-synced to builder/mobile/extension.
- **HubSpot split 3 apps (Wilbur)**: (1) web for service agent aliases, (2) mobile for widget records, (3) service agent project.
- **2026 strategy (Vijay)**: Add value to HubSpot, Zendesk, Snowflake users; use orgs as channels. Lumber = pattern to implement correctly to unlock others (Mukund).

---

## 2026-03-04 — [Google Doc](https://docs.google.com/document/d/1j9U40Wg4YT0mOEHUGy3dQdjry9UoKAodAGLYsc4HnlM)

- **Lumber PM reset (Didier)**: Client treated Remix as shrink-wrapped; no schema/ETL PM. Phased delivery agreed. Oleg (not Anand) now owns requirements; walks through first 3–4 reports with John +
  Arvind + Vijay.
- **Internal productization**: Productize widgets/facets/charts on HubSpot data — platform vs consulting differentiation, doubles as demo. Lumber facets (Sirshendu): Snowflake-backed + static picker.

---

## 2026-03-03 — [Google Doc](https://docs.google.com/document/d/1Ehuv1xAFGWOuKHECLSudHKfGZOF_KapKkNo95P0dyC0)

- **Desktop for Lumber**: Target India QA end-of-day. Arrow key Tauri bug — JS keydown workaround at text boundaries.
- **Env overhaul (Gerd)**: AMP-era vars → two clean vars (backend/Mixer URL + assets URL). Themes deployed: style collections, auto-propagate, no manual `make`.
- **Core sync agents**: Manual one-click update by workspace admin for now; management app shows out-of-sync apps.
- **Table primitives (Tyler + Arvind)**: Any subtree must be list-convertible → piped into primitive — essential for data-driven tables via loops.

---

## 2026-03-02 — [Google Doc](https://docs.google.com/document/d/1UIeN8IGXMnsnqmL3IC1laqEKFkhNistbkVEbq8olrz4)

- **Desktop + Mixer fixes (Benedikt)**: Configurable HTTP port merged; custom-protocol URLs now valid HTTP externally; E10 check prevents unnecessary reinstalls.
- **MCP bridge revamp (Benedikt)**: Removed auto-start; Remix must be open first for MCP tools.
- **Subprocess/shell (Vijay)**: Proposed for CLI access (Gemini CLI, cloud code). Security model needs design; no decision yet. Lumber EKS template built for deployed Mixer on Kubernetes (Chris).
- **Parser refactoring for LSP (Oliver)**: Separating type creation from syntax parsing. LSP server to work with any editor, not custom web component.
