# Standup Notes — Mar 2026 (Week 2)

**Coverage:** Mar 10–13, 2026
**Format:** Condensed bullets per standup. Google Doc link on each date for on-demand full transcript fetch.
**See also:** [standup-notes-mar-2026-1.md](./standup-notes-mar-2026-1.md) — Mar 2–5

---

## 2026-03-13 — [Google Doc](https://docs.google.com/document/d/1s7gZWM2_uN3PoRPTbHRqfQtPlz7nFAK0F99VTi712qA)

- **Sales pipeline progress (Mukund)**: Cycles with Starlight (RCS messaging) and US (services company) progressing well. Zendesk partnership conversations moving forward. Snowflake submissions
  proceeding, goal to submit next week. All current engagements transitioning to commercial discussions — pricing/scope proposals needed.
- **Lumber = largest current go-live effort**: Other customer efforts remain in sales motion phase. Direct deals: Starlight. Partner-type: US, IP agent AI.
- **Deal visualization decision**: Quadrant chart to categorize/track deals. Will leverage HubSpot with quadrant annotations and notes per customer. Made available via Slack automatically, starting ~
  2026-03-20.
- **Website refresh**: New messaging emphasizes building "context-aware experiences." Design expected to change slightly but core messaging is set.
- **RCS demos well received externally**: External demos more polished than internal versions. Full scheduling flow via RCS particularly well-received by USG Global, Starlight, Zendesk contacts.
  Zendesk rep saw immediate applicability after viewing demo video.
- **Web-based Remix board tool (Vijay)**: Fully web-based, cards in standardized JSON card format (not HTML) with data binding layer. Multi-format export (PPTX, HTML) rendered by Rust — no browser in
  flow. Use cases: landing pages, customer case studies, sales dossiers auto-generated from CRM data. Runs Rust Wasm (no download required). Also demonstrated on native Swift app for mobile widgets.
  CMOs have expressed high interest in rapid landing page creation.
- **Node counter zero bug fix (Simon)**: Removing stage 1 codegen for desktop caused stage 2 to be first encounter of constants → counter starts at 0. Fix: compute only node validity instead of
  recomputing full RTI. PR submitted.
- **Constants in app meta debate (Didier vs Simon)**: Didier proposed materializing constants in app meta (like styles) to avoid compile/import issues during codegen. Simon concerned about inflating
  runtime app meta size (e.g., RMX Tailwind already replicated in every runtime app meta). Current behavior: full runtime app meta is created for self-contained distribution — `make` creates stable
  copy unaffected by subsequent builder changes.
- **Desktop screenshot challenge (Didier)**: Embedded Safari view + mixer's string prefix format for external blob objects makes screenshots difficult. Simon suggested Tauri plugins for screenshots (
  similar approach used for copy-paste).
- **Lumber implementation architecture (Didier)**: Desktop builder + runtime. Users query Snowflake via AI → generate reports/charts in runtime → copy/paste cards into builder to compose dashboards →
  produce .remix file → render in client portal.

---

## 2026-03-12 — [Google Doc](https://docs.google.com/document/d/19I6hu6Iykg80W2ZkjkrWefctkmzsH4OXmMkd1_BeO2o)

- **USG Global demo (John Dismore)**: Product demo for USG Global, a multi-billion dollar consulting/systems integrator (comparable to Accenture/Deloitte). Thai duty-free store use case with Wolver
  chat app — product comparison, store finder flows. Scheduled for noon. Recorded self-service version would be valuable for partner to share with their customers.
- **"Experiences" architecture (Vijay + John)**: AI maps user requests to invocable "experiences" — RMX Remix files provisioned across channels (chat, RCS, landing pages). Experiences registered with
  descriptions so AI knows what to show. Simple models sufficient (even "Apple Intelligence") — primary AI role is dispatch/mapping, not content generation. No canned scripts; standard app against
  standard service agent.
- **Twilio RCS integration demo (Wilbur Claudio)**: Interactive web view inside SMS app. E-commerce catalog example opens tequila catalog on "shop now" tap. Meeting booking experience fully functional
  against Google Calendar. Web view size is controllable. Same functionality available on WhatsApp. Webhook logs user interactions.
- **Twilio platform opportunity**: Poor UX and documentation on Twilio side. Web view config requires API (content builder doesn't support it) — identified as opportunity for Remix to add value on top
  of Twilio layer.
- **Lumber facets + pagination (Mark)**: Implemented pagination and sorting logic for CT-based query set. Component outputs page limit + offset to query. Arvind working on design changes using
  existing table UI alongside this logic.
- **PubSub bug**: Known bug requiring refreshing the log view. Affects runtime. Full repro to be created post-demo.
- **GTM strategy — experience vocabularies**: Build up large vocabulary of domain-specific experiences → take to market. Exploring partnerships with Twilio, Zendesk Sunshine Conversations, Broadway.
  Deciding between direct sales vs partnership approach.

---

## 2026-03-11 — [Google Doc](https://docs.google.com/document/d/1f7n5z9qnfsWK0UqU3ScqqPfq26k8J23gR6D_RhbErbc)

- **Lumber 5-report delivery scope**: Client PM (Oleg) documented 5 specific reports as proof points. Reports demonstrate: facets, selection criteria, downloading, single facet driving multiple
  components (bar chart + table), adjusting technical DB fields to user-friendly column headers. Two chart types needed: bar and scatter.
- **Component publishing workflow**: Build configurator UI around component → configure via friendly interface → clipboard → paste into drop zone on Studio screen → one-click publish to preview
  bucket. Process "almost there" per Arvind.
- **Client expectations clarified (Reza + Vijay)**: No imminent go/no-go decision. Client now understands customization is on top of platform. Client acknowledges Snowflake schema + ETL is their
  responsibility (~1-2 months). Team must return to client with scope and timeline for delivery phases immediately.
- **Product positioning**: Team currently positioned as "report composition and publishing product" — not full development platform. If team avoids training client on Studio, must provide full tooling
  for publishing + lifecycle management (schema differences between environments, testing, authorization).
- **Lifecycle management = significant scope**: Handling schema differences, publishing, testing, authorization across environments represents major portion of overall requirements.
- **Delivery phasing strategy**: Provide hard dates and scope. Include "should" category low-hanging fruit items. Leverage existing relationship to shape phasing even if cycle is 3 weeks.

---

## 2026-03-10 — [Google Doc](https://docs.google.com/document/d/1oAgvK2ZhSiWqoEVM6Eq26PZEgSZDhbknB6DYxk7N9-U)

- **Documentation format overhaul (Gerd)**: Existing Notion docs confirmed obsolete. New docs use custom format with processor that adds typing info → JSON output → easily converted to Markdown.
  Enables LSP integration and AI training (successful experiments training AI to generate functions from machine-readable docs). Doc viewer app broken — fix is current sprint priority.
- **LSP status (Oliver)**: Refactoring parser to separate type creation from pure syntax parsing. Key step: switch LSP messaging from WebSocket to standard browser messages (compiler runs in browser).
- **Engineering process decision**: Creating GitHub issue is standard practice for new work. Immediate bug fixes can proceed via PR without issue. Bare PRs auto-appear on the board. Sprint labeling: "
  sprint zero" and "sprint one" labels added across all repos. "Ready to pick up" column = prioritized queue. Old backlog renamed to "icebox."
- **File upload + web component improvements (Simon)**: Bug fix for make agent error surfacing when VM ID already in use. Restored drag-and-drop file upload for desktop + plugins. Two content PRs: (1)
  invoke methods with arguments on web components (general solution replacing previous hacks), (2) content binding on file node — read file content without separate function node.
- **Table primitives merged (Tyler)**: Components: table head container → table header cells / table rows. Default styles; Arvin to work on styling. Bug reported: global styles not working on tables.
- **Builder copy-paste improvements (Didier)**: Node swapping now available at L1 (was L2 only). New duplicate node dialog (replace / create copy / keep both) — eliminates three-step process for
  duplicate home screens. L2→L1 paste now shows proper warning (was silent failure). Search optimization: UI no longer freezes when multiple catalogs loaded.
- **Synced preferences infrastructure (Didier)**: Cloud sync for user settings — name-value pairs synced across extension/desktop/mobile. General-purpose infrastructure for any app.

---

## 2026-03-16 — [Google Doc](https://docs.google.com/document/d/1LDofaFupgzEcJoiJql2NVvYzFoveQFCTO6Y7j-T6p10)

- **Command FFI development (Benedikt Becker)**: Started work on command FFI to allow running host commands from 'mix' with security measures. OPFS PR comprehensive, ready for merge Friday (may have
  breaking changes affecting databases, file installation, caching).
- **CI flow design + Snowflake native app (Chris Vermilion)**: Completed design sketching for continuous integration flow for remix extension (Tyler expected to implement). Snowflake native app
  finishing touches: glue code for accessing customer data functional. Admin tasks on compute instances next (VM node performance testing).
- **Command line utilities (Fred Im)**: Building CLI for database management/fixing using 'clap' (same approach as Benedikt's mixer work). Will merge features into standard mixer executable.
- **Agent deployment plugin (Gerd Stolpmann)**: First version complete — allows selecting library agents + deploying to target app/workspace (app must exist, user needs write permissions). Deploy
  target stored as "cloud deployment" RMX type record (modules, renamings, constants).
- **Agent destinations as sources (John Dismore)**: Demonstrated feature to save agent destinations as "sources" for auto-config of in-params. Proposed storing in prefs (like library list) for
  cross-app accessibility and admin bootstrapping. Follow-up: rationalize with Gerd's deployment plugin.
- **Agent alias creation plugin (John Dismore)**: New plugin demonstrated for workspace tooling — create aliases by selecting base agent + configuring fixed parameters. Can extend fixed parameters for
  specialization. Will be merged into standard toolkit.
- **AST + phrases work (Oliver Bandel)**: "As match patterns" merged. Now starting phrases work with tests. Work will be ready once phrases complete.
- **Grid component waiting (Reza Mohsin)**: Waiting for finalization, plan to sync with Arvin soon.

---

## 2026-03-17 (Morning) — [Google Doc](https://docs.google.com/document/d/1P_Y0mu7Op4zMzcQTQCMReYL1ZnkNGEwHhJRnwqCncl8)

- **Meeting skipped**: Low attendance (Chris absent) → no agenda → team decided acceptable given other work priorities.

---

## 2026-03-17 (Afternoon) — [Google Doc](https://docs.google.com/document/d/1C6AQ6JLLn35ZiCSQwgXouTUaOPFTgXE6HPLCeAVn2fE)

- **Data source storage in sync prefs (Didier Prophete)**: Working on storing agent destination records in sync preferences today (delay ~1 day). Old AMP feature for saving images as blob previously
  blocked screenshot functionality.
- **Desktop screenshot fixed (Didier Prophete)**: Old screenshot library (unmaintained, multi-second latency) replaced with new solution (millisecond performance). Complex screens (tables) still ~7
  seconds. System detects oversized elements, bypasses screenshot.
- **Style sync issue (Benedikt Becker)**: Styles disappear during "rebuild and make." Related: themes synced appear empty sometimes. Old records deleted, new inserted — newly saved records may not
  persist. Reported to Fred Im.
- **Table component status (Arvind Thyagarajan)**: New in-house table component nearing Lumber library readiness. Fine-tuning sticky elements (footer total row, left-side sticky columns). Single
  config object (data rows + column config) → needs SQL-driven component config post-query.
- **Grouped headers requirement (Arvind Thyagarajan)**: Working on implementation based on Oleg's example. Complex transposition/data mixing makes builder reasoning difficult.
- **TUI web component debate (Didier vs Tyler)**: TUI component potentially offers sticky headers/resizable columns for free, but lacks event dispatching for column changes (persistence). Tyler
  clarified TUI web component doesn't persist resizing.
- **Job costing report priority (Arvind + Reza + John)**: Assemble report now (even if table substandard) for customer call tomorrow. Job costing report + payroll section demo → schema-focused call
  tomorrow → prepared reports day after.
- **Chart component missing (Arvind)**: Toast UI chart integration unassigned. John offered to help integrate once table packaging complete. India team potentially takes chart work (timezone/timeline
  constraints).
- **Table requirements summary (Arvind)**: Grouped columns, totals row (sticky bottom), 1-2 sticky left columns, sort functionality, CSV/Excel/PDF export. Column width resize not tackled yet (complex
  custom JS, resource constraints).
- **Task breakdown needed (Vijay)**: Comprehensive breakdown of all customer requirements, not just next increment. Separate meeting scheduled.

---

## 2026-03-18 — [Google Doc](https://docs.google.com/document/d/1K5g3NwkOdGzsN82i34v-xoMjlwIMAhr-RcrJzbo5FYU)

- **Plugin debugging + system plugins (John Dismore + Benedikt Becker)**: System plugins disabled on Lumber workspace by default. John will adjust Benedikt's user settings to enable for debugging.
  System plugin usage issue blocks productivity.
- **India team delivers clean Lumber tables (Reza Mohsin)**: Clean reporting table set created, hourly population, pushed to production. Lumber project ready for schema design collaboration.
- **Snowflake schema design call (Reza + Vijay)**: Friday call with Anand/Trinad to present/suggest schema structure for payroll reporting. Team expects resources for report-building next week
  post-design.
- **Data cleanup timeline met (Reza)**: Lumber team delivered clean data + tables on schedule. Focus remains on first deliverable (composing reports) before customer customization requests.
- **Compiler errors ongoing (Didier)**: Critical fix ready. Edge case errors expected when changing compilation technology/protocol. Almost resolved.
- **Desktop vs AMP debugging efficiency (Gerd + Didier)**: AMP still convenient for quick UI testing locally. Should establish similar debug process for new desktop environment.
- **Screen save performance killer (Didier + Gerd)**: Saving screen with table component takes ~5 seconds. Database likely source. Needs investigation.

---

## 2026-03-19 — [Google Doc](https://docs.google.com/document/d/1iBzIjYrRmrjGOL1Obn54VSpa1yTSpdjRW3PofN473iw)

- **Data source persistence (John Dismore)**: Works at L2, inconsistent at L1. Storage decision: user-centric (cross-app) preferred over app-specific, especially for database service agents.
- **Alias management extended (John Dismore)**: Alias creation flow now shows all aliases for specific database within workspace. Allows drilling into individual alias to copy FX params, delete
  aliases. Delete component published to library for reuse.
- **Preferences management infrastructure (John Dismore)**: Working on allowing users to toggle system preferences display + set home screens. Prototyping tools for content creation (slides, landing
  pages).
- **RCS project focus shift (John Dismore + Wilbur Claudio)**: Immediate focus: calendar booking main experience (embed + RCS context). Extensive cleanup of demo "hacks" underway for Starlight
  deployment. Current state unfit for demo (likely next day).
- **RCS feature expansion planned**: Facebook Messenger + WhatsApp integrations post-basic scheduling (demo purposes, not Starlight immediate).
- **Lumber facet builder updates (Mark Loevenstein)**: Cleaned up design supporting new facet types: searchable select + text search facet (filters against 1+ table columns with single text box).
  Close to pushing; next: table builder design review, alias implementation, account-level config.
- **Starlight RCS proposal (Mukund Ramaratnam)**: Finalizing proposal; John + VJ + Wilbur + Yum meet post-standup to discuss.
