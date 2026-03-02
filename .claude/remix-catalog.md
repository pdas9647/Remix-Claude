# Remix Catalog — Widgets, Libraries, Connectors, GTM

> See also: [federated-servers.md](./federated-servers.md), [remix-snowflake.md](./remix-snowflake.md)

---

## Widget Catalog

> [Notion](https://www.notion.so/2721d464528f8075a7abfdd7f94a2f68)

### Snowflake Widgets (iOS/Android, API key, AI: Anthropic)

All share configurator: database, schema, table, human prompt, widget title. Project: `remix-india.remixlabs.com/e/edit/snowflake/`

| Widget | Output |
|--------|--------|
| List | AI-generated title + list of rows with selected columns |
| Aggregate | AI-generated title + aggregated values (prices, salaries, etc.) |
| Details | AI-generated title + single row details |
| Chart | AI-generated title + bar chart (e.g. top 3 products by price) |

### ZenDesk Widgets (iOS/Android, OAuth, AI: Anthropic)

Configurator: subdomain, human prompt, widget title. Project: `remix-india.remixlabs.com/e/edit/zendesk_template_widgets`

| Widget              | Size         | Output                                |
|---------------------|--------------|---------------------------------------|
| Ticket count & list | small/medium | AI query → ticket list + count        |
| Users count         | small        | AI query → user count                 |
| Agent leaderboard   | medium       | AI query → agents assigned to tickets |

---

## Base Catalog

> [Notion](https://www.notion.so/1051d464528f80c3952fd7cd198da342) | Library: `remix-libraries` on `agt.remixlabs.com`

- **Table Component** ([Notion](https://www.notion.so/f66595ea7c7146d895513e296da96275)): Editable or display-only table. Cell clicks emit column/row/value. R1.0 Sep 2024.
- **Charts** ([Notion](https://www.notion.so/d1b57c77c9014194b59b6f5a23b7f188)): Chart.js WebComponent visualization. R1.0 Sep 2024.
- **rmx-voice-chat-headless** ([Notion](https://www.notion.so/26c1d464528f80ed87f0cec9d0c31635)): No content yet.

---

## HubSpot Widget Research

> [Notion](https://www.notion.so/25e1d464528f80d4af58e6ef53aa4ddb)

**Key findings:** No comparable native iOS/Android widget approach in HubSpot ecosystem — Remix is unique. HubSpot's own mobile widgets: only 2 (Recently Called, Upcoming Calls). Marketplace: only 50 apps have 10k+ installs (3% of 300k customers); top = Gmail 392k, Google Cal 245k; Databox (most comparable) = 17k (#33). Getting to 1% penetration = top 100 apps.

---

## remix_labs Library Rules

> [Notion](https://www.notion.so/2791d464528f8032b76cfb5e3be51176)

1. All assets must be in `remix_labs` — no dependency on other libraries
2. Replace or unlink external assets and publish to `remix_labs`
3. Use `draft` tag for assets under review
4. Goal: start purely from library assets and get things working

### ZenDesk Setup Pattern

**Service agents** (deploy to workspace): `set_secret` (save `anthropic_api_key`), `get_secret` (retrieve), `generate_zendesk_query` (AI→ZenDesk API params).

**OAuth:** Auth server `remix-india`; ZenDesk = subdomain-per-org → OAuth config per org in `oauth_manager`. Set `auth_config.config_name` in Constants. Pull `zendesk_connect` + `confirmation` screens from library.

**Constants:** Add `auth_config` + `central_workspace` (tags: `constants`, `data_model`) + ZenDesk `colors` palette (tag: `zendesk`).

---

## Connectors (MCP Tools)

> [Notion](https://www.notion.so/1e61d464528f80508ef4c3df63b25a5f)

**Common pattern:** Local Mixer + `remix_toolbox` + Claude/MCP client. OAuth callback: `https://{ENV}.remixlabs.com/a/x/auth/{NAME}/callback` (prod: `auth.remixlabs.com`). Configs via OAuth Manager App.

| Connector     | Auth                           | Project                                   | Capabilities                                                        |
|---------------|--------------------------------|-------------------------------------------|---------------------------------------------------------------------|
| Notion        | Internal integration token     | `remix-india/.../notion_tools`            | Search, get/create/update pages, comments (6 tools)                 |
| Gmail         | GCP OAuth (`gmail.compose`)    | `remix-dev/.../gmail_tools`               | Create draft, send email                                            |
| Google Sheets | GCP OAuth (spreadsheets+drive) | `remix-india/.../google_sheets_connector` | 28 ops: CRUD sheets/tabs/rows/columns, charts, sort, filter, format |
| Airtable      | Personal Access Token          | `remix-india/.../airtable_connecter`      | 14 tools: bases, tables, records, fields CRUD + search              |
| Confluence    | API token + email              | `remix-india/.../confuence_connector`     | Get page content by page_id                                         |
| Slack         | Slack App OAuth (bot+user)     | —                                         | Post message, canvas, list users/channels/files (6 tools)           |

Snowflake connectors: see [remix-snowflake.md](./remix-snowflake.md)

---

## GTM Strategy

> [Notion](https://www.notion.so/2001d464528f803293aaebc066f1acb4)

**Collaborative AI warm leads:** Person builds ~100 account target list via LinkedIn clipping → construct search URLs → distribute as smart tasks to network → recipients find 1-2 degree connections → system builds network graph → AI ghost-writes intro messages → additional tasks for LinkedIn posts. High value for regional events; Alteryx interested.

**Reusable asset pattern:** Generic CRUD services → AI-specialized instances (e.g. "Save Entity" → "Save Lead"). Pipeline: generalized components → libraries → build → deploy at scale. Live artifact saves to well-known location → tools submit to shared workspace.

---

## T1/T2 Customer Categories

> [Notion](https://www.notion.so/20d1d464528f80608571fc691f0bf69f)

| # | Category                        | Description                                                                           |
|---|---------------------------------|---------------------------------------------------------------------------------------|
| 1 | **Informational awareness**     | Show org data: ad hoc queries, NatLang widgets, federated queries, diffs over time    |
| 2 | **Human-in-the-loop**           | Beacons + Chrome ext → data back to Snowflake, note-taking, rapid workflow deployment |
| 3 | **Agentic**                     | Generalized MCP: Snowflake info, email, etc. IT-focused, inline prompting in L2       |
| 4 | **Digital experience at scale** | CDP+ETL = DBP. Educational sale. Self-serve template → auto-create + deploy           |

Prospects: OptHealth, Funda, NFC, Alteryx, Guidewire, Snowflake.

---

## AI Agents (Library Assets)

> [Notion](https://www.notion.so/2991d464528f8043b82ff22040962cc2) | Source app: `remix.remixlabs.com/e/edit/ai`

**Prerequisites:** Add `remix_labs` library URL (`https://agt.remixlabs.com/ws/remix_labs`) to searchable libraries. Save API tokens via `set_secret` agent ([asset](https://remix.app/remix/asset?source=https://agt.remixlabs.com/ws/remix_labs/QvzOgJPsKHQgaGQwgiM4BQfCfXMkWwH8)) or `save_secret` helper screen.

| Agent                               | Provider  | Description                                    |
|-------------------------------------|-----------|------------------------------------------------|
| `anthropic_messages`                | Anthropic | Fully configurable (secret_key + body)         |
| `anthropic`                         | Anthropic | Pre-configured with artifact for setup         |
| `openai_completions`                | OpenAI    | Fully configurable (api_key + org_id + body)   |
| `openai_json/text/image/embeddings` | OpenAI    | Specialized: JSON, text, image gen, embeddings |
| `gemini_json/text`                  | Gemini    | JSON or text response                          |

OpenAI/Gemini agents have no artifacts — require manually updating API key fields.
