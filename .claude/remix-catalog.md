# Remix Catalog — Widgets, Libraries, Connectors, GTM

> See also: [federated-servers.md](./federated-servers.md) (federated search, published Remix libraries), [remix-snowflake.md](./remix-snowflake.md) (Snowflake connectors/widgets)

---

## Widget Catalog

> Source: [List of widgets](https://www.notion.so/2721d464528f8075a7abfdd7f94a2f68)

Content pipeline visibility — current widget templates. Snowflake widgets also listed in [remix-snowflake.md](./remix-snowflake.md).

### ZenDesk Widgets (iOS/Android, OAuth, AI: Anthropic)

| Widget              | Size         | Output                                          |
|---------------------|--------------|-------------------------------------------------|
| Ticket count & list | small/medium | AI-generated query → ticket list + count        |
| Users count         | small        | AI-generated query → user count                 |
| Agent leaderboard   | medium       | AI-generated query → agents assigned to tickets |

**Configurator fields (all ZenDesk):** subdomain, human prompt, widget title
**Project:** `remix-india.remixlabs.com/e/edit/zendesk_template_widgets`

---

## Base Catalog (Published Components)

> Source: [Base catalog](https://www.notion.so/1051d464528f80c3952fd7cd198da342)
> Parent: Remix Documentation > Remix Catalogs and Toolkits
> Library: `remix-libraries` workspace on `agt.remixlabs.com` (see [federated-servers.md](./federated-servers.md) for access setup)

Documented components in the base catalog:

### Table Component

> Source: [Table Component](https://www.notion.so/f66595ea7c7146d895513e296da96275)

- **Description:** Displays a table on a page. Can be editable or display-only. Cell clicks emit column number, row number, and cell value.
- **Dependencies:** None
- **Release:** R1.0 — 11 September 2024

### Charts

> Source: [Charts](https://www.notion.so/d1b57c77c9014194b59b6f5a23b7f188)

- **Description:** WebComponent-based data visualization using the [Chart.js](https://www.chartjs.org/docs/latest/) library. A web-component allows leveraging 3rd-party libraries and creates a
  reusable custom HTML element.
- **Dependencies:** None
- **Release:** 1.0 — 24 September 2024

### rmx-voice-chat-headless

> Source: [rmx-voice-chat-headless](https://www.notion.so/26c1d464528f80ed87f0cec9d0c31635)

- Page exists but has no content yet.

---

## HubSpot Widget Research

> Source: [iOS/Android widgets for HubSpot - Research](https://www.notion.so/25e1d464528f80d4af58e6ef53aa4ddb)

### Key Findings

1. **No comparable approach in the HubSpot ecosystem** — Remix's native iOS/Android widgets + Chrome Extension widgets is unique; no similar solution found in marketplace or internet research
2. **HubSpot's own mobile widgets:** only 2 (Recently Called Contacts, Upcoming Calls) — minimal investment
3. **HubSpot Marketplace penetration is low:**
    - Only 50 apps have 10k+ installs (3% of 300k customers)
    - Only 107 apps have 3k+ installs (1% penetration)
    - Top apps are Gmail (392k), Google Calendar (245k), Outlook (226k), Zapier (172k), Slack (79k)
    - Databox (analytics, most comparable) = 17k installs (#33)
4. **Analytics landscape:** customers extract data to Tableau/Power BI/Looker; HubSpot mobile app has dashboards + AI report generation (Pro plan only)

### Strategic Implication

Remix approach is differentiated (no direct competitor in HubSpot ecosystem), but marketplace penetration data suggests getting to 1% = top 100 apps.

---

## remix_labs Library Guidance

> Source: [Discussion on remix_labs library](https://www.notion.so/2791d464528f8032b76cfb5e3be51176)

### Rules

1. **All assets must be in `remix_labs` library** — service agents, local agents, screens, components. No dependency on any other library for Remix published assets.
2. Assets from other libraries: replace with `remix_labs` equivalent, or unlink from source and publish to `remix_labs`
3. Use `draft` tag for assets undergoing review
4. Goal: a person can start purely from library assets and get things working

### ZenDesk Widget Setup (from-scratch pattern)

**Service agents** (deploy to workspace):

1. `set_secret` — saves `anthropic_api_key`
2. `get_secret` — retrieves `anthropic_api_key`
3. `generate_zendesk_query` — AI generates ZenDesk API parameters from human prompt

**OAuth setup:**

- Auth server: `remix-india` (current)
- ZenDesk uses subdomain-per-org → create OAuth config per org in `oauth_manager`
- Set `auth_config.config_name` in Constants; if auth server changes, update `auth_config.server`
- Pull in `zendesk_connect` + `confirmation` screens from remix_labs library

**Constants:**

1. Add `auth_config` and `central_workspace` objects from remix_labs (tags: `constants`, `data_model`)
2. ZenDesk color palette: `colors` constant (tag: `zendesk`)
3. Connect all 3 constant objects to Constants screen and save

**Widget template screen:** `zendesk_template_widget`

---

## Connectors (MCP Tools)

> Source: [Connectors](https://www.notion.so/1e61d464528f80508ef4c3df63b25a5f)
> Sub-pages are setup guides — re-fetch for step-by-step details.

### Common Pattern

- All connectors require: Local Mixer + `remix_toolbox` setup + Claude/MCP client
- OAuth callback URI pattern: `https://{ENV}.remixlabs.com/a/x/auth/{AUTH_CONFIG_NAME}/callback`
    - ENV: `remix-dev`, `remix-beta`, `remix`, `remix-india`
    - Production: `https://auth.remixlabs.com/a/x/auth/{AUTH_CONFIG_NAME}/callback`
- OAuth configs managed via OAuth Manager App (per environment)
- MCP tool walkthrough recording: Zoom (see Connectors hub page)

### Connector Inventory

| Connector     | Auth Method                                           | Auth Config Name | Project URL                                                | Owner        | Capabilities                                                                                                                  |
|---------------|-------------------------------------------------------|------------------|------------------------------------------------------------|--------------|-------------------------------------------------------------------------------------------------------------------------------|
| Notion        | Internal integration token                            | —                | `remix-india.remixlabs.com/e/edit/notion_tools`            | Arka         | Search, get/create/update pages, add/get comments (6 tools)                                                                   |
| Gmail         | GCP OAuth (`gmail.compose`)                           | `gmail_connect`  | `remix-dev.remixlabs.com/e/edit/gmail_tools`, `gmail_mcp`  | Wilber, Mark | Create draft, send email                                                                                                      |
| Google Sheets | GCP OAuth (`spreadsheets`, `drive`, `userinfo.email`) | `sheets_connect` | `remix-india.remixlabs.com/e/edit/google_sheets_connector` | Padma        | 28 operations: CRUD sheets/tabs/rows/columns, charts, sort, filter, conditional formatting, merge, freeze, protect (all Done) |
| Airtable      | Personal Access Token                                 | —                | `remix-india.remixlabs.com/e/edit/airtable_connecter`      | Riju         | 14 tools: bases, tables, records, fields CRUD + search                                                                        |
| Confluence    | API token + email                                     | —                | `remix-india.remixlabs.com/e/edit/confuence_connector`     | Nivesh       | Get page content by page_id                                                                                                   |
| Slack         | Slack App OAuth (bot + user tokens)                   | `slack_oauth`    | —                                                          | Shawn        | Post message, create/add to canvas, list users/channels/files (6 tools)                                                       |

Snowflake connectors: see [remix-snowflake.md](./remix-snowflake.md)

### Slack OAuth Scopes (Bot Token)

`channels:read`, `channels:history`, `canvases:read`, `canvases:write`, `chat:write`, `chat:write.customize`, `chat:write.public`, `files:read`, `users:read`, `users.profile:read`, `users:read.email`

---

## GTM Strategy & Reusable Patterns

> Source: [Remix Next GTM](https://www.notion.so/2001d464528f803293aaebc066f1acb4)

### Collaborative AI-Based Warm Leads

A GTM motion for prospecting via personal networks:

1. Person builds a campaign target list of ~100 accounts by clipping in LinkedIn → list of internal company IDs
2. A search URL is constructed using those IDs with filters like "looking for CIOs/CSOs"
3. Distribute as **smart tasks** to friends/family/network
4. Recipient opens LinkedIn with the search term showing 1-2 degree connections → clips matching profiles
5. System creates a **network graph** (who-knows-who, degrees of separation)
6. Plan formulated for warm intros → AI ghost-writes intro messages
7. Additional smart tasks created for LinkedIn posts (awareness amplification)

High value for regional events. Alteryx expressed interest.

### Reusable Asset Pattern (from GTM delivery)

- Generic CRUD services → specialized instances via AI (e.g. "Save Entity" → "Save Lead")
- All specializations done by AI using the **live artifacts model**
- Pipeline: generalized components → libraries with specialized components → build → deploy at scale
- Live artifact saves to well-known local location → tools pick up and submit to shared workspace (same format as publish)

---

## T1/T2 Customer Categories

> Source: [T1/T2 Customers](https://www.notion.so/20d1d464528f80608571fc691f0bf69f)
> Customer database: `collection://20d1d464-528f-8054-84c5-000bef3022bd`

Four patterns/categories for customer solutions:

| # | Category                                           | Description                                                                                                                               | Examples          |
|---|----------------------------------------------------|-------------------------------------------------------------------------------------------------------------------------------------------|-------------------|
| 1 | **Informational awareness**                        | Show what's going on in your org. Ad hoc queries, NatLang widgets, federated queries, point-of-use, query diffs over time                 | Info consumption  |
| 2 | **Human-in-the-loop / Distributed data gathering** | Beacons + Chrome ext → data back into Snowflake, note-taking. Info creation. Create and deploy workflows rapidly on different surfaces    | LinkedIn clipping |
| 3 | **Agentic use cases**                              | Generalized MCP: get info from Snowflake, send email, etc. IT focused, potential end-user config. Inline prompting in L2 / template tools | MCP connectors    |
| 4 | **Digital experience at scale**                    | CDP+ETL = DBP. Educational sale (hasn't crossed the chasm). Self-serve. Given a quiz template → auto-create quiz and deploy               | DBP sites         |

Named prospects: OptHealth, Funda, NFC, Alteryx, Guidewire, Snowflake.

---

## AI Agents (Library Assets)

> Source: [AI](https://www.notion.so/2991d464528f8043b82ff22040962cc2)

Library agents in `remix_labs` for integrating with third-party AI providers. All agents are cloud agents deployed to a workspace.

### Prerequisites

1. Add `remix_labs` library to searchable libraries: URL `https://agt.remixlabs.com/ws/remix_labs` (in builder: Find → Config → Add URL)
2. Save AI provider API tokens to the target workspace using:
    - **`set_secret`** agent (deploy to workspace first) — [library asset](https://remix.app/remix/asset?source=https://agt.remixlabs.com/ws/remix_labs/QvzOgJPsKHQgaGQwgiM4BQfCfXMkWwH8)
    - **`save_secret`** helper screen — [library asset](https://remix.app/remix/asset?source=https://agt.remixlabs.com/ws/com_remixlabs_john/Hmn3PDz9LkEVfaLWlTiRt97mcGnwYCUb) (update Node 14 bindings
      to point to your deployed `set_secret` agent before saving)

### Anthropic Agents

| Agent                | Description                                                                                                 | Input                                   |
|----------------------|-------------------------------------------------------------------------------------------------------------|-----------------------------------------|
| `anthropic_messages` | Fully configurable agent                                                                                    | `secret_key`, `body` (API request body) |
| `anthropic`          | Pre-configured agent with artifact for setup (edit secret_key, model, system prompt, tools for JSON output) | —                                       |

### OpenAI Agents

No artifacts — require manually updating API key / Org ID fields within the agent.

| Agent                | Description                                                                                                                      | Input                                             |
|----------------------|----------------------------------------------------------------------------------------------------------------------------------|---------------------------------------------------|
| `openai_completions` | Fully configurable agent                                                                                                         | `api_key_secret_key`, `org_id_secret_key`, `body` |
| `openai_json`        | Returns JSON conforming to a specified output structure                                                                          | —                                                 |
| `openai_text`        | Returns text response                                                                                                            | —                                                 |
| `openai_image`       | Returns URL of a generated + uploaded image for a given prompt                                                                   | —                                                 |
| `openai_embeddings`  | Generates text embeddings for a given string (measures relatedness). Requires a vector database for nearest-match queries (TBD). | —                                                 |

### Gemini Agents

No artifacts — require manually updating API key fields within the agent.

| Agent         | Description                                             |
|---------------|---------------------------------------------------------|
| `gemini_json` | Returns JSON conforming to a specified output structure |
| `gemini_text` | Returns text response                                   |

**Source app:** `https://remix.remixlabs.com/e/edit/ai`
