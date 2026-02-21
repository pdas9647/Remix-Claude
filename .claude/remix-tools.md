# Remix Tools & Process — Bug Reporting, MCP, Cloud Workspace, Repository

> Split from [remix-infra.md](./remix-infra.md).
> Parent Notion pages linked per section below.

---

## Bug Reporting Process

> Source: [Bug reporting](https://www.notion.so/3061d464528f80cdacf7eed2612bad07)

### Step 1 — Is it a bug?

Ask in `#bugbash` or `#dumbquestionsanswered` Slack channels. Search existing GitHub issues. If known bug, optionally add context to GitHub issue and stop.

- Content bugs: track resolution on Slack (no formal process yet)

### Step 2 — Provide a repro

For builder bugs especially. Most useful:

- Link to cloud builder screen exhibiting the bug
- Minimal `.remix` file, or step-by-step from scratch
- Product/surface + version + OS + browser
- If no repro: what you were doing + exact time (for server log lookup)

### Step 3 — Fix it or file it

Engineering team either: (a) works immediately + links PR in thread, or (b) creates GitHub issue and links it. Initial reporter is responsible for ensuring one happens.

### Step 4 — Mark resolution

React to original `#bugbash` post:

| Reaction | Meaning                          |
|----------|----------------------------------|
| ✅        | Fixed                            |
| 👍       | Expected behavior                |
| 📝       | Ticket created / already existed |
| 🚧       | Content bug, working on it       |

---

## Tasks — Asset Submission Guide

> Source: [Tasks - Asset Submission Guide](https://www.notion.so/2111d464528f8031a05fd76e687b6379)

Process for submitting artifacts (flows, builder nodes, auth configs) for review and potential publication to the Remix catalog or library.

### Prerequisites

1. Install the **"Tasks"** app via **Workspace Tools** builder plugin → **App Hub**
2. Configure Tasks to point to your workspace:
    - **Prod US:** `remix.remixlabs.com/tasks/start`
    - **Prod India:** `remix-india.remixlabs.com/tasks/start`
3. Go to **Set workspace** → enter the workspace for storing/managing tasks

### Supported Artifact Types

| Type                   | Description                                            |
|------------------------|--------------------------------------------------------|
| **Auth Configuration** | JSON payload of an auth configuration                  |
| **MCP Tools**          | A .remix file containing agents accessible through MCP |
| **Builder Asset**      | A builder node (screen, agent, component, etc.)        |

### Workflow

1. **Create** — New task with name and description (starts as "backlog")
2. **Attach** — Add artifacts via the corresponding button
3. **Complete** — Mark task as complete
4. **Submit for review** — Fill review form (name/description for reviewer). Submitted to **`remix_review`** workspace

### Important Notes

- Don't submit more than once unless necessary (no duplicate safeguard)
- MCP .remix files must include MCP records with `_rmx_type = mcp_tool` already packaged

---

## MCP Tools — Installation Guide

> Source: [MCP Tools Installation Guide](https://www.notion.so/2141d464528f807486ebee2a6b261cfe)
> Parent: [MCP Tools](https://www.notion.so/2141d464528f8070b79bfdece10ff94f)

Prerequisite: Latest Remix Desktop app installed and running.

### Steps

1. **Create the service** — Build your MCP service agent
    - Example: `remix.remixlabs.com/e/edit/wc_tools/pong` (returns "pong")

2. **Generate MCP configuration** — JSON schema for your service
    - Use the library asset for auto-generation: `remix.app/remix/asset?source=https://agt.remixlabs.com/ws/com_remixlabs_wilber/CNbG46sMWSbhQg5j4iEkkcj26Y2wRGG6`
    - Pass `record` parameter (example of valid input params) → most of the JSON schema auto-generated
    - Review for accuracy; add tool name and agent name where needed

3. **Save MCP configs** — Save into the same project as the services, with `_rmx_type = mcp_tool`
    - Example: `remix.remixlabs.com/e/edit/wc_tools/mcp_tools`
    - Alternative install method: Workspace plugin tool → navigate to target workspace → "Deploy App" → enable "Include MCP Tools" → Deploy

4. **Restart Claude** — Tool appears after restart. Test by invoking the tool.

---

## MCP Tool Prompting — Best Practices

> Source: [Tool Prompting](https://www.notion.so/2141d464528f80beb14be0ffecf31bf7)

Guide for writing effective MCP tool descriptions. Tool prompts must focus on precise behavior specification and parameter handling.

### Required Sections (in order)

| Section            | Purpose                          | Example                                                                                                        |
|--------------------|----------------------------------|----------------------------------------------------------------------------------------------------------------|
| `[TOOL PURPOSE]`   | What the tool does               | "Generates a summary of a Parquet file from a URL using DuckDB."                                               |
| `[WHEN TO USE]`    | Scenarios to invoke + boundaries | "Use when users need to understand structure of a remote Parquet file. Do not use for detailed data analysis." |
| `[OUTPUT]`         | What data is returned            | "Returns column info (names, types, null %, sample values) + file metadata (total rows, columns)."             |
| `[PRESENTATION]`   | Formatting instructions          | "ALWAYS format as markdown table with file metadata bullet points above."                                      |
| `[ERROR HANDLING]` | How to handle/communicate errors | "Do NOT attempt workarounds. Translate errors clearly for users."                                              |

### JSON Tool Definition Format

```json
{
  "agent": "<agent-name>",
  "app": "<app-name>",
  "description": "[TOOL PURPOSE]\n...\n\n[WHEN TO USE]\n...\n\n[OUTPUT]\n...\n\n[PRESENTATION]\n...\n\n[ERROR HANDLING]\n...",
  "inputSchema": {
    "properties": {
      "<param>": {
        "description": "<specific format, constraints, context>",
        "type": "string"
      }
    },
    "required": ["<param>"],
    "type": "object"
  },
  "name": "<tool-name>"
}
```

### Best Practices

- Use clear section headers — creates scannable structure
- Keep sections focused (one aspect each, no overlap)
- Maintain consistency across all tools (same section structure)
- Be specific about boundaries (what it does AND what it does not)
- Parameter descriptions: specify format, validation, common pitfalls, context

---

## Remix Tools

> Source: [Remix Tools](https://www.notion.so/13b1d464528f80d593a7f1de1e0d6d43)
> Parent: Remix Documentation > User Guides and Tutorials

Container page for builder-integrated tools. Child pages: Cloud Workspace Tool, Repository Tool.

### Cloud Workspace Tool

> Source: [Cloud Workspace Tool](https://www.notion.so/12c1d464528f809b9f58e6a707b93fe1)

Tool for managing cloud workspaces. Available as a **builder plugin** (Plugins menu → "Cloud Workspace POC") or on the [web](https://remixlabs.com/app.html?src=https://agt.files.remix.app/remix/files/poc/cloud_workspace_poc.remix).

**Prerequisite:** Access to a cloud workspace (request from Remix admin if needed).

#### Screens

| Screen                    | Purpose                                                                                                                                                                               |
|---------------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| **Workspace entry**       | Enter workspace ID to manage. Pin button to auto-navigate on next open.                                                                                                               |
| **Workspace overview**    | Menu of management options. Cloud button returns to entry screen.                                                                                                                     |
| **Deploy App**            | Install apps from **Builder** (last published state, can include builder assets for library hosting) or **Hosted** (.remix URL)                                                       |
| **Workspace permissions** | Search users by email, view/add workspace-level roles. Add multiple users (one email per line).                                                                                       |
| **App Hub**               | Install pre-configured apps: **Files** (file management), **Federated Search** (component search from workspace libraries). Blue icon = not installed, Green icon = update available. |
| **App overview**          | Per-app user permissions, agent permissions, delete app (requires confirmation code).                                                                                                 |
| **Agent permissions**     | Per-agent access types: **Whitelist** (specific auth'd users), **Anonymous** (anyone), **Authenticated** (any auth'd user). Agents without permissions don't appear.                  |

---

### Repository Tool

> Source: [Repository Tool](https://www.notion.so/2e91d464528f800985eee93ec9fd5842)

Version control system for builder projects. Repository data lives in a workspace. Pre-configured Remix Labs internal repository: `agt.remixlabs.com` / `repository`.

#### Concepts

- **Granularity of comparison:**
    - Screens, agents, widgets → individual **tiles** (e.g. a card tile as a whole)
    - Code modules → each module as a whole
    - Files → each file as a whole
    - App metadata → all metadata together as one unit
- Database records **cannot** be checked in
- Each revision records: who, when, commit description

#### Four Plugins

| Plugin             | Purpose                                                                                                                                                |
|--------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------|
| **create_project** | Check in a local builder project as a new repository project. Also used to switch to a different repository.                                           |
| **list_projects**  | Browse repository projects, view revisions, edit project details, clone head or specific revision.                                                     |
| **pull**           | Download changes from repository. Two modes: **Discard** (reset local to match repo exactly) or **Keep** (merge non-conflicting local changes).        |
| **push**           | Upload local changes to repository. Conflict prevention: must pull first if another contributor pushed since your last sync. Supports commit comments. |

#### Workflow

1. **Create** or **Clone** to connect a local database to a repository project
2. **Push** to upload changes (with commit message)
3. **Pull** to download others' changes
4. After Clone/Pull: **must run "Rebuild & Make"** from the Make menu (only builder assets are checked out, not generated code)
5. Known bug: after Rebuild & Make, some screens may flag red — run a plain "Make" to fix

#### Clone Warning

Cloning **overwrites/deletes** the current database contents. Always create a new builder project before cloning.
