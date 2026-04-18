# #builder-runtime Slack Channel — Remix Labs

**Coverage:** Dec 31, 2025 – Apr 18, 2026
**Channel ID:** C58HC9EC8
**Topic:** Builder, remix.app, WebApp

**Key participants:** Simon (builder arch lead), Didier (features/cleanup), Tyler (reviewer/extensions), Arvind (UX/design), Wilber (workspace plugin/tooling), Gerd (backend/mixer)

---

## Key Decisions & Discussions

**Artifact URL for mixer-backed builder [Dec 31]:** `https://sh-builder.remix-app-2j0.pages.dev/e/artifact?_rmx_host=https://agt-dev.remixlabs.com&_rmx_ws=simon&_rmx_asset=...`

**Service agent build fix + mqtt SetLocal removal [Jan 5]:** Fixed desktop rebuild for service agents; removed unused `mqtt SetLocal` from runtime. turntable/pull/11625.

**Branch picker doesn't work with mixer [Jan 9]:** Branch URL drops path when `_rmx_host`/`_rmx_ws` params present. Branch picker is amp-only; no mixer equivalent planned.

**Amp feature cleanup [Jan 16]:** Didier removed InMem DB (replaced by app state tile) and session mirroring (unused). No objections. Debug channel: stdlib debug port for remote-controlling apps via
pub/sub; works in browser/desktop, not service agents.

**Groups as catalog items [Jan 20]:** Groups can be saved as catalog items with syncing. Complex external binding reconnection after sync. Merged.

**runtime.json 570KB → 106KB [Jan 21]:** Stripped tailwind style views (full card-picker views per style). turntable/pull/11650. Also: runtime.json duplicated in .remix (v1 artifact); use
`export_package` plugin for v2.

**Scenegraph JSON paste [Jan 24]:** Direct paste of scenegraph JSON creates card tile. turntable/pull/11656. Full paste catalog: string→value, rectangular JSON→table, arbitrary JSON→object, catalog
URL→download+insert, scenegraph→card. Missing: HTML→card (html2card).

**Workspace plugin: cloud export [Jan 24]:** Wilber added cloud workspace export. Issue: export silently fails with "register" for debug objects; works with "omit".

**appName vs dbName confusion [Jan 26]:** Dual naming causes confusion; backend prefers appName, frontend uses dbName. John proposed optional display name only; ongoing.

**DragGroup decoder removal [Feb 6]:** turntable/pull/11692. Old cards using DragGroup no longer decode.

**L0 plugins [Feb 12]:** turntable/pull/11708. Repository apps can be added to L0 dashboard. Button merged but hidden (no plugins registered yet).

**File node refresh [Feb 13]:** turntable/pull/11712. Refresh button added; desktop token fix. No more features on existing file nodes; file.register node built separately.

**File Register node [Feb 16]:** turntable/pull/11723. Exposes all `file.register` variants. Replaces partial File Upload action once Mix has native `blob://` URL handling.

**Remote Data Sources [Feb 18]:** turntable/pull/11730. L1 button for Remote Data Sources modal. Remote data sources ≠ remote catalogs (run apps vs browse catalogs). Applies to entire app, not
per-agent. Remote catalog list now synced via `_rmx_prefs` agents (not user preferences).

**Open-link in builder [Feb 21-22]:** Decision: always open links in new tab in builder, never navigate away. Desktop: external links always open in browser.

**Remove Server (Go) VM from runtime [Feb 23]:** Simon's long-term proposal: `remix-*.remixlabs.com` uses client VM (mixrt) exclusively. No change to remix.app/Desktop/Flutter. First step:
turntable/pull/11745 (plugins use client VM on Amp). In progress.

**State node propagation: won't fix [Feb 24]:** Single cell per state object → updating one field propagates to all consumers. Proper fix impossible for dynamic fields (common case). Real-world impact
negligible.

**`flex-shrink: 0` constraint [Feb 26]:** Can't remove from parent containers — prevents TUI grid/chart webcomp clashes. Leave as-is.

**"Download .remix file" action removed [Feb 26] (BREAKING):** turntable/pull/11760. Only worked with v1+amp; superseded by workspace export plugin. Existing apps using it won't compile after
Rebuild & Make. Wilber + Arvind must update before next publish.

**Protect core screens [Feb 27-28]:** Didier's fix: make home/settings/constants/symbols/styles/files screens undeletable (PR in progress). `_rmx_desktop` has no home screen (intentional — infra-only
app).

## Mar 2–7 Additions

**Read File node + file upload patterns [Mar 4]:** Simon added idiomatic Read File node for client VM — previously required API node workaround. Part of updated file upload patterns (no need to
rewrite existing screens).

**File node error handling decision [Mar 5]:** Simon proposed standardizing on exceptions (not error bindings) for Local Files and File Pointers. Gerd + Wilber agreed. **Decision: exceptions, not
error bindings.** Rationale: simpler for builder users; explicit error handling available via custom actions with `file.read()`.

**Web Component methods with arguments [Mar 6]:** Simon implemented parameterized method invocation on web components. Previously methods either took no params or relied on attribute-passing hacks (
race conditions). Tyler confirmed race condition eliminated. Backwards-compatible. Open issues: (1) no return type from methods — must use events, (2) zagjs-style components with many methods → 150+
bindings per node. Tyler proposed "edit mode" to select visible bindings.

**Synced prefs prefix convention [Mar 6]:** Didier added `builder.` prefix to builder synced prefs (`search.libs` → `builder.search.libs`). Clean prefix per surface (builder/mobile/desktop/extension).
turntable/pull/11788, approved.

**Elm virtual DOM perf: not a priority [Mar 6]:** Simon provided data showing Elm vDOM not inherently a perf bottleneck. Gerd: could switch to pure JS renderer with direct DOM instructions (no vDOM).
Vijay agreed with Gerd's approach in principle. Didier: rendering is ~5-10% of overall time (HTTP/agents/Snowflake dominate). **Consensus: not a priority; spike later.**

**Minimized groups hide error indicators [Mar 5]:** Wilber reported: nodes with red error glow are invisible inside minimized groups. Feature request to surface error state on collapsed groups.

**Platform version in builder UI [Mar 4]:** Arvind requested version number (e.g. 1.45/1.46) visible in builder. Agreed.

## Mar 10–14 Additions

**Pub/sub node discrepancies [Mar 11]:** Simon: cloud subscribe lacks topic context binding (subscribe has it). "topic type" binding exists but likely never used dynamically. Consensus (Gerd,
Benedikt, Arvind): keep topic type static; use decisions if needed. Arvind: keep aliased agent names from settings/data/config supported.

