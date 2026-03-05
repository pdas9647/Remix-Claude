# Standup Notes — Feb 2026

**Coverage:** Feb 2–27, 2026
**Format:** Condensed bullets per standup. Google Doc link on each date for on-demand full transcript fetch.

---

## 2026-02-27 — [Google Doc](https://docs.google.com/document/d/1VhqWT-wJVVN8KLVcwEazXEr_hyG90f6sPXcVrZx1DXY)

- **Desktop delivery plan (~Mar 3)**: Push dev → beta today. Chris owns beta cut. QA: India team Sunday night → Monday results. Full stable prod deferred 2–3 weeks. **John's rule**: only production builds to customers.
- **GCS redesign (Arvind)**: Infinitely-versioned URL scheme unsustainable → draft/test/public channels (max 1–2 versions). Channel terminology: mobile = app tiers (draft/test/public); desktop = build channels (dev/beta/release).
- **Workspace-as-environment (VJ)**: Isolated workspaces as deployment environments replacing GCS versioning. QA at workspace layer. Sync deferred: Arvind + Benedikt + Didier.
- **Lumber**: Oleg = single requirements owner. Schema discussion flagged as critical (VJ) — need denormalized guidance.
- **Pipeline (Mukund)**: Atomic, Starlight, legal firm advancing.

---

## 2026-02-26 — [Google Doc](https://docs.google.com/document/d/1yzv5uSmnV_FPh4cHrinc78VcJsfiFRkYUypZzb7Xe-M)

- **Facet builder demo (Mark, CTE confirmed)**: CTE stacking for filters. Select column → facet type → clipboard variant → paste → auto-populates top 20 values. Outstanding: display name; aggregator not in grid.
- **HubSpot widgets for Atomicwork (Wilber)**: Reworked for web dashboards. Widgets = templated service agents with alias records. Pure runtime unifies mobile + web.
- **Lumber customer call**: Customer struggles with generality — views features as binary. IT team resistant.
- **Mar 4 delivery commitment**: Reza: test on dev, package stable Tuesday night → deliver Wednesday. Deliverables: Desktop+Studio, home, catalog+configurators, report publish/preview.
- **Lumber arch debate (deferred)**: Reza/Arvind: catalog of `.remix` configurators. VJ: build specific app from library — generic approach equals config complexity. Arch sync next day.
- **Repo access resolved**: Arvind pushed live during standup. Aliasing confirmed: agents specialized via alias records.

---

## 2026-02-25 — [Google Doc](https://docs.google.com/document/d/1SuvpCldR5E0caz7-4-eGLN5JcmC-D4rVUBkeGZlPZ_E)

- **Lumber facets — two approaches**: (1) Arka's no-CTE: LLM → SQL with placeholders → string-swap. (2) CTE (John+Mark): no re-running AI per facet, safer for injection. Both to be compared. SQL injection concern (Vijay): accepted string swapping for now given Snowflake's sequence-based binds.
- **Propagation bug found**: Bind variable state issue during Arka's demo.
- **Publish workflow demo (Reza)**: Metadata → export/upload .remix → hosted URL → report record → web component viewer.
- **Demo channel risk**: Chris: lock to stable build, don't update before demo.

---

## 2026-02-24 — [Google Doc](https://docs.google.com/document/d/13Jxgr6nUeaQpI4W9O07aWXEpXV2Y7VZhsNK5C0gLmcE)

- **Sprint process decided**: Two-week cycle. GitHub board primary. Sprint = 'Ready to pick up'. Notion = high-level coordination. 5 repos auto-added.
- **6-month roadmap**: Platform stability for self-hosted Mixer (desktop, extension, mobile). Customer-driven: Lumber + Atomicwork.
- **Lumber release target: next Tuesday**: Trial on AGT prod. India team QA on Windows (catalog → flow → publish pipeline). Known friction trained, not fixed.
- **Lumber "steel thread"**: Plugin → publish → .remix URL → report registry → end-users load in web component. Each report = separate .remix (single screen).
- **Windows desktop**: Critical for Lumber. Manual build, not CI/CD. Fred to create download page with versioning.
- **Release strategy**: (1) Quick Lumber content deliverable now, (2) refined desktop product release later.

---

## 2026-02-23 — [Google Doc](https://docs.google.com/document/d/1bgjJ6N9eQUTdfi3g6tS3ujxEpYm0a0_YTRgeEQjzpUA)

