# Remix Studio User Guide — Servers, RBAC, External Actions, Modes

> Notion: [Remix Studio User Guide](https://www.notion.so/1051d464528f8007a5f5d40969c49427)
> Subpages: Nodes (In/Out params, Cards, Data Objects, Queries, Decisions, Functions, Actions, Annotations), Cloud Agent Server, Studio Server, Self-hosted Server, Remix Database, Working with State,
> Anonymous access, Mixed Auth, Macros, External Actions, iOS Widget/Shortcut Agents, Edit/Run/Configure modes, Service Agent modules

---

## Nodes (L3 Editor)

Nodes are the primary building blocks within a module (L2).

### Parameters (In/Out)

- **Reordering**: Bindings in in/out-params can be reordered using up/down arrows. Publishing updates the order in connected Push, Navigate, and Agent Connect nodes (automatic "patching").
- **Agent Errors**: Agent out-params include two special bindings: an **error trigger** and an **error string**. Invoking the `return error` binding causes the caller's `error` next-binding to fire.

### Cards & Components

- **Style Units**: Support for `px`, `em`, `rem`, `%`, etc. (added Dec 2024).
- **Text Inputs**: `debounce` property defaults to 150ms.
- **Headless Components**: Components that return only data (no card UI). Enabled via "Set as Headless" in In/Out-Params. Format changes to show bindings more clearly when used.
- **Triggered Components**: Advanced components for the `triggerable` runner (part of `com_remixlabs_simon`). Have no action in/out bindings.

### Queries (Query Builder 2.0)

Introduced Oct 2025 with major DB engine and builder improvements:

- **Rich Projections**: Get all data in one query to avoid multiple round-trips.
- **No Post-processing**: All Mix-based post-processing must be handled by users; function projections are no longer supported in the DB engine for maximum speed.
- **Migration**: QB 1.0 nodes can be migrated to 2.0 (revertible until leaving module). 1.0 support remains for legacy needs.
- **Live Queries**: "Live" mode on Query tiles enables auto-refresh when the underlying database changes.

### Database Actions

- **Save**: Create or update records.
- **Delete**: Requires DB location and record `ref`. Includes error bindings (since Oct 2024) for failure handling.

### Agent Connect Tiles

- **Service Agents**: Require Cloud URL and workspace. "Refresh" fetches available Apps/Agents.
- **Local Agents**: Run on the same server as the caller (Amp or Cloud).
- **Client-side Execution**: Optional toggle to run the call from the client environment (useful for reaching `localhost` MCPs).

---

## Working with State

Remix supports three scoped state types:

1. **Local State**: Scoped to a single level in the component graph.
2. **Screen State**: Available throughout the component stack of a screen (for assembly).
3. **App State**: Global state that persists across navigation.

### State Nodes & Actions

- **State Node**: An object cell for reading/writing. Initial values on the main in-binding are only used once; subsequent updates require `Set State`.
- **Set State**:
    - **Static**: Select fields from a dropdown in the green navbar.
    - **Dynamic**: Specify a "Field" name or a "Path" (array of keys/indexes like `["output", "name"]`).
- **Get State**: Acts as a preview of the state value if no specific field is selected.

---

## Macros (Builder Remote Control)

Macros allow a Mix program to orchestrate the Studio Builder (e.g., creating nodes, connecting bindings).

### Invocation Methods

1. **Macro Tiles**: Individual action tiles (e.g., Node Search, Reload Graph).
2. **Macro Runner**: Takes a list of macro objects (created via the Mix `macro` module) and runs them sequentially.

### Key Macro Commands

- **Node/Screen Search**: Triggers library sidebar for asset selection.
- **Create/Clone/Move/Rename Node**: Manipulation of the L2 graph.
- **AddConnection / AutoConnect**: Programmatic binding management.
- **Make Agent**: Compile and deploy a screen/agent to a workspace.
- **ArtifactSave / ArtifactSaveAs**: Customizing the save dialog in artifact mode (supports `autosave: true`).
- **EditComponent / DoneEdit**: Programmatic navigation into/out of component L3 editors.

---

## External Actions Reference

External actions are async local function calls from the Mix runtime to the embedding environment (Browser or Flutter).

### Key Built-in Actions

- **remix/test**: Connectivity test (echoes args + timestamp).
- **remix/logout**: Logs out the user; optional redirect URL.
- **remix/keyboard**: Listen for key combos (e.g., `ctrl+shift+p`) using `ctrl-keys`.
- **remix/local-storage/***: Namespaced (`rmx-local-data--`) access to device storage.
- **remix/mobile/esc-pos-print**: Print to thermal printers (supports text, images, QR codes, rows).
- **remix/mobile/share-***: iOS sharesheet integration (invoke agents, render cards to PNG/PDF).
- **remix/store/***: In-app purchase management (current-plan, purchase, restore).

---

## Anonymous Access

- **Configuration**: Screens marked "Anonymous" via footer menu show as green in the builder.
- **Auth Flow**: Transition to logged-in state uses the `_rmx_auth` screen.
- **Requirement**: The embedding environment must implement the `remix/user-signin` Client Action.
- **Testing**: Use `remixlabs.com/run.html` (builder emulation is incomplete).

---

## Service Agent Modules

- **Edit Cloud Server**: Navbar button to point all queries/saves in a module to a remote cloud server for easier development against live data. Does not affect final generated code.
