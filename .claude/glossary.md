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
    - **Types:** Projects (colloquially "apps"), Libraries (Templates + Collections of Assets), Themes, Packaged Projects
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

## Build & Deploy Concepts

- **Executable** — Publishing a project creates a compiled, runnable version saved in the project's database. Consists of database records with bytecode + metadata. Programs can be statically or
  dynamically linked.
- **Remix file (.remix)** — ZIP packaging of a directory structure. Deployment mechanism from Studio to other environments (agent server, browser runtime, etc.). Can contain one or more projects.
  Generally contains only compiled executable assets.
- **Package** — Exporting compiled executable assets from Studio into a .remix file for deployment elsewhere. Can optionally include studio or user data.

## UI/Editor Terms (pending formal definitions)

theme, styles (local/global), search panel, new panel, insert, asset, component, source, import, symbol, node, binding, bindpoint, element, card, tree, active binding, function, plugins, settings,
user preferences, api
