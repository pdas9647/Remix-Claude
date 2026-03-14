# Standup Notes — Mar 2026 (Week 2)

**Coverage:** Mar 10–13, 2026
**Format:** Condensed bullets per standup. Google Doc link on each date for on-demand full transcript fetch.
**See also:** [standup-notes-mar-2026-1.md](./standup-notes-mar-2026-1.md) — Mar 2–5

---

## 2026-03-13 — [Google Doc](https://docs.google.com/document/d/1s7gZWM2_uN3PoRPTbHRqfQtPlz7nFAK0F99VTi712qA)

- **Sales pipeline progress (Mukund)**: Cycles with Starlight (RCS messaging) and US (services company) progressing well. Zendesk partnership conversations moving forward. Snowflake submissions proceeding, goal to submit next week. All current engagements transitioning to commercial discussions — pricing/scope proposals needed.
- **Lumber = largest current go-live effort**: Other customer efforts remain in sales motion phase. Direct deals: Starlight. Partner-type: US, IP agent AI.
- **Deal visualization decision**: Quadrant chart to categorize/track deals. Will leverage HubSpot with quadrant annotations and notes per customer. Made available via Slack automatically, starting ~2026-03-20.
- **Website refresh**: New messaging emphasizes building "context-aware experiences." Design expected to change slightly but core messaging is set.
- **RCS demos well received externally**: External demos more polished than internal versions. Full scheduling flow via RCS particularly well-received by USG Global, Starlight, Zendesk contacts. Zendesk rep saw immediate applicability after viewing demo video.
- **Web-based Remix board tool (Vijay)**: Fully web-based, cards in standardized JSON card format (not HTML) with data binding layer. Multi-format export (PPTX, HTML) rendered by Rust — no browser in flow. Use cases: landing pages, customer case studies, sales dossiers auto-generated from CRM data. Runs Rust Wasm (no download required). Also demonstrated on native Swift app for mobile widgets. CMOs have expressed high interest in rapid landing page creation.
- **Node counter zero bug fix (Simon)**: Removing stage 1 codegen for desktop caused stage 2 to be first encounter of constants → counter starts at 0. Fix: compute only node validity instead of recomputing full RTI. PR submitted.
- **Constants in app meta debate (Didier vs Simon)**: Didier proposed materializing constants in app meta (like styles) to avoid compile/import issues during codegen. Simon concerned about inflating runtime app meta size (e.g., RMX Tailwind already replicated in every runtime app meta). Current behavior: full runtime app meta is created for self-contained distribution — `make` creates stable copy unaffected by subsequent builder changes.
- **Desktop screenshot challenge (Didier)**: Embedded Safari view + mixer's string prefix format for external blob objects makes screenshots difficult. Simon suggested Tauri plugins for screenshots (similar approach used for copy-paste).
- **Lumber implementation architecture (Didier)**: Desktop builder + runtime. Users query Snowflake via AI → generate reports/charts in runtime → copy/paste cards into builder to compose dashboards → produce .remix file → render in client portal.

---

## 2026-03-12 — [Google Doc](https://docs.google.com/document/d/19I6hu6Iykg80W2ZkjkrWefctkmzsH4OXmMkd1_BeO2o)

