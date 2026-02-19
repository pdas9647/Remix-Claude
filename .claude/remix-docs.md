# Remix Documentation — Glossary, Deployment, Mix Language, Syncing, Deeplinks, Preferences

> Notion: [Remix Documentation](https://www.notion.so/fe31096a438b4f29b103c189bc9a7fb8)
> Subpages: Glossary, Getting Started, Studio user guide, Cloud Workspace Tool, Deploying, iOS/Android widgets, Component overriding, Catalogs & Toolkits, Mix Language, .remix file syncing, Deeplinks,
> Synced preferences

---

## Glossary (User-Facing Terms)

> Notion: [Glossary](https://www.notion.so/3bea488f743e43f38c4b4c2b44d58000)

### Studio Hierarchy (L0–L3)

Internal terminology → external terminology:

| Level | Internal | External Name                                                        | Description                                    |
|-------|----------|----------------------------------------------------------------------|------------------------------------------------|
| L0    | L0       | **Projects Dashboard**                                               | Landing page; list of project tiles            |
| L1    | L1       | **Project Home**                                                     | Graph/list view of all modules in a project    |
| L2    | L2       | **Screen/Agent/Component Graph**                                     | Graph view of a module                         |
| L3    | L3       | **Editors** (Card / Query / Data Tree / Table / Decision / Function) | The editing interface for a specific node type |

### Key Concepts

- **Remix Studio**: The editor (server-hosted, in-browser, or desktop). Often called "the builder".
- **Project**: Basic container of work. Types: Projects (apps), Libraries, Themes, Packaged Projects. Project == `db`.
- **Flow**: What a user experiences when using a published package on a surface. The Remix term for "app" (to avoid overloading that term). **"App" is deprecated** for Remix runtime UIs.
- **Library**: A type of Project housing Collections (groups of standalone assets of any node type) and Templates (fully realized functional screens usable after configuration).
- **Module types**: Screens, Agents (Local/Cloud), Code Modules, Plugins, Styles, Symbols, Constants, Settings. Projects also have Files.
- **Mix**: Custom programming language. Studio generates Mix code from visual editor tiles → compiled to bytecode → executed by runtimes.
- **Executable**: Publishing a project creates a compiled runnable version saved in the project's DB. Programs consist of bytecode records + metadata. Can be statically or dynamically linked. By
  default includes all screens + agents; can also compile standalone executables for subsets.
- **.remix file**: ZIP packaging of a directory structure. Packaging mechanism for Remix programs + data. Used to deploy compiled projects from Studio to other environments (agent server, browser
  runtime, etc.). Can contain multiple projects. Generally contains only compiled executable assets (most recent versions, no history).
- **Package**: Exporting compiled executable assets from Studio into a .remix file for deployment.
- **Agent**: A "headless screen" — callable like a function/API endpoint, cannot be stateful or interactive. Takes input params, returns output params. Can be invoked from same project or over HTTP.
  Deployable on Studio server or dedicated cloud agent server.
- **Cloud agent server**: Server runtime supporting only agent invocations (not interactive sessions). Also supports file upload/storage/hosting. Divided into workspaces (namespaces for permissions
  and projects).

---

## Deployment Surfaces

> Notion: [Deploying Remix applications](https://www.notion.so/11b1d464528f8030a107dc7f5e8d27cc)

10 deployment targets:

1. Flow in Remix container app (iOS/Android)
2. iOS or Android widgets
3. Full screen web page
4. Embedded in someone else's web page
5. Flows in Remix Chrome extension
6. iOS native app clip
7. Cloud agents
8. ISV — your own mobile app with Remix SDK
9. Embed in other ecosystems
10. Embedded widget in a web portal

**Patching**: Remix apps are modifiable at runtime via shareable patches — end users can configure and remix their own UI adjustments and extensions.

---

## Catalogs and Toolkits

> Notion: [Remix Catalogs and Toolkits](https://www.notion.so/e6a31848c0794c738fb9dc9b88041847)

### Library Locations

Libraries live on workspace `remix-libraries` at `agt.remixlabs.com`. Add these URLs to Studio → System Preferences → Search Preferences (then refresh browser):

| Library       | URL                                                         | Description                         |
|---------------|-------------------------------------------------------------|-------------------------------------|
| Remix Design  | `https://agt.remixlabs.com/ws/remix-libraries/_rmx_design`  | Base UI components, layouts, styles |
| Remix HubSpot | `https://agt.remixlabs.com/ws/remix-libraries/_rmx_hubspot` | HubSpot connector toolkit           |
| Remix Drafts  | `https://agt.remixlabs.com/ws/remix-libraries/_rmx_drafts`  | Components in draft/review stage    |

### Available Catalogs

| Catalog                         | Description                                                                        |
|---------------------------------|------------------------------------------------------------------------------------|
| **Base**                        | Table component, Charts, Voice Chat headless                                       |
| **HubSpot toolkit**             | Assets for HubSpot-connected apps (requires HubSpot Connector + Configurator apps) |
| **SMB**                         | Library assets for small/midsize business applications                             |
| **Business Profile / About Us** | Digital business profile assets                                                    |
| **Integrations**                | Integration-specific assets                                                        |
| **AI**                          | AI-related assets                                                                  |
| **DuckDB**                      | DuckDB integration                                                                 |
| **OAuth Handler**               | OAuth flow handling                                                                |
| **Snowflake — Cortex Search**   | Snowflake Cortex Search integration                                                |

OAuth documentation also available for: Desktop, Chrome Extension, and Widgets surfaces.

---

## The Mix Programming Language

> Notion: [The Mix Programming Language](https://www.notion.so/259ede6504e34505982dde5dc4b63d10)
> Key
> subpages: [Mix Syntax](https://www.notion.so/1061d464528f81d492f4e1b0f8c8adf4), [Type system](https://www.notion.so/1061d464528f81c0ba7bdc8fc2ce8556), [Standard Library](https://www.notion.so/1061d464528f8010b0cfc60836c20290), [Builtin operations](https://www.notion.so/1061d464528f81a5a5bbff09ca40f2cf)
> Additional topics: cells, links & aliases, case types, recursion, streams, pattern matching, modules, action closures, directives, libraries & executables, JSON changes, imperative, embedding
> viewstacks
> Special screens: `_rmx_init`, `_rmx_auth`, `_rmx_error`, `_rmx_entry`, `_rmx_debugShell`
> Extra core libraries: `mixc/parsing` (CSV), `mixc/driver` (compiler), `mixc/agents`

### Language Characteristics

- **Typed with two modes**: Non-strict (default, inserts runtime type assertions, broadens types) and strict (`_pragma_ {strictTyping:true}`, no auto-assertions, types kept minimal)
- **Spreadsheet-based**: Core abstraction is **cells** (reactive, propagating values) rather than imperative variables. Cells auto-update when dependencies change.
- **Functional-first, imperative-capable**: Lambda functions, pattern matching, currying. Imperative via `do` blocks, `var` bindings, `while`/`for` loops.
- **Custom bytecode**: Compiles to QCode or Wasm, executed by Mix VM (`mixrt`)
- **Data-oriented**: `data` type represents all serializable values (null, bool, number, string, arrays, maps). Non-data types: functions, streams.

### Key Constructs

| Construct   | Syntax                                          | Description                                                          |
|-------------|-------------------------------------------------|----------------------------------------------------------------------|
| Module      | `module M ... module end`                       | Namespace for cells, defs, types. Can import other modules.          |
| Cell        | `cell c = <expr>`                               | Propagating (reactive) value. Auto-updates on dependency change.     |
| Var cell    | `var cell c = <expr>`                           | Mutable cell, assignable in actions.                                 |
| Def         | `def f(x) = <expr>`                             | Named value/function definition.                                     |
| Action      | `action(v) { ... }`                             | Closure with access to app state; can mutate var cells and write DB. |
| Case        | `case red` / `case rgb([number,number,number])` | Tagged union constructor.                                            |
| Observer    | `on x do { ... }`                               | Runs action when cell is evaluated. Supports `when` guards.          |
| Initializer | `initializer { ... }`                           | Runs once after module var cells are defined.                        |

### Type System Highlights

- **Simple types**: `null`, `bool`, `number`, `string`, `word` (number|string), `data` (all serializable), `any` (top type)
- **Containers**: tuples (`[string, number]`), arrays (`array(number)`), labeled tuples (`[#name: string, #age: number]`), records (`{field: type}`), maps (`map(type)`)
- **`undefined`**: element of all types; bubbles through operators; test with `x?`
- **Case/union types**: `type color = red | green | blue | black`; cases can carry params (`case euro(number)`)
- **Type variables**: `T` (unconstrained), `_` (anonymous), `T@datarange` (constrained to data)
- **Type assertions** (`type(T, expr)`): runtime check, generates code. **Type hints** (`x : T`): compile-time only, no codegen.
- **Foreign functions**: `foreign def f: <type> = <name>` — implemented in host environment (amp/groovebox/mixrun), with optional per-environment fallbacks

### Cell Special Notations

- `x'` — previous value of cell x
- `cellVersion(x)`, `cellAge(x)` — version/age tracking
- `link(x)` / `contents(lnk)` — create/dereference cell links
- `onRefresh` / `onView` — trigger recomputation on refresh/view-show
- `newConstCell(expr)` / `newVarCell(expr)` — dynamically create cells

### Host Environments (for foreign function fallbacks)

- `builderEnvironment` — Amp/builder REPL
- `serverEnvironment` — Amp/not builder REPL
- `webEnvironment` — client VM/groovebox JS
- `widgetEnvironment` — Rust/mixswift

---

## .remix File Syncing

> Notion: [Mobile/Desktop .remix file syncing](https://www.notion.so/25d1d464528f806b8c82e873d275aab7)

### Core Concepts

**App**: A .remix file hosted on a remote server, defined by JSON:

```json
{
  "dbName": "_rmx_home",
  "appName": "Remix Home"
}
```

- Storage types: `gcs/versioned` (default, via Publish & List), `gcs/channel` (via Host on GCS), `url` (exact URL)

**Group**: A list of apps for a surface. Defines: `surface` (mobile/desktop), `name` (unique per surface), optional `home` (home screen in `dbName/screenName` format).

**Subscriptions**: A user subscribes to (in order):

1. Global **system** group (`_rmx_home`, `_rmx_widgets`, etc.)
2. Per-workspace **default** group (org-specific)
3. Optional named groups (marketing, sales, hr...)
4. **User** group (personal, identified by email) — always last

Last group defining `home` wins (user > default > system).

### GCS Storage Paths

- **Publish & List**: `rmx-static/packaged/files/<app>/<version>/<app>.remix` + version pointer at `rmx-static/packaged/<semver>/<channel>/<app>.json`
- **Host on GCS**: `rmx-static/<channel>/<app>.remix` (no version pointer)

### _rmx_sync Agents

Deployed on the `_rmx_sync` app in a mixer workspace.

**User agents:**

- `get_apps` — input: `{channel, version, surface}` → output: `{channel, version, surface, user, groups}` with fully resolved URLs
- `set_app` — set apps for current user's personal group. Input: `{surface, home, apps}`
- `set_user_groups` — set custom group subscriptions. Input: `{groups}` (array of names)
- `get_apps_for_group` — get apps for a single named group (security: email groups restricted to own user)

**Admin agents:** `get_apps_admin`, `set_app_admin`, `set_user_groups_admin` — same as user agents but can target other users/groups.

### Channel Promotion

Apps published via Publish & List default to `draft` channel. Promotion via Content Catalog in Tools & Settings:

- **Draft → Test**: copies version pointer JSON (e.g., `rmx-static/packaged/1.43/draft/app.json` → `rmx-static/packaged/1.43/test/app.json`)
- **Test → Public**: same copy mechanism

### Platform Version Promotion

When platform version bumps (e.g., 1.43 → 1.44), all pointer files must be copied. Done manually via scripts in `flutter-runtime` repo: `bin/download_packaged_folder.sh` +
`bin/upload_packaged_folder.sh`.

### Mobile Bootstrap Flow

1. **First start** (after auth): invokes `get_apps` → downloads all core apps → sets home page (`_rmx_home/home`) → blocks until complete
2. **Restart/foreground**: invokes `get_apps` → downloads only modified apps (using `if-modified-since` header) → installs in background with progress bar

Setup screen: `remix.remixlabs.com/e/edit/_rmx_sync/setup`

---

## Deeplinks

> Notion: [Mobile/Desktop deeplinks](https://www.notion.so/27a1d464528f803db94ac980a0bd84eb)

**Unified (mobile + desktop):**

- Download & open .remix: `com.remixlabs.runtime://localhost/run-remix-file?url=<url>&workspace=<ws>&app=<app>&screen=<screen>&params=<params>`
    - Downloads, shows confirmation screen, installs locally, runs
- Open installed screen: `com.remixlabs.runtime://localhost/open-screen?workspace=<ws>&app=<app>&screen=<screen>&params=<params>`
    - If screen omitted, defaults to `home`

**Desktop only:**

- Open artifact: `com.remixlabs.runtime://localhost/open-artifact?url=<url>&mode=<mode>`
    - Modes: `configure` (default, switches to `run` if no config node), `run`

`workspace` parameter is only meaningful on desktop.

---

## Synced Preferences

> Notion: [Synced preferences](https://www.notion.so/2871d464528f80ae96dfeff9a738ca67)

### Structure

Preferences are name/value pairs (names can be namespaced, e.g., `address.mailing.city`).

**Three-layer override chain** (each takes precedence over previous):

1. **System prefs** — compile-time constants (e.g., default `search.libs` pointing to `remix_labs` catalog)
2. **Default prefs** — per-workspace, set by admin (dynamic)
3. **User prefs** — per-user (dynamic)

### _rmx_prefs Agents

Deployed on `_rmx_prefs` app in the user's org workspace.

- `get_prefs` — returns merged prefs (system + default + user). No input.
- `set_prefs` — merge-update user prefs. Input: `{prefs}` hashmap. Pass `null` to delete a key. Returns full merged prefs after change.
- `set_default_prefs` — admin-only. Same interface as `set_prefs` but updates workspace defaults. Pass `null` to reset to system value.

```bash
# Example: fetch prefs
curl -H "Authorization: Bearer $REMIX_TOKEN" \
  "https://agt.remixlabs.com/run-agent/<workspace>/_rmx_prefs/get_prefs" -d '' | jq
```