- **Desktop "too many open files" root cause (Fred)**: ID-to-row-ID lookup index not loaded into memory → frequent file open/close. Fix: load lookup map into memory.
- **Lumber AWS docs**: Chris writing deployment docs. Existing = ECS but Lumber uses EKS (Kubernetes). Test install planned before Snowflake submission.
- **Lumber e2e demo targeting Wednesday**: Reza + Arvind: full flow (home, catalog, facets, grid, publish).
- **Build server for service agents (Gerd)**: Extending to build/deploy from catalog. Catalog items in editor screen 2 format. Integration fits well (3 lines). Next: plugin for deployment selection.
- **Desktop**: Benedikt returned, fixed runtime nav bug, created new tickets.

---

## 2026-02-20 — [Google Doc](https://docs.google.com/document/d/1kry9apOGDq3GnCwLjvgfv3PDpIyyGxEmsu3FmCUblqI)

- **HubSpot widgets**: John + Mukund to review and set up examples. Fix critical UX issues before Atomicwork presentation.
- **Snowflake campaign**: Lead-gen targeting first week of March. Lumber's Snowflake story resonating.
- **Content creation tool (Vijay)**: Nested tree data → card bindings → rendered templates. Uses: landing pages, knowledge articles, static reports. Hooked to MCP.
- **Customer status**: Funda = stalled. Orderly = expansion (2–3 more hotel chains). **Better View = lost**.

---

## 2026-02-19 — [Google Doc](https://docs.google.com/document/d/1DW7eXLROwpIe87wGpeyily7F-l5U5ioOg8CK5XQaUQQ)

- **HubSpot widget workflow (Wilber)**: 3 types — aggregate metric, bar chart, general query. Flow: sync HubSpot metadata → AI generates filters via service agent → configure → paste into L2 .remix.
- **Desktop declared non-viable**: "Too many files open" prevents saving, can't delete screens, suspected corruption. Team must use AMP until Benedikt returns. Desktop **not a viable build surface**.
- **Facet builder**: Mark working on CTE generation inside base query for Lumber.

---

## 2026-02-18 — [Google Doc](https://docs.google.com/document/d/1KCeVRawkVEcbk53rV5k6CFjFTTp_iLVMfcgtoR1Tkgc)

- **Lumber facets (Arka demo)**: Drop-down config → Cortex SQL → state variable → filter. Date facet done. Issue: Cortex sometimes returns empty SQL silently. Vijay: offer simple + advanced options.
- **Lumber integration proposal**: Didier wrote UI takeover + deployment doc. In-memory DB decoding removed (mid-Jan).
- **New team member**: **Arka Bhattacharya** — Lumber facets, mobile QA. **New website** in progress (Sirshendu + Mukund).

---

## 2026-02-17 — [Google Doc](https://docs.google.com/document/d/1CBOiVwk8AEo3wfvmXn0pUOJsAO3h4yYorY1kDZGINhw)

- **Eliminate AMP on mobile**: Options: Tauri, OPFS WebView, cloud Mixer. Fred: get off AMP in cloud first. AMP largely vestigial.
- **Deprecated styles**: Stop auto-including `_rmx_style`. Enforce migration to new design system.
- **File upload**: Simon's `file.register` node — returns URL for PUT. R2 URLs include 5-min temp auth.
- **Process**: Notion = product priorities, GitHub = tasks. Windows desktop: manual build, CI blocked by RCM TS→Go.

---

## 2026-02-13 — [Google Doc](https://docs.google.com/document/d/19SW1UXVfxookkfJx0VGQ7bGt-AqtEWSvkCAAdNXwsUA)

- **Website messaging**: "Context is king in AI." Three pillars: build, context prep (Snowflake), deploy (embed runtime).
- **Snowflake marketplace**: Listing submission targeting late next week. Campaign: network → marketing-driven pipeline.
- **Customer roundup**: Lumber = mid-impl (1-year, 5 builders). Atomicwork = pilot. Better View = holding. Orderly = expansion. Funda = push or close.
- **New team member**: **Nivesh Mishra**.

---

## 2026-02-12 — [Google Doc](https://docs.google.com/document/d/1VJn8-csg220vSBHLKjyL4NMgmxLq9wwl19s__jU7GSQ)

