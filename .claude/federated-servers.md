# Federated Servers (Federated Search)

> Source: [Federated Servers](https://www.notion.so/6496884844e74752af5b94d933afb22a)
> Parent: Remix Documentation > User Guides and Tutorials > Topics > Catalog and Collections

---

## Overview

L2 search (and future L1) in the builder can be powered by external libraries:

- **Amp-based library** — An L0 project of type Design Library
- **Workspace-based library** — A Design Library hosted on Remix cloud infrastructure
- Plus local Design Libraries on the machine where the builder runs

## Setup

One-time setup requires deploying `_rmx_search.remix` (collection of search agents):

- **Amp setup:** Drag & drop .remix at L0, set access control to "all users" via Tools & Settings plugin > View access
- **Workspace setup:** Use workspace setup tool at `remix.remixlabs.com`, create workspace, go to App Hub, install "Federated Search" app

### Builder Configuration

In System Preferences, list federated servers to access:

- Default server: `community` (all libraries on `community.remixlabs.com`)
- Add workspace libraries: `https://agt-dev.remixlabs.com/ws/<workspace-name>`
- Add single library: `https://agt-dev.remixlabs.com/ws/<workspace-name>/<library-name>`

## Deploying Libraries

- **Amp:** Any Design Library on the machine shows in search results. Create at L0 or upload via drag & drop.
- **Workspace:** Export design library from builder (package with builder assets), upload via workspace setup tool > "Deploy app"

## Asset Sync (Push/Pull Model)

After initial .remix upload, updates use push/pull on individual assets:

Insert a remote asset via L2 search (Shift+Space). Footer sync menu provides:

1. **Sync from source** — Pull latest version from cloud
2. **Update source** — Push current L2 node to cloud
3. **Unlink from source** — Create independent copy
4. **Show/hide source binding** — Toggle source info binding (JSON of source endpoint)
5. **Info section** — Where the asset comes from

## Copy/Paste with References (L2SourceNode)

Clipboard type `L2SourceNode` holds a **reference** to a cloud asset (vs full AST copy):

```json
{
  "type": "L2SourceNode",
  "val": {
    "bindings": {
      "<binding-name>": "<value>",
      ...
    },
    "source": {
      "location": {
        "server": "https://agt-dev.remixlabs.com/",
        "type": "workspace",
        "ws": "<workspace-name>"
      },
      "dbName": "<library-name>",
      "modName": "<module-name>",
      "nodeKey": "<node-key>"
    }
  }
}
```

The `source` object comes from the source info binding of the remote asset.

## Published Remix Libraries

> Source: [Accessing Remix libraries](https://www.notion.so/13c1d464528f8039beb0c08aa8413722)
> Parent: Remix Documentation > Remix Catalogs and Toolkits

Remix publishes component libraries to a workspace `remix-libraries` on US production agent servers (`https://agt.remixlabs.com/`).

| Library       | Description                                                                                              | Federated Search URL                                        |
|---------------|----------------------------------------------------------------------------------------------------------|-------------------------------------------------------------|
| Remix Design  | Base Remix library of UI components, layouts and styles                                                  | `https://agt.remixlabs.com/ws/remix-libraries/_rmx_design`  |
| Remix HubSpot | Toolkit for building HubSpot-connected apps (also needs HubSpot Connector + Configurator apps/templates) | `https://agt.remixlabs.com/ws/remix-libraries/_rmx_hubspot` |
| Remix Drafts  | Components in draft stages of development and review                                                     | `https://agt.remixlabs.com/ws/remix-libraries/_rmx_drafts`  |

### Setup Steps

1. Copy library URLs from above
2. In Studio → click down arrow (upper right) → open **SYSTEM PREFERENCES**
3. Paste URLs into **Search Preferences**
4. Close SYSTEM PREFERENCES
5. **IMPORTANT:** Refresh the browser page to trigger federated search cache refresh

**Note:** This is distinct from the `remix_labs` catalog library (`https://agt.remixlabs.com/ws/remix_labs`) used for service agents, AI agents, and internal assets. The `remix-libraries` workspace
hosts the externally published UI component libraries.

---

## Key URLs

- `_rmx_search` source of truth: `https://remix.remixlabs.com/e/edit/_rmx_search`
- Hosted on GCS: `https://storage.googleapis.com/rmx-static/draft/_rmx_search.remix`
- Workspace management: `https://remix.remixlabs.com/cloud_workspace/workspaces`

---

## Catalog and Collections

> Source: [Catalog and Collections](https://www.notion.so/ebc636c1e547468f80944acbbf751d70)
> Parent: User Guides and Tutorials > Topics

### Overview

A **catalog app** (Notion page, full Remix app, or custom website) lets users:
1. Browse and search asset collections (components, cards, function nodes, etc.)
2. Configure assets via an embedded .remix configuration app
3. View showcase examples
4. Insert configured assets into the builder

Builder L2 sidebar search is a lightweight catalog — shows a green `Configure` button if the component has a configurator app linked.

### Configurator App Pattern

A standard Remix app exported to GCS via "Host on GCS". Two sections:

**Examples** — pre-configured instances of the component:
- Data node (binding values) + component with bindings connected + source info revealed + function node combining source + bindings + copy/insert button

**Build your own** — form elements → live preview → copy/insert:
- Form elements wired to component in-bindings
- "Reveal source info" in node footer sync menu → new `source` outbinding
- Function node assembles the L2SourceNode JSON:
  ```json
  {
    "type": "L2SourceNode",
    "val": { "source": source, "bindings": bindings }
  }
  ```
- **Copy/Insert button**: from `community` federated library, search `catalog copy/insert`
  - Input: `content` binding (the L2SourceNode JSON)
  - Detects env via `client_surface` variable: `plugin` → insert into screen; `app` → copy to clipboard
  - App creator doesn't need to handle this distinction

### Component Metadata Setup

1. Footer menu → "edit node metadata"
2. Paste GCS .remix link into **Node Configurator** section (just the .remix URL, not the full runner URL)
3. Optionally add custom "Catalog page" menu pointing to the Notion/hosted catalog page
4. Push to library: footer → "update source"
5. Once set: `Edit` button becomes green `Configure` button; L2 search shows `Configure` button

**Catch 22 on first setup:**
- Host configuration app on GCS first → get the GCS link → update component metadata → update source → resync instances in config app → republish app on GCS (one-time only)

### Catalog Page Setup

- Notion page with embedded .remix runner using `?src=<GCS link>`
- Target specific screen: add `&screen=<screen-name>` to embed URL
- Universal runner: `https://remixlabs.com/run.html?src=<GCS link>`
- Example icon component: [Example of a catalog page for an Icon Component](https://www.notion.so/f806cab33faf4872810f9fbe04c654d1) (embed only, no text)
  - Builder: `https://remix-beta.remixlabs.com/e/edit/catalog-page-icon-component/home`
  - GCS .remix: `https://storage.googleapis.com/rmx-static/draft/catalog-page-icon-component.remix`

---

## Faceted Search

> Source: [Faceted Search](https://www.notion.so/10d1d464528f80ad8d5bf181ea499394)
> Parent: Catalog and Collections

Side panel alongside regular search results in L2 node search (Shift+Space).

### Filter Sections

| Filter | Description |
|--------|-------------|
| `with configurator` | Only items with a linked configurator app |
| `with preview` | Only items with a visual thumbnail |
| `Location` | Hierarchical: server → library → collection |

- Selections on left panel update right panel results, and vice versa
- Numbers next to each section auto-update as you type in search

---

## Catalog V2

> Source: [Catalog V2](https://www.notion.so/1a51d464528f80db82b8d7d4555f117a)
> Parent: Catalog and Collections

New workspace-based catalog implementation (deployed Q1 2025), supersedes Catalog V1 (mid-2024). **Not compatible with V1** — requires migration.

### Benefits over V1

- Assets movable between libraries (V1: tied to library + screen)
- Libraries created on the fly (V1: only at .remix import time)
- Each asset is URL-addressable with a stable ID (stable within a workspace)
- Assets can have **tags** (replaces the V1 "collections" concept)
- Opens door to group assets and screen/agent-level assets (V1: nodes only)

### Setup

**Workspace:**
1. Cloud Workspace Tool → App Hub → install **Federated Search** app
2. Workspace can now host library assets

**Builder:**
1. Avatar menu → System Preferences (or search panel settings icon)
2. Add library URLs (one per line, format: `<server>/ws/<workspace>`)
3. Start with: `https://agt.remixlabs.com/ws/remix-libraries`

### Migration (V1 → V2)

**Assets you have locally:**
- Use **"Bulk Publish Selected"** tool → select nodes → choose target V2 library → add tags → publish

**Assets only in V1 library:**
- Use **"Catalog V1 Migration"** tool (avatar menu) → download V1 collection as local screen → then use Bulk Publish

**Migrating screens that use V1 assets:**
- Open screen → migration banner appears with two counts:
  - **Orange (auto-migrate)**: unique name match in V2 → press orange button or migrate node-by-node via sync menu
  - **Red (manual)**: ambiguous or missing in V2 → use swap mode search or contact asset maintainer