**Pub/sub refresh for parameterized bd [Mar 12]:** Simon implemented missing refresh for parameterized bd view (turntable#11805). Could not repro "with params bd, you only get the view" issue reported
by Gerd.

**Screen state across component boundaries [Mar 13]:** Arvind: once screen state is set or view topic triggered, it must be maintained when editing across component boundaries. No replies yet — open
design question.

---

## Channel Patterns

- Simon + Didier drive builder features; Tyler reviews PRs
- Amp-to-mixer migration steady theme: removing amp-only features, adding mixer equivalents
- Didier proactively cleans legacy (inmem DB, session mirroring, DragGroup)
- Low-traffic (~1-2 msgs/day); substantive posts with PR links and video demos

## Mar 14–21 Additions

**Cloud Subscribe QoL improvements [Mar 17, Simon]:** Simon shared a screen recording showing quality-of-life improvements for Cloud Subscribe nodes (no textual detail shared — video only).

**`deploy_lib_agent` system-wide plugin [Mar 18, Gerd → Simon]:** Gerd announced a new system-wide plugin with entry points `deploy_lib_agent_l1` and `deploy_lib_agent_l2`. Same `.remix` file as
`export_package`: `Build_Tools.remix`. Source code in the `Build_Tools` project. Gerd asked how to configure the plugin menu. Simon directed to turntable#11822 (merge to enable).

**inevent tile QoL improvements [Mar 19, Didier → John]:** Small description added for each event + selected nodes always visible in the inevent tile, even when the screen is used as a plugin. (
Screenshot shared.)

## Mar 24–Apr 3 Additions

**Auto-navigate trap fixed [Mar 24, Tyler]:** Navigation cells auto-navigated to sub-screens inside builder, trapping users. Fix: turntable/pull/11843. Modal now shows "Stay" option.

**Snowflake plugin: mixcore.wasm + screen params [Mar 26, Gerd → Simon]:** Two issues: (1) mixcore.wasm fails to load if plugin URL has wrong path; (2) `_rmx_` prefix is restricted — screen params
can't use it. turntable/pull/11856: run plugin independently of host params.

**Toast misalignment in rmx-remix no-shadow [Mar 30→Apr 1, Tyler]:** Toasts misalign in `rmx-remix` webcomp with `no-shadow=true`. Root cause: toast anchored to shadow root, not `document.body`. Fix:
turntable/pull/11871. Resolved Apr 1.

**Rename collapsed nodes at L2 [Apr 1, Didier]:** Shipped — nodes can now be renamed while collapsed at L2.

**MDI icon picker guidance [Apr 1]:** Use Studio's built-in icon picker (not direct MDI imports) for cross-surface compatibility.

**New L1 macro: Rebuild & Make post-clone [Apr 3, Wilber]:** New macro at L1 to run Rebuild & Make after a repository clone. Shipped.

**Subscription node visibility scoping [Apr 3, Arvind + Simon]:** Subscription node should not fire on non-visible screens. Arvind: lower views in a viewstack are silent EXCEPT the last view under an
overlay (must stay active). Simon: needs formalization. **No final decision.**

## Apr 4–11 Additions

**Embedding .remix in a webpage (Didier, Apr 7, 18 replies):** Notion doc with examples: https://www.notion.so/Embedding-a-remix-into-a-webpage-33a1d464528f80c7b9cdd44e7c2baa0f
Bug: Google fonts injected into `<head>` are blocked by shadow DOM. With `no-shadow` param, fonts work; without, they fail. Fix: turntable/pull/11898 (Didier, approved Tyler).

**`builderUrl` param removal from Screen Plugins (Simon, Apr 9, 8 replies):** Removing `builderUrl` — no reliable mapping to assets/API prefix across Desktop/Browser/Snowflake/remix.app. Replacement:
env variables for host-independent prefixes. Wilber agreed, will update Starlight plugin. PR: turntable/pull/11914.

**Settings L1 modal UX (Didier, Apr 9, 9 replies):** Editing app settings now shows a clean L1 modal instead of the L2 sidepanel. PWA settings separated — open question: still needed given
Desktop/mobile/AppClip/RCS coverage? Vijay: discuss early next week. **No decision yet.**

## Apr 11–18, 2026 Additions

### JS invoke via client actions [Apr 15–17, Simon/Didier/Tyler]

Simon demonstrated invoking arbitrary JS functions via client actions (only index.html changes — any platform host can enable this). Tyler: **cannot include in default runtime or Chrome extension**
due to `import(...)` syntax (Chrome store rejection). Simon: will implement in full-screen / `rmx-remix` web component instead.

Didier's alternative: **wasm-based js-eval (QuickJS)** — provide JS string + function name + args → result. Works in agents and browser; no `import()` needed. Desktop: JS files in Files tile are
backed by the local filesystem — editing in Finder is picked up live by the builder. Extension restriction: no webcomps, no wasm, no custom JS modules.

Apr 17 Simon: both approaches working at remix-dev.remixlabs.com/e/edit/simon/from_file and /eval_js. **Client actions only — not available in agents.**

### E2E test coverage table [Apr 17, Benedikt]

Benedikt started a Notion table of all modern stack configs with one-step manual tests: notion/33b1d464528f803d9c2fc9d8c52471db. Simon added runtime hosting doc (notion: 3451d464). Existing
playwright (`test:playwright`) uses Amp; Mixer switch blocked on mix-rs#1075. CI can fetch .remix files from internet — no need to commit to repo.

### Remote file fetch in File node — decision [Apr 16, Simon/Gerd]

Simon proposed adding `file.get` from remote mixer to the File node. Gerd: `file-manager` requires workspace admin privilege — end users won't have it. **Decision: don't add; define an agent instead.
**