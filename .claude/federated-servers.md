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

## Key URLs

- `_rmx_search` source of truth: `https://remix.remixlabs.com/e/edit/_rmx_search`
- Hosted on GCS: `https://storage.googleapis.com/rmx-static/draft/_rmx_search.remix`
- Workspace management: `https://remix.remixlabs.com/cloud_workspace/workspaces`
