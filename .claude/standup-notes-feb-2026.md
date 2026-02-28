# Standup Notes — Feb 2026

**Coverage:** Feb 17–27, 2026 | Older entries: [standup-notes-feb-2026-2.md](./standup-notes-feb-2026-2.md)
**Format:** Condensed bullets per standup. Google Doc link on each date for on-demand full transcript fetch.

---

## 2026-02-27 — [Google Doc](https://docs.google.com/document/d/1VhqWT-wJVVN8KLVcwEazXEr_hyG90f6sPXcVrZx1DXY)

- **Desktop delivery plan (Tuesday ~Mar 3)**: Decision: push current dev → beta today (Feb 27). Chris Vermilion owns cutting the beta release (pending fix #147 + local agents fix landing). QA plan:
  India team starts late Sunday, results by Monday morning. Chris + Didier to assess Windows QA stability by Monday. Gerd: repeat QA Monday after weekend fixes; full switch to stable prod build
  deferred 2–3 weeks.
- **Build channel policy (John)**: Strongly against giving customers dev or beta builds — only production. If stability is a concern, delay delivery date rather than ship a non-prod build to the
  customer.
- **Lumber builder scope for QA**: Minimal — paste configured L2 components, connect them, publish, run `make`. QA cycles can complete quickly; a few full cycles possible in one day.
- **GCS infrastructure redesign (Arvind)**: Current infinitely-versioned GCS URL scheme is over-engineered and not sustainable. Proposal: replace with draft/test/public channels (max 1–2 versions per
  channel). Also update all default apps to V2 format in a stable location to resolve project sync confusion.
- **Channel terminology clarified (Arvind)**: "Channel" means different things in each context — mobile: end-user consumption tiers (draft/test/public for apps); desktop: build release channels (
  dev/beta/release). Both sets required for desktop; promotion mechanism between build channels still needs definition.
- **Workspace-as-environment model (VJ)**: Use isolated workspaces (e.g., staging workspace) as deployment environments, replacing complex GCS versioning. Deployment and QA of assets happen at
  workspace layer. Billing implication: future per-workspace pricing; concern about unlimited workspaces on self-hosted servers. Sync mechanism discussion deferred to smaller group: Arvind +
  Benedikt + Didier.
- **Lumber expectations set**: Shisha + Manish informed desktop will have "sharp edges"; runtime for end-customers must be stable. Oleg is the single authorized requirements owner at Lumber. After
  Tuesday delivery, next step = plan first functional report delivery date.
- **Schema discussion flagged as critical (VJ)**: Dedicated meeting needed to guide Lumber toward a denormalized or specific schema. Concern: loose design space if platform must query arbitrary end
  tables in reports.
- **Business pipeline (Mukund)**: Opportunities advancing — Atomic, widget work, Starlight, a legal firm. Marketing calendar in development; needs VJ approval before external release.

---

## 2026-02-26 — [Google Doc](https://docs.google.com/document/d/1yzv5uSmnV_FPh4cHrinc78VcJsfiFRkYUypZzb7Xe-M)

- **Facet builder demo (Mark, CTE approach confirmed)**: CTE (`WITH` clause) stacking is the confirmed pattern for accumulating filters. Workflow: select column → facet type restricted by column
  type (text → single/multi-select with equals/not-equals; date also supported) → creates clipboard variant → paste into builder → facet auto-populates with top 20 distinct values from column.
  Multiple facets stack via additional `WITH` clauses. Only connection needed for final report assembly = main card outbinding. Still outstanding: display name/title for pasted facet; aggregator for
  state settings not yet integrated into grid.
- **HubSpot widgets for Atomicwork (Wilber)**: Reworked for web dashboards (not just mobile). Widgets = templated service agents; alias record configures the "widget agent" with parameters (e.g.,
  total deals won). Pure runtime flow unifies mobile + web widget delivery. Simon: widget parameters must follow conventions (similar to plugins) for rendering system compatibility. Web widgets not
  constrained by mobile size limits. Error handling relies on tool definitions to enforce structure; data quality depends on HubSpot account knowledge.
- **Lumber customer call feedback** (VJ + Arvind reporting from call): Customer struggles with platform generality — views features as binary ("is it there or not") rather than malleable/rapidly
  customizable. Communication hurdle: customer's IT development team is resistant (prior failures, focused on their own long-term roadmap).
- **Wednesday delivery commitment** (target: ~Mar 4): Reza's plan — stay on dev channel, aggressively test Lumber flow, package stable version Tuesday night → deliver Wednesday morning. Customer will
  NOT receive dev or beta build. Reza's enumerated deliverables: (1) Remix Desktop + Studio, (2) home app, (3) catalog with configurators, (4) report publishing/previewing tool.
- **Desktop release process**: Engineering team (Chris) owns delivery quality. QA capacity allocation remains an ongoing challenge — heroic per-delivery effort is not sustainable. John proposed
  accelerating dev→beta promotion to a daily cadence. Chris to communicate desktop production channel migration steps.
- **Lumber architecture debate (deferred)**: Fundamental disagreement on deliverable structure. Reza/Arvind: catalog of `.remix` configurators; content team populates; runtime configuration. VJ: build
  a specific Lumber app from library components directly — generic platform approach creates configuration complexity equal to building the app itself. VJ/John/Reza scheduled arch sync for next day.
- **Repository access resolved during standup**: John could not access Arvind's project (components missing from library). Arvind pushed to repo live during the meeting → John confirmed successful
  pull. Follow-up: Arvind to add home project to Lumber workspace repo; John to add agents to separate repo. Aliasing pattern confirmed: agents (e.g., `save catalog item`) specialized via alias
  records without passing full entity info.
- **Content generation app** (John): Building Remix app extending VJ's landing page generation work; goal is an AI-driven content generation tool for Mukund. In progress.
- **Bugs fixed/flagged**: CSS overflow bug in Studio dev env (edit button unreachable) — Simon fixed same day. Member query + projection issues flagged by John — Simon to follow up with Fred.

---

## 2026-02-25 — [Google Doc](https://docs.google.com/document/d/1SuvpCldR5E0caz7-4-eGLN5JcmC-D4rVUBkeGZlPZ_E)

- **Lumber facets — two approaches under evaluation**: (1) **Arka's no-CTE approach**: LLM receives DDL + bind variable format + user prompt → returns SQL with placeholders → string-swap placeholders
  with user-selected facet values at query time. (2) **John + Mark's CTE approach**: does not require re-running AI per new facet, potentially safer for value injection. Consensus: both versions to be
  developed and compared before committing.
- **SQL injection concern raised** (Vijay): Arka's string-templating approach vulnerable to injection. Bind variables proposed as mitigation but Snowflake API uses sequence-based binds (1, 2, 3...),
  not named — hard to correlate variables when a field is used multiple times. Team accepted string swapping for now given Snowflake limitations.
- **Facet no-value case**: Arka's solution — ask LLM to return two queries: SQL with placeholders (filter present) + SQL without WHERE clause (no facet selected). System runs appropriate version based
  on user input.
- **Propagation bug found**: John identified bind variable state issue + "weird state thing" during Arka's live demo. Vijay identified as a propagation bug. Needs investigation.
- **Home screen + publish workflow demo** (Reza driving; Arvind's machine styles broken by a `make` issue from recent desktop update): Home screen is shared across all Lumber users. Publish report
  wizard flow: set report metadata → export/upload .remix file → retrieve hosted URL → add URL to report record → end-users select report from list → loads via Remix web component viewer.
- **Lumber stakeholder demo**: Presentation scheduled Feb 26 at 8:00 AM Pacific. Two parallel demo versions: John + Mark (CTE approach) and Reza + Arka (no-CTE approach). Next steps: fix home app
  styles; Arka to add no-WHERE-clause query and put facet code in repo; Arvind to send registration flow video.
- **Demo channel risk**: Reza concerned about using Remix Desktop's dev channel for the presentation (active dev fixes may break app). Chris's recommendation: lock to a stable desktop build, do not
  update immediately before the demo.

---

## 2026-02-24 — [Google Doc](https://docs.google.com/document/d/13Jxgr6nUeaQpI4W9O07aWXEpXV2Y7VZhsNK5C0gLmcE)

- **Sprint process decided**: Two-week cycle. GitHub project board is primary (not Notion). Sprint = 'Ready to pick up' column. Notion = high-level coordination with product/sales. Manual
  Notion↔GitHub linking (auto-integration needs paid Notion). 5 repos auto-added (AMP, Groove Box, Mix RS, Protoquery, Turntable); others manual.
- **6-month roadmap**: Platform stability + bug fixes for self-hosted Mixer setup (desktop builder, Chrome extension, mobile container). Customer-driven: Lumber + Atomicwork are the two driving
  deliverables.
- **Lumber release target: next Tuesday**: Trial on AGT prod server (company infra, not behind Lumber's firewall). India team does exhaustive QA on Windows Thursday/Friday for full pipeline (catalog →
  flow → publish). Known friction (manually finding report template, creating project) will be trained, not fixed for this release.
- **Lumber "steel thread" publish flow**: Plugin → one-click publish → generates .remix file URL → report registry form creates record → end-users select from report list → loads in embedded RMX Remix
  web component. Each report = separate .remix file (separate project, single screen). Team acknowledges this will change later but proceeding now.
- **Windows desktop**: Critical for Lumber. Manual build, not CI/CD. Fred to create professional download page with versioning. Build will be frozen + QA'd.
- **Alignment debate**: Gerd: issue isn't innovation vs release, it's unclear goals preventing convergence. Reza: adopt release mindset, define use cases, identify gaps, assign accountability with
  dates. VJ: most Lumber deliverables are content-layer, few engineering blockers. John: two core engineering deps — custom OR provider + behind-firewall deployment.
- **Release strategy**: Two snapshots — (1) quick Lumber content deliverable now, (2) refined desktop product release incorporating Lumber learnings later.

---

## 2026-02-23 — [Google Doc](https://docs.google.com/document/d/1bgjJ6N9eQUTdfi3g6tS3ujxEpYm0a0_YTRgeEQjzpUA)

- **Desktop "too many open files" root cause found**: Fred identified — ID-to-row-ID lookup index not loaded into memory, causing frequent file open/close on every request. Fix: load lookup map into
  memory. PR expected soon. Benedikt also addressed the issue (not a leak, just excess open files).
- **Sprint planning**: Chris organized backlog. First sprint planning session scheduled for tomorrow's engineering meeting. Will discuss two-week cadence format.
- **Lumber AWS docs**: Chris writing deployment documentation. Existing process uses **ECS** but Lumber uses **EKS** (Kubernetes) — requires adjustments. Test install planned in Remix AWS account
  before Snowflake submission.
- **Lumber e2e demo targeting Wednesday**: Reza + Arvind aiming for full flow: updated home app, catalog screen, working facets with configured grid, publish flow. Arvind connecting with Wilbur on
  publish side. SQE work needed for facet→grid connection.
- **Lumber stakeholder confusion**: External stakeholders still questioning proposed approach + UI design (single button for reports, report creation flow). Reza to call Oleg to clarify.
- **Build server for service agents** (Gerd): Extending build server to build/deploy service agents from catalog. Catalog items stored in older editor screen 2 format (fully JSONified). Code
  generation script has detail error — not producing output yet. Integration fits well (3 lines of setup in recipe). Next: plugin to list workspace service agents for deployment selection.
- **Desktop improvements**: Benedikt returned, fixed runtime navigation bug, working with Gerd on dev experience, created new tickets.

---

## 2026-02-20 — [Google Doc](https://docs.google.com/document/d/1kry9apOGDq3GnCwLjvgfv3PDpIyyGxEmsu3FmCUblqI)

- **HubSpot widgets**: John + Mukund to review flow and set up Remix examples. Mukund as "guinea pig" (sales analytics). Fix critical UX issues before presenting to AD (Atomicwork).
- **Snowflake campaign**: Lead-gen campaign targeting first week of March. Finalizing messaging, landing pages, target company list (partners + Snowflake users). Lumber project's Snowflake story
  already resonating with other companies.
- **Content creation tool** (Vijay): Nested tree data → auto-apply to card bindings → render templates for static content. Use cases: landing pages, closed cases → knowledge articles, static reports.
  Hooked up to MCP for element changes. Demo with Arvind before team rollout.
- **Customer status**: Funda = not a sales cycle (client doesn't strongly need it; internal issues). Orderly = expansion opportunity (2–3 more hotel chains). **Better View = lost** — found own
  solution; pattern reused in content creation work. OEM/white-label model (Better View type) generally not ideal — small vendors constantly evaluate build-vs-buy.

---

## 2026-02-19 — [Google Doc](https://docs.google.com/document/d/1DW7eXLROwpIe87wGpeyily7F-l5U5ioOg8CK5XQaUQQ)

- **HubSpot widget workflow** (Wilber demo): 3 component types — aggregate metric (count), bar chart, general query (with lookups). Flow: sync HubSpot property metadata (admin enablement step) → AI
  generates parameterized filters via service agent → configure component → copy/paste into L2 .remix. First two types could be runtime-only; general query requires L2 builder for data→card binding.
  Wilbur building runtime-only version for bar chart + total.
- **Facet builder**: Mark working on CTE generation inside base query. Post-standup sync with John + Vijay to finalize for Lumber end-of-week.
- **Content production app**: John + Vijay building app for AI-driven card/card-data generation + modification. Demo postponed.
- **Desktop declared non-viable**: Wilber hit severe issues — "too many files open" error prevents saving, can't delete screens, suspected data corruption. Gerd/Chris: likely Rust file handling or
  unflushed write logs (DB journals to short-lived files, flushed at size threshold). Team must use AMP until Benedikt returns Monday. Desktop currently **not a viable build surface**.
- **Dynamic widget preview**: Next step is testing on mobile. Didier indicated it may already be installed by default in workspace.

---

## 2026-02-18 — [Google Doc](https://docs.google.com/document/d/1KCeVRawkVEcbk53rV5k6CFjFTTp_iLVMfcgtoR1Tkgc)

- **Lumber facets**: Arka demoed drop-down facet configuration — select tables, enter prompt → Snowflake Cortex generates SQL. Selected value written to state variable, used as filter in report SQL.
  Date field facet also done. All config screens published to Lumber workspace. **Issue**: Cortex sometimes fails silently (empty SQL string) — needs user-facing error handling. Vijay: offer simple (
  no AI) + advanced (Cortex) options. Rename "path value" to "facet name" for clarity.
- **Lumber technical integration proposal**: Didier wrote document detailing UI takeover + deployment methods. Reza + Arvind to review before call with Lumber tomorrow.
- **In-memory DB decoding removed**: Simon clarified query tiles using in-memory DB will not decode — support pulled mid-January.
- **Snowflake fixes**: Padmanabha fixed widget bugs + undecodable component issues.
- **Mobile app flow issue**: Specific flow not showing — Didier believes user-space/installation problem on Arvind's end, not mobile app issue. Must fix before app store submission.
- **Orderly**: Plugin not being used (no activity in logger). No new requirements. Mukund to check logs.
- **New website**: Sirshendu + Mukund building new version with demos (app clips, web apps, reports, widgets, chat).
- **New team member**: **Arka Bhattacharya** — Lumber reporting facets, mobile QA.

---

## 2026-02-17 — [Google Doc](https://docs.google.com/document/d/1CBOiVwk8AEo3wfvmXn0pUOJsAO3h4yYorY1kDZGINhw)

- **V1 release blocker**: Extension inclusion in release cycle. Simon questioned perpetual "V1" label — products shipped for 5 years without formal V1.
- **Mobile app status**: Generally satisfactory. Home app consistency (desktop/extension/mobile) is priority — Arvind working on it. Mobile is most usable home app implementation currently.
- **Eliminate AMP on mobile**: Discussion on removing AMP to stop maintaining Go/Rust DB sync. Options: Tauri for mobile (simpler than Flutter without AMP), OPFS-based DB in WebView (persistence
  concerns raised by Didier), cloud-based Mixer. Fred: prioritize getting off AMP in cloud before mobile. AMP server largely vestigial, mainly maintained for mobile.
- **Windows desktop**: Manual build exists. Blocker for automated/releasable: RCM move from TypeScript to Go.
- **Notion + GitHub process**: Notion = high-level product priorities (entry point for product team requests). GitHub = granular tasks. Not every small bug fix needs a ticket first, but most work
  should start as an issue. Sprint = internal visibility tool, not rigid contract. Two-week sprint goal selection pushed to next week. Immediate: clean up task lists, assign tickets to ongoing work.
- **Deprecated styles decision**: Stop auto-including old RMX_style in new projects. Enforces migration to new design system. Users can manually add old style if needed. Didier will include in next
  desktop + builder release.
- **File upload simplification**: Simon working on `file.register` node — registration returns URL for PUT upload. Required for agent server (security), not needed for desktop local FS. R2 URLs
  include 5-min temp auth. Upload step may use existing API tile (PUT with binary) rather than custom node.
- **Desktop function node**: Simon working on "ripple part" — function node + screenshots are only known missing desktop components.
- **Sorting fix**: PR merged — yellow sorting node now supports nested fields.
- **Extension**: Tyler working on push notifications + cloud code skills for web component creation. Production release pipeline being discussed with Chris.
- **HubSpot widgets**: Wilbur has both examples working, ready for review.
- **Lumber auth**: Reza scheduling call with Chris + Didier. Reporting demo planned for Friday.
