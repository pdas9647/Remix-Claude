# #builder-runtime Slack Channel — Remix Labs

**Coverage:** Dec 31, 2025 – Feb 28, 2026
**Channel ID:** C58HC9EC8
**Topic:** Builder, remix.app, WebApp

**Key participants:** Simon (builder arch lead), Didier (features/cleanup), Tyler (reviewer/extensions), Arvind (UX/design), Wilber (workspace plugin/tooling), Gerd (backend/mixer)

---

## Key Decisions & Discussions

**Artifact URL for mixer-backed builder [Dec 31]:** `https://sh-builder.remix-app-2j0.pages.dev/e/artifact?_rmx_host=https://agt-dev.remixlabs.com&_rmx_ws=simon&_rmx_asset=...`

**Service agent build fix + mqtt SetLocal removal [Jan 5]:** Fixed desktop rebuild for service agents; removed unused `mqtt SetLocal` from runtime. turntable/pull/11625.

**Branch picker doesn't work with mixer [Jan 9]:** Branch URL drops path when `_rmx_host`/`_rmx_ws` params present. Branch picker is amp-only; no mixer equivalent planned.

**Amp feature cleanup [Jan 16]:** Didier removed InMem DB (replaced by app state tile) and session mirroring (unused). No objections. Debug channel: stdlib debug port for remote-controlling apps via pub/sub; works in browser/desktop, not service agents.

**Groups as catalog items [Jan 20]:** Groups can be saved as catalog items with syncing. Complex external binding reconnection after sync. Merged.

**runtime.json 570KB → 106KB [Jan 21]:** Stripped tailwind style views (full card-picker views per style). turntable/pull/11650. Also: runtime.json duplicated in .remix (v1 artifact); use `export_package` plugin for v2.

**Scenegraph JSON paste [Jan 24]:** Direct paste of scenegraph JSON creates card tile. turntable/pull/11656. Full paste catalog: string→value, rectangular JSON→table, arbitrary JSON→object, catalog URL→download+insert, scenegraph→card. Missing: HTML→card (html2card).

**Workspace plugin: cloud export [Jan 24]:** Wilber added cloud workspace export. Issue: export silently fails with "register" for debug objects; works with "omit".

**appName vs dbName confusion [Jan 26]:** Dual naming causes confusion; backend prefers appName, frontend uses dbName. John proposed optional display name only; ongoing.

**DragGroup decoder removal [Feb 6]:** turntable/pull/11692. Old cards using DragGroup no longer decode.

**L0 plugins [Feb 12]:** turntable/pull/11708. Repository apps can be added to L0 dashboard. Button merged but hidden (no plugins registered yet).

**File node refresh [Feb 13]:** turntable/pull/11712. Refresh button added; desktop token fix. No more features on existing file nodes; file.register node built separately.

**File Register node [Feb 16]:** turntable/pull/11723. Exposes all `file.register` variants. Replaces partial File Upload action once Mix has native `blob://` URL handling.

**Remote Data Sources [Feb 18]:** turntable/pull/11730. L1 button for Remote Data Sources modal. Remote data sources ≠ remote catalogs (run apps vs browse catalogs). Applies to entire app, not per-agent. Remote catalog list now synced via `_rmx_prefs` agents (not user preferences).

**Open-link in builder [Feb 21-22]:** Decision: always open links in new tab in builder, never navigate away. Desktop: external links always open in browser.

**Remove Server (Go) VM from runtime [Feb 23]:** Simon's long-term proposal: `remix-*.remixlabs.com` uses client VM (mixrt) exclusively. No change to remix.app/Desktop/Flutter. First step: turntable/pull/11745 (plugins use client VM on Amp). In progress.

**State node propagation: won't fix [Feb 24]:** Single cell per state object → updating one field propagates to all consumers. Proper fix impossible for dynamic fields (common case). Real-world impact negligible.

**`flex-shrink: 0` constraint [Feb 26]:** Can't remove from parent containers — prevents TUI grid/chart webcomp clashes. Leave as-is.

**"Download .remix file" action removed [Feb 26] (BREAKING):** turntable/pull/11760. Only worked with v1+amp; superseded by workspace export plugin. Existing apps using it won't compile after Rebuild & Make. Wilber + Arvind must update before next publish.

**Protect core screens [Feb 27-28]:** Didier's fix: make home/settings/constants/symbols/styles/files screens undeletable (PR in progress). `_rmx_desktop` has no home screen (intentional — infra-only app).

---

## Channel Patterns

- Simon + Didier drive builder features; Tyler reviews PRs
- Amp-to-mixer migration steady theme: removing amp-only features, adding mixer equivalents
- Didier proactively cleans legacy (inmem DB, session mirroring, DragGroup)
- Low-traffic (~1-2 msgs/day); substantive posts with PR links and video demos