- **USG Global demo (John Dismore)**: Product demo for USG Global, a multi-billion dollar consulting/systems integrator (comparable to Accenture/Deloitte). Thai duty-free store use case with Wolver chat app — product comparison, store finder flows. Scheduled for noon. Recorded self-service version would be valuable for partner to share with their customers.
- **"Experiences" architecture (Vijay + John)**: AI maps user requests to invocable "experiences" — RMX Remix files provisioned across channels (chat, RCS, landing pages). Experiences registered with descriptions so AI knows what to show. Simple models sufficient (even "Apple Intelligence") — primary AI role is dispatch/mapping, not content generation. No canned scripts; standard app against standard service agent.
- **Twilio RCS integration demo (Wilbur Claudio)**: Interactive web view inside SMS app. E-commerce catalog example opens tequila catalog on "shop now" tap. Meeting booking experience fully functional against Google Calendar. Web view size is controllable. Same functionality available on WhatsApp. Webhook logs user interactions.
- **Twilio platform opportunity**: Poor UX and documentation on Twilio side. Web view config requires API (content builder doesn't support it) — identified as opportunity for Remix to add value on top of Twilio layer.
- **Lumber facets + pagination (Mark)**: Implemented pagination and sorting logic for CT-based query set. Component outputs page limit + offset to query. Arvind working on design changes using existing table UI alongside this logic.
- **PubSub bug**: Known bug requiring refreshing the log view. Affects runtime. Full repro to be created post-demo.
- **GTM strategy — experience vocabularies**: Build up large vocabulary of domain-specific experiences → take to market. Exploring partnerships with Twilio, Zendesk Sunshine Conversations, Broadway. Deciding between direct sales vs partnership approach.

---

## 2026-03-11 — [Google Doc](https://docs.google.com/document/d/1f7n5z9qnfsWK0UqU3ScqqPfq26k8J23gR6D_RhbErbc)

- **Lumber 5-report delivery scope**: Client PM (Oleg) documented 5 specific reports as proof points. Reports demonstrate: facets, selection criteria, downloading, single facet driving multiple components (bar chart + table), adjusting technical DB fields to user-friendly column headers. Two chart types needed: bar and scatter.
- **Component publishing workflow**: Build configurator UI around component → configure via friendly interface → clipboard → paste into drop zone on Studio screen → one-click publish to preview bucket. Process "almost there" per Arvind.
- **Client expectations clarified (Reza + Vijay)**: No imminent go/no-go decision. Client now understands customization is on top of platform. Client acknowledges Snowflake schema + ETL is their responsibility (~1-2 months). Team must return to client with scope and timeline for delivery phases immediately.
- **Product positioning**: Team currently positioned as "report composition and publishing product" — not full development platform. If team avoids training client on Studio, must provide full tooling for publishing + lifecycle management (schema differences between environments, testing, authorization).
- **Lifecycle management = significant scope**: Handling schema differences, publishing, testing, authorization across environments represents major portion of overall requirements.
- **Delivery phasing strategy**: Provide hard dates and scope. Include "should" category low-hanging fruit items. Leverage existing relationship to shape phasing even if cycle is 3 weeks.

---

## 2026-03-10 — [Google Doc](https://docs.google.com/document/d/1oAgvK2ZhSiWqoEVM6Eq26PZEgSZDhbknB6DYxk7N9-U)

- **Documentation format overhaul (Gerd)**: Existing Notion docs confirmed obsolete. New docs use custom format with processor that adds typing info → JSON output → easily converted to Markdown. Enables LSP integration and AI training (successful experiments training AI to generate functions from machine-readable docs). Doc viewer app broken — fix is current sprint priority.
- **LSP status (Oliver)**: Refactoring parser to separate type creation from pure syntax parsing. Key step: switch LSP messaging from WebSocket to standard browser messages (compiler runs in browser).
- **Engineering process decision**: Creating GitHub issue is standard practice for new work. Immediate bug fixes can proceed via PR without issue. Bare PRs auto-appear on the board. Sprint labeling: "sprint zero" and "sprint one" labels added across all repos. "Ready to pick up" column = prioritized queue. Old backlog renamed to "icebox."
- **File upload + web component improvements (Simon)**: Bug fix for make agent error surfacing when VM ID already in use. Restored drag-and-drop file upload for desktop + plugins. Two content PRs: (1) invoke methods with arguments on web components (general solution replacing previous hacks), (2) content binding on file node — read file content without separate function node.
- **Table primitives merged (Tyler)**: Components: table head container → table header cells / table rows. Default styles; Arvin to work on styling. Bug reported: global styles not working on tables.
- **Builder copy-paste improvements (Didier)**: Node swapping now available at L1 (was L2 only). New duplicate node dialog (replace / create copy / keep both) — eliminates three-step process for duplicate home screens. L2→L1 paste now shows proper warning (was silent failure). Search optimization: UI no longer freezes when multiple catalogs loaded.
- **Synced preferences infrastructure (Didier)**: Cloud sync for user settings — name-value pairs synced across extension/desktop/mobile. General-purpose infrastructure for any app.
