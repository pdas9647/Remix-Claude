# #builder-runtime Slack Channel — Remix Labs

**Coverage:** Dec 31, 2025 – Feb 22, 2026
**Channel ID:** C58HC9EC8
**Topic:** Builder, remix.app, WebApp
**Purpose:** Discussion about builder functionality

---

## Key Participants

- **Simon** — builder architecture lead; runtime integration, remote data sources, L0/L1/L2 features
- **Didier Prophete** — builder features; catalog, paste, styles, amp feature cleanup
- **Tyler Lentz** — builder reviewer; extensions, runtime
- **Arvind Thyagarajan** — UX/design for builder features
- **Wilber Claudio** — workspace plugin, .remix export tooling
- **Gerd Stolpmann** — backend context; mixer integration, debug channel, runtime.json

---

## Key Decisions & Technical Discussions

### Dec 31, 2025 — Artifact URL for mixer-backed builder

- Simon shared working URL pattern for running builder against mixer with remote catalog assets:
  `https://sh-builder.remix-app-2j0.pages.dev/e/artifact?_rmx_host=https://agt-dev.remixlabs.com&_rmx_ws=simon&_rmx_asset=...`

### Jan 5 — Service agent build fix + mqtt SetLocal removal

- Simon merged fix for service agents not being correctly made in desktop rebuild
- Removed unused `mqtt SetLocal` from runtime — confirmed unused by Gerd, Didier, Tyler
- PR: turntable/pull/11625

### Jan 9 — Branch picker doesn't work with mixer

- Gerd: branch URL (`/b/<branch>/e?_rmx_host=...&_rmx_ws=...`) redirects and drops branch path when `_rmx_host`/`_rmx_ws` params are present
- Simon: branch picker is amp-only; no mixer equivalent yet; not planning a compatibility shim

### Jan 16 — Amp feature cleanup: inmem DB + session mirroring removed

- **Didier removed two amp-only features** (no objections from team):
    - **InMem DB**: replaced by app state tile; not supported by mixer
    - **Session mirroring**: unused, amp-only
- **Debug channel** clarified by Gerd: stdlib opens a debug port for remote-controlling apps via pub/sub; works in browser/desktop but not for service agents

### Jan 20 — Groups saved as catalog items

- Didier: groups can now be saved as catalog items (with syncing support)
- Significant complexity in reconnecting external bindings after sync
- PR merged; demo video shared

### Jan 21 — runtime.json size reduction (570KB → 106KB)

- **Problem**: runtime.json bloated by tailwind style views (full card-picker views stored for each style)
- **Fix**: strip style views from runtime.json; only needed for rmx-editor webcomp at runtime (edge case, deprioritized)
- PR: turntable/pull/11650
- Also discovered runtime.json duplicated in .remix files (once at top level, once in `/code/`) — old v1 format artifact; use `export_package` plugin for v2 format

### Jan 24 — Scenegraph JSON paste in L2

- Didier: direct pasting of scenegraph JSON format into L2 creates card tile
- PR: turntable/pull/11656
- **Clarification**: this is NOT a replacement for normal copy/paste — it's a shortcut for externally-generated content (AI, runtime)
- Full paste behavior catalog:
    - String/number → value tile
    - Rectangular JSON → table tile
    - Arbitrary JSON → object tile
    - Catalog URL → download + insert node
    - **New**: scenegraph JSON → card tile
    - **Missing (desired)**: HTML paste → card creation (via html2card)

### Jan 24 — Workspace plugin: export .remix to cloud workspace

- Wilber: added cloud workspace export option to workspace plugin
- Arvind found issue: export silently fails when "register" is selected for debug objects; works with "omit"
- Wilber investigating

### Jan 26 — appName vs dbName confusion

- Gerd asked if `appName` field was removed from runtime.json → Didier confirmed it's required but not critical
- **Broader issue**: dual naming (appName vs dbName) causes confusion in builder; backend prefers appName, frontend uses dbName
- John proposed making it an optional display name only; discussion ongoing

### Feb 6 — DragGroup decoder removal

- Simon removed DragGroup decoders (used for kanban boards); runtime support was already gone
- Old cards using DragGroup will no longer decode
- PR: turntable/pull/11692

### Feb 12 — L0 plugins + .remix file plugins

- Simon: L0 plugin system merged; enables adding repository apps directly to L0 dashboard
- PR: turntable/pull/11708
- Didier proposed reintroducing a plugin button in navbar (L0/L1/L2)
- Arvind: L0 doesn't need an edit menu; thinking about design
- Button merged but hidden (no dashboard plugins registered yet)

### Feb 13 — File node refresh button

- Simon: added refresh button to file node (needed by John and Wilber)
- Also fixed desktop cases where token was missing from HTTP requests
- PR: turntable/pull/11712
- Decided not to add more features to existing file nodes; file.register node being built separately

### Feb 16 — File Register node (new)

- Simon: new node exposing all `file.register` variants
- Existing File Upload action only exposed partial variants; will be replaced once Mix has native `blob://` URL handling
- PR: turntable/pull/11723

### Feb 18 — Remote Data Sources support

- Simon: button at L1 to access Remote Data Sources modal
- PR: turntable/pull/11730
- **Next step**: support multiple remote sources; remote data sources differ from remote catalogs — they're where users want to *run* apps, not just browse catalogs
- Didier: remote catalog list now synced via `_rmx_prefs` agents on each workspace (no longer in user preferences)
- **Agreed**: remote data source applies to the *entire app* (not per-agent)
- Simon and Wilber's tooling should be consistent for exchanging this data
- Early preview video shared (Feb 20)

### Feb 21–22 — Open-link behavior in builder (decision)

- **Problem**: builder navigates away when open-link action fires with "same window" target — user loses builder tab
- Current behavior: if unsaved changes → modal with options; if no changes → silently replaces tab
- **Decision**: always open links in a new tab in the builder, never navigate away
    - Desktop: external links always open in browser tab (desktop-first approach)
    - Web builder: same behavior for consistency; "open in place" has no real use case
- Simon to implement change

---

## PRs Referenced

| Date   | PR                   | What                                    | Who    |
|--------|----------------------|-----------------------------------------|--------|
| Jan 5  | turntable/pull/11625 | Remove mqtt SetLocal from runtime       | Simon  |
| Jan 20 | (merged, no PR#)     | Groups saved as catalog items           | Didier |
| Jan 21 | turntable/pull/11650 | runtime.json size 570KB→106KB           | Didier |
| Jan 24 | turntable/pull/11656 | Scenegraph JSON paste in L2             | Didier |
| Feb 6  | turntable/pull/11692 | Remove DragGroup decoders               | Simon  |
| Feb 12 | turntable/pull/11708 | L0 plugins + .remix file plugins        | Simon  |
| Feb 13 | turntable/pull/11712 | File node refresh + desktop token fixes | Simon  |
| Feb 16 | turntable/pull/11723 | File Register node (all variants)       | Simon  |
| Feb 18 | turntable/pull/11730 | Remote Data Sources modal at L1         | Simon  |

---

## Channel Tone & Patterns

- Primarily Simon and Didier driving builder features; Tyler reviews most PRs
- Feature announcements often include video demos or screenshots
- Amp-to-mixer migration is a steady theme: removing amp-only features, adding mixer equivalents
- Didier proactively cleans up unused/legacy features (inmem DB, session mirroring, DragGroup)
- Arvind consulted on UX/design decisions; responds async
- John, Vijay, Wilber tagged for product perspective on builder features
- Low-traffic channel (~1-2 messages per day); substantive posts with PR links
- Croissant emoji is Didier's signature reaction