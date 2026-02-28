# Standup Notes — Feb 2026 (Early)

**Coverage:** Feb 2–13, 2026 | Newer entries: [standup-notes-feb-2026.md](./standup-notes-feb-2026.md)
**Format:** Condensed bullets per standup. Google Doc link on each date for on-demand full transcript fetch.

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