- **Atomicwork (Wilber)**: Widget creation flow (configure → copy into L2 .remix). Desktop propagation errors + file upload issues. Meeting pushed to Monday.
- **CTE standardization**: Mark examined CTE SQL + facets. Need single pattern. John scheduling sync with Mark + Reza.
- **Desktop file upload bug**: `file:` protocol URLs not allowed in browsers; `remix:` gives 403. Workaround: export to public directory. Bug for Benedikt.

---

## 2026-02-11 — [Google Doc](https://docs.google.com/document/d/10gKtcIsZZ9g8kOOHhUIOxHoAM0XhcoQZ_s3JGj8fANs)

- **Lumber resistance**: Reporting in Java ORM, not SQL. Strategy: position Snowflake layer as faster than refactoring 50K lines.
- **Repository agents deployed**: John deployed Gerd's sync agents to Lumber workspace. "GitHub for desktop" — no branching/merging, no DB records synced. Desktop Studio = primary dev surface.

---

## 2026-02-10 — [Google Doc](https://docs.google.com/document/d/1f_gyK2Muj9h7FP1ptNLVCH3-dublT1vr09wpA7Irg-c)

- **Engineering process formalized**: Bi-weekly sprints + retros. Reza takes over content PM. Notion for product views, GitHub for tasks. New requests via Notion to mitigate "VJ-driven development."
- **DB robustness issue**: Failed index cleanup → corruption → records file grew ~1 GB. Fix: ignore missing indexes, cleanup dirs, better logging.
- **Lumber**: Per-component .remix (Arvind) vs single app (VJ). Configurators should be Lumber-specific. Auth sandbox credentials received (**Descope**). Chris building auth flow.
- **Runtime HTML-to-card (Didier)**: VM function to render AI-generated HTML at runtime. **.remix publishing UX**: plugin invocation "painful" — proposals to simplify.

---

## 2026-02-09 — [Google Doc](https://docs.google.com/document/d/1ugVYmq8UuYu17OxINSRg09bEcBYu24z7TV6tLBKnGkI)

- **Mixer Swagger docs complete**: Full OpenAPI at `/v1/swagger`. V0 endpoints partially maintained.
- **Agent prod disk space outage**: Ran out despite prior cleanup. Need better monitoring.
- **RCM conversion to Go complete (Fred)**: Added tests + fake projects for round-trip verification. Deployment setup remaining.
- **Compiler**: Stack overflow fix (_rmx_tailwind). Deleting project service agent now deletes binary.

---

## 2026-02-05 — [Google Doc](https://docs.google.com/document/d/175dN9RbPPn_73xTnDEO2o1rSMy3k6TQ3xPh2j_u38Tg)

- **Widgets broken ~1 month**: JSON parameter passing in .remix export broken since switch to single file. Undetected due to no widget usage. Didier fixing.
- **Plugin tool**: .remix V2 export + direct workspace hosting. File privacy: public vs private (5-min temp key). Private upload bug found.

---

## 2026-02-04 — [Google Doc](https://docs.google.com/document/d/1RBsp93XOQXhDy6oIRigkEcO-Ypa-f95omCYk70knM44)

- **Lumber configurator built (Padmanabha)**: Schema/table select → Cortex SQL → grid. Copy-to-clipboard done. Facets next.
- **Lumber scope confirmed**: Self-service report creation, not delivering specific reports.

---

## 2026-02-03 — [Google Doc](https://docs.google.com/document/d/19Y4X458kgpRx1smnH9IuXjmY3Xnhl8dKaDee6F-_hcg)

- **Engineering process**: Bi-weekly sprints (GitHub + Notion). Chris volunteering as incremental PM. Series A is immediate goal; product definition "blurry."
- **Collapsible groups shipped**: Collapsed groups show dynamic connections. Library guidance: standard library should mostly be components, not groups.
- **Home app**: Unified across mobile/desktop/extension. Nearly complete on mobile. Lumber doesn't need it (deep link suffices).

---

## 2026-02-02 — [Google Doc](https://docs.google.com/document/d/10cJU3pvSy_syxNdZGtrrEW72jzvGpbKZe0G0PcIab7Y)

- **remix:// protocol lacks caching**: WebKit hardcodes caching for http/https only. `.wasm` files recompile every load. Partial fix: `mixt.wasm` loaded once, shared across VM instances.
- **Mix query**: Save behavior altered (AMP side too). Return value PR (queries don't always return records). RCM reimplemented for Windows desktop CI.