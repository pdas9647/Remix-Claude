# Standup Notes — Feb 2026

**Coverage:** Feb 2–24, 2026 (ongoing)
**Format:** Condensed bullets per standup. Google Doc link on each date for on-demand full transcript fetch.

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

---

## 2026-02-13 — [Google Doc](https://docs.google.com/document/d/19SW1UXVfxookkfJx0VGQ7bGt-AqtEWSvkCAAdNXwsUA)

- **Website messaging**: "Context is king in AI." Three pillars: **build** (compose from catalog), **context prep** (sync records to Snowflake, semantic mapping), **deploy** (embed runtime in
  chatbots/IVR/web apps). Data lake = context engine.
- **Website demos**: (1) Interactive chat with embedded experiences, (2) custom mobile widgets with live data + role picker, (3) shareable insights reports (QBRs, sales, marketing). Mukund wants
  widgets shareable/syndicatable.
- **Unstructured data viz**: Calculations on embeddings via Snowflake notebook — tickets → low-dimensional representation → auto-clustering/classification. Record sync to data lake via webhooks (e.g.,
  Zendesk ticket sync).
- **Snowflake marketplace**: Public listing submission targeting late next week. Chris needs to finalize a few items. Campaign planned to transition from network leads to marketing-driven pipeline.
- **Customer status roundup**: Lumber = mid-implementation (1-year subscription, services + monthly fee, 5 builder users). Atomicwork = pilot after component work done. Better View = holding,
  evaluating desktop. Orderly ("Only Guys") = positive feedback, planning hotel expansion. Funda = push to move forward or close.
- **New team member**: **Nivesh Mishra** — asking about unstructured data viz and webhook-based record sync.

---

## 2026-02-12 — [Google Doc](https://docs.google.com/document/d/1VJn8-csg220vSBHLKjyL4NMgmxLq9wwl19s__jU7GSQ)

- **Atomicwork project**: Wilbur building widget creation flow (configure → copy into L2 .remix). Service agents + AI-prompted widget config screen. Desktop propagation errors + file upload issues.
  Atomicwork meeting pushed to Monday to resolve instability first.
- **Long-term reporting vision**: Current = desktop assembly for mobile widgets. Goal = web app dashboards with rich click-through, actionability, and AI assistance for enterprises (marketing, sales,
  service). Mukund: full framework laid out next week.
- **CTE standardization**: Mark examined CTE SQL alongside facets. Discussion needed to establish single pattern. John scheduling sync-up with Mark + Reza for Lumber.
- **Desktop file upload bug**: `file:` protocol URLs not allowed in browsers; `remix:` protocol gives 403 (missing token). **Workaround**: export to public directory bypasses token issue. System
  producing `file:` protocol is a bug for Benedikt. Works on AMP but not desktop context.
- **Notebook/scatter plot**: Mark templated notebook, cleaned web component. API execution for parquet data source not working — seeking workaround.

---

## 2026-02-11 — [Google Doc](https://docs.google.com/document/d/10gKtcIsZZ9g8kOOHhUIOxHoAM0XhcoQZ_s3JGj8fANs)

- **Lumber team resistance**: Reporting code is in Java ORM layer (not Postgres SQL) + upstream data architecture issues (API feeds, payroll integrations). Lumber team feels solution is imposed from
  top; reluctant — doesn't solve their main challenges. Strategy: position Snowflake-based reporting layer as faster than refactoring 50K lines of code.
- **Repository agents deployed to Lumber workspace**: John deployed Gerd's project sync agents. Arvind pushed catalog project → Padmanabha + Sirshendu pulled successfully. Functions like "GitHub for
  desktop" — syncs builder projects across users without AMP. **Limitations**: no branching, no merging, no concurrent screen update detection; users must work on different screens. No DB records
  synced, only builder records (code gen/compile needed after checkout). R2 storage writes slow. Checksum change bug (unnecessarily long change list).
- **Desktop Studio as primary dev surface**: John emphasized team should use desktop Studio instead of AMP for better performance.
- **Bomisco clipper update**: Padmanabha added error message showing current page URL + valid pages when used on wrong page (UI only).

---

## 2026-02-10 — [Google Doc](https://docs.google.com/document/d/1f_gyK2Muj9h7FP1ptNLVCH3-dublT1vr09wpA7Irg-c)

- **Engineering process formalized**: Bi-weekly sprint planning (Monday, 45 min before standup) + bi-weekly retro (Friday). Notion for top-level product views (by product: desktop v1, server v1,
  mobile, extension). GitHub for granular tasks. Reza takes over **content project management**. New requests routed through Notion prioritization to mitigate "VJ-driven development."
- **Vijay's position**: Platform "basically feature complete," should go to market; content is incomplete.
- **Bug tracking**: Keep Bug Bash Slack for initial triage; engineering converts threads to GitHub tickets (not reporter's job). Minimal repro encouraged.
- **DB robustness issue**: Failed index cleanup (deleting non-existent indexes) caused corruption — write logs not cleaned, records file grew to ~1 GB. No logging for flush failures. Fix needed:
  delete code should ignore missing indexes, cleanup unlisted index dirs, better logging.
- **Lumber architecture debate**: Per-component .remix files (Arvind: extensible, good for collaboration) vs single app (VJ: fragmented state sharing — Snowflake auth, schema params passed repeatedly
  between files). VJ proposed Gerd's plugin mechanism. Consensus: deploy Arvind's projects to Lumber workspace for team review. Configurators should be **Lumber-specific**, not general-purpose.
- **Lumber auth**: Sandbox credentials received (**Dcope** identity system, similar to Okta). Chris building auth flow.
- **Runtime HTML-to-card**: Didier building web component/VM function to render AI-generated HTML at runtime (not just builder). Broader utility for AI conversation display.
- **.remix publishing UX**: Plugin invocation called "painful." Proposals: default public + files app for one-click, plugin remembers user choices, elevate frequent plugins to main menu.
- **Mobile**: Widget config fix on TestFlight/Play Store. App store push targeting Thursday.
- **Desktop tech debt**: Simon — DB maintenance migrations for distributed desktop databases. Save/close grayed-out fix (compiler crash + timeout mechanism).
- **Library concerns**: Load time degrades as library grows. Ranking algorithm for search results. Asset migration needed (especially desktop).

---

## 2026-02-09 — [Google Doc](https://docs.google.com/document/d/1ugVYmq8UuYu17OxINSRg09bEcBYu24z7TV6tLBKnGkI)

- **Mixer Swagger docs complete**: Full OpenAPI documentation at `/v1/swagger` route (agent dev/beta). V0 endpoints not all maintained; some core functions (agent invocation, .remix upload) may remain
  V0 only.
- **Agent prod disk space outage**: Ran out over weekend despite prior index cleanup (had freed to 60%). Need better monitoring/alerting. Chris + Fred investigating.
- **RCM conversion to Go complete**: Fred added tests + fake projects for round-trip verification. No prior tests existed. Deployment setup remaining (build for all architectures). Chris to review.
- **Mix query**: PR to return values instead of records. Unflushed rights weren't being indexed — added index fix. Needs review.
- **Compiler**: Stack overflow fix (RMX tailwind project). Deleting project service agent now also deletes corresponding binary (new protocol flag for `make agent`).
- **AST work**: Type syntax part merged (refactoring, no user impact yet). Match patterns in progress — harder to test than type AST.

---

## 2026-02-06 — [Google Doc](https://docs.google.com/document/d/1sCXABFFytW6a0doht1FgqyUTyQZhVUWdblwizYaY8Ik)

- Low attendance, no substantive content.

---

## 2026-02-05 — [Google Doc](https://docs.google.com/document/d/175dN9RbPPn_73xTnDEO2o1rSMy3k6TQ3xPh2j_u38Tg)

- **Widgets broken ~1 month**: JSON parameter passing in .remix export broken since switch to single `.remix` file (agent + widget records). Undetected due to no widget usage. Didier fixing.
- **Scripted chat demo complete**: Wilbur placed on remix.app URL. AI pause feature praised.
- **Plugin tool**: .remix V2 export + direct workspace hosting. File privacy: public (open URL) vs private (5-min temp key). Private upload bug found.
- **Lumber**: Mark finishing scatterplot e2e. Extra libraries per component problem (Simon investigating). John reviewing Lumber component patterns.
- **HubSpot widget**: Needs Atomicwork fix-up, deferred.

---

## 2026-02-04 — [Google Doc](https://docs.google.com/document/d/1RBsp93XOQXhDy6oIRigkEcO-Ypa-f95omCYk70knM44)

- **Lumber configurator built** (Padmanabha): Schema/table select → Cortex SQL → grid display. Copy-to-clipboard done. Facets next. Reza building denormalized schema + semantic definition + data
  profile for AI.
- **Lumber scope confirmed**: Self-service report creation, not delivering specific reports. Parallelization concern flagged (many people, not parallel).
- **Bomisco extension**: LinkedIn pinging prevents clean injection; app-level reload fix risks perpetual reloads on selector failure. Vijay wants error differentiation.

---

## 2026-02-03 — [Google Doc](https://docs.google.com/document/d/19Y4X458kgpRx1smnH9IuXjmY3Xnhl8dKaDee6F-_hcg)

- **Engineering process**: Bi-weekly sprints decided (GitHub labels/milestones + Notion). Chris volunteering as incremental PM. Success requires John + VJ buy-in on clear goals. Series A is immediate
  goal; product definition "blurry." Release dates missed multiple times.
- **Collapsible groups shipped**: Collapsed groups show dynamic connections. Library guidance: standard library should mostly be **components** (not groups). Follow-up: Arvind + VJ + Didier.
- **Home app**: Unified across mobile/desktop/extension. Nearly complete on mobile but lacks remote-asset linking for customers. Lumber doesn't need it (deep link suffices).
- **Extension**: Clipper bug fixed, republished after remote code rejection. Bomisco: Mihir on direct-install version; flow enhanced with extract failure detection + reload.

---

## 2026-02-02 — [Google Doc](https://docs.google.com/document/d/10cJU3pvSy_syxNdZGtrrEW72jzvGpbKZe0G0PcIab7Y)

- **Desktop**: Blank screen of death still unresolved. MCP functionality restored. Native desktop rationale still debated.
- **remix:// protocol lacks caching**: WebKit hardcodes caching for http/https only. .wasm files recompile every load. Partial fix: `mixt.wasm` loaded once, shared across VM instances. Mix compiler
  startup cost inherent.
- **Mix query**: Save behavior altered (AMP side too). Return value PR (queries don't always return records). RCM reimplemented for Windows desktop CI (heavy review expected).
