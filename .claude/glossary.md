# Remix Studio Glossary

> Source: [Glossary](https://www.notion.so/3bea488f743e43f38c4b4c2b44d58000)
> Parent: Remix Documentation > User Guides and Tutorials

---

## Core Concepts

- **Remix Studio** — The editor (server-hosted, in-browser, or desktop). Often called "the builder" internally.
- **Mix** — Custom programming language backing Remix projects. Studio generates Mix code from visual editor tiles, compiled to bytecode executed by Remix runtimes. Custom function/action tiles let
  you write Mix directly. Full docs: [Mix Language](https://www.notion.so/259ede6504e34505982dde5dc4b63d10)
- **Flow** — The Remix term for what's colloquially an "app". A project has screens linked into flows; a flow is what a user experiences when using a published package on a surface.
- **~~App~~** — Deprecated for Remix runtime UIs. Use "Flow" instead. "Mobile app" still refers to the native app installed from an app store (which can run many Remix flows).

## Project Structure

- **Project** — Basic container of work. A project has a `database` (Project == `db`).
    - **Types:** Projects (colloquially "apps"), Libraries (Templates + Collections of Assets), Themes (global styles only), Packaged Projects
    - **Permanent ID** — Unique project identifier, hard to change (references between projects can break). Delete: move to Trash → delete from Trash to free up ID.
    - **Default modules** on creation: Settings, Constants, Styles, Symbols, Files
- **Library & Catalog** — A Library is a project type created in Studio. Moving towards housing Collections and Templates.
    - **Collections** — Groups of freestanding (unconnected) assets of any node type (Components, Cards, Data, Actions, Decisions, etc.). Assets from Collections are added as Nodes into a Screen.
    - **Templates** — Fully realised functional screens with layout and behaviour, used as-a-whole after configuration. Added as Screens into a Project.

## Navigation Levels (L0-L3)

Internal terminology — use the public names with external partners:

| Internal | Public Name                                            | Description                                 |
|----------|--------------------------------------------------------|---------------------------------------------|
| L0       | Projects Dashboard                                     | Landing page, list of project tiles         |
| L1       | Project Home                                           | Graph/list view of all Modules in a project |
| L2       | Screen/Agent/Component Graph                           | Graph view of a module                      |
| L3       | Editors (Card/Query/Data Tree/Table/Decision/Function) | e.g. "Card Editor", "Query Editor"          |

## Module Types (within a Project at L1)

- Screens
- Agents (Local Agents, Cloud Agents)
- Code Modules
- Plugins
- Styles
- Symbols
- Constants
- Settings
- Files (not a "Module" but exists in the same graph)

## Agents

- **Agent** — A "headless screen", a Mix module callable like a function or API endpoint. Cannot be stateful or interactive. Takes input params, returns output params. Can be invoked from
  screens/agents in same project or over HTTP.
- **Cloud Agent Server** — Server runtime for Remix programs supporting only agent invocations (not interactive sessions). Also supports file upload/storage/service, web hosting.
  Docs: [Cloud Agent Server](https://www.notion.so/1061d464528f80f18e46dd27895aeda2)
    - Divided into **workspaces** (namespaces for permissions and projects with agents). Per-workspace overhead is minimal.

## Module Anatomy & Bindings

- **Module** — Generic unit of work in a project (Screen, Agent, Component, Code Module, Plugin).
- Every module starts with an **In Params** node and an output node (e.g. **Screen** node for screens).
    - **In Params** — Data values passed into the module when it loads. Includes system params like the signed-in Remix Environment User.
    - **Screen node** — Accumulates the final view; everything wired into it gets displayed.
- **Home Screen** — Every project has one. Can't be deleted (reassign first). Default name `home` but can be any name. Users start here when running a published flow.
- **Bindpoint** — A connection point on a node. Toggle ON in the L3 Card Editor to expose a value for wiring in the L2 nodegraph. Bindpoints accept Data, Card, or Event types.
- **Binding** — A wire connecting an Out Bindpoint to an In Bindpoint. Data flows instantly along bindings (live reactive system).
- **Publish vs Package** — Publishing makes changes viewable on the Studio workspace (team testing URL: `{host}/{project}/{screen}`). Packaging creates a `.remix` file for distribution to other surfaces.

## Remix Database (Record Store)

> Source: [Step 8: Make Something Real](https://www.notion.so/1331d464528f8030ac34e057e2cef870)

Row-based, schemaless, JSON record store. Each project has its own database.

- **`entity`** field — Defines the "type of thing" (e.g. `person`, `company`). Crucial for organizing records.
- **`_rmx_id`** — System-assigned unique record ID (on save).
- **System fields** (all `_rmx_` prefixed, auto-created, cannot be manually set/forged):
    - `_rmx_created_by`, `_rmx_created_at` — Who and when
    - `_rmx_last_modified_by`, `_rmx_last_modified_at` — Last edit
    - `_rmx_content_hash`, `_rmx_rev` — Content integrity / revision
- **Record relationships** — Use `_rmx_id` values to link records across entity types (1:1, 1:N, M:N patterns).

## Build & Deploy Concepts

- **Executable** — Publishing a project creates a compiled, runnable version saved in the project's database. Consists of database records with bytecode + metadata. Programs can be statically or
  dynamically linked.
- **Remix file (.remix)** — ZIP packaging of a directory structure. Deployment mechanism from Studio to other environments (agent server, browser runtime, etc.). Can contain one or more projects.
  Generally contains only compiled executable assets.
- **Package** — Exporting compiled executable assets from Studio into a .remix file for deployment elsewhere. Can optionally include studio or user data.

## Nodegraph & Node Types

> Source: [Getting Started in Remix Studio](https://www.notion.so/1051d464528f8008b584dd33015fd5b3) — Steps 6, 7, 9
> Parent: Remix Documentation > Child pages

The L2 editor is an infinite-canvas **nodegraph** — a graph of nodes connected by wires. Nodes have a **binding interface**: **In Bindings** (left edge) accept data values and event triggers; **Out Bindings** (right edge) emit data, cards, or events. The system is **live** — data flows instantly along wires as you build.

### Node Color Coding

| Color | Category | Purpose |
|-------|----------|---------|
| Blue | Cards & Components | Render information, layout, design on screen |
| Yellow | Data | Declare and move data values through the system |
| Dark Green | Queries | Interact with the database |
| Light Green | Logic | Conditional decisions, state machines |
| Orange | Functions & Code | Textual code (Mix language, WASM) |
| Pink | Actions | Do things — navigate, save, call APIs, etc. |
| Yellow (flower) | Annotations | Code comments with markdown + `[[nodeNum]]` linking |

### Complete Node Catalog

**Cards & Components (Blue)**
- **Card** — Nested tree of containers + display primitives (Text, Icons, Dates, Images, User Inputs). Layout uses flexbox. Edited in L3 Card Editor.
- **Component** — Self-contained sub-graph (a Module). Has own nodegraph with all node types. Exposes **In Params** node (data/events in) and **Out Params** node (data/cards/events out). Reusable black-box. Components can nest.
- **Webcomponents (4)** — Standard web components (custom HTML tags with JS). One generic + three Remix embeds (embed entire Remix runtime inside another Remix flow). Escape hatch #2.

**Data & Transforms (Yellow)**
Object (Data Tree), Value, State, Get State, Table (List of Objects), List (of Values), Sort, Filter, Group By, Lookup Tables, Custom Transform, In Events, Config

**Queries (Dark Green)**
Query, Record, Advanced Query

**Decision & State Logic (Light Green)**
Decision, State Machine

**Functions & Code (Orange)**
- **Function** — Mix language code editor. Escape hatch #1.
- **Wasm** — WebAssembly bundles (C/C++/Rust compiled). Registered + hosted, found via Library. Escape hatch #3.

**Actions (Pink)**
Push to Screen, Navigate, Show Overlay, Open Link, Go Back, Reset View, Reload Screen, Close App, Set a Value, Set State, On Change, Postpone (agent-only: return initial value before complex calc), Delay, Toast, Save, Save Multiple, Delete, Delete Multiple, API Connect, Agent Connect, Publish, Subscribe, Custom Action, Get Location, Copy to Clipboard, Local Storage Set/Get, Share, Get Push Messaging Token, Messaging Status, Download File, Play Alert, Inject Local File, Upload File, Open Document, Mobile POS Print, SMS, SMS Permission, Reload Widgets, Experimental Client Action, Macros (4), Profiler

**Annotations (Yellow flower)**
Annotation — Markdown-formatted sticky notes. `[[10]]` creates a clickable link to Node #10.

### Three Escape Hatches

When built-in node primitives are insufficient:
1. **Function Nodes** — Write Mix code directly
2. **Webcomponents** — Embed custom HTML/JS components
3. **WASM Bundles** — High-performance compiled code (C/C++/Rust)

## UI/Editor Terms (pending formal definitions)

theme, styles (local/global), search panel, new panel, insert, asset, component, source, import, symbol, node, binding, bindpoint, element, card, tree, active binding, function, plugins, settings,
user preferences, api
