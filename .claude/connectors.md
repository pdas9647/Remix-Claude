# Connectors — Integration Setup Guides

> Source: [Connectors](https://www.notion.so/1e61d464528f80508ef4c3df63b25a5f) | Child
> pages: [Notion](https://www.notion.so/1ea1d464528f8046bd7feb94aa1137fc) · [Gmail](https://www.notion.so/1e41d464528f80bab43bc79f02116d05) · [Google Sheet](https://www.notion.so/1eb1d464528f809f90e3f5d0ec35450b) · [Airtable](https://www.notion.so/1eb1d464528f80d6a1dacd85a0708175) · [Confluence](https://www.notion.so/1eb1d464528f80cd828bd5aea15d505d) · [Slack](https://www.notion.so/1ee1d464528f80b2bdefe231f4280888)

All connectors are MCP-based tools via **remix_toolbox** on Remix Desktop (Local Mixer). Prerequisites: Remix Desktop installed, Claude/MCP app connected.

**Assignments:** Wilber+Mark = Gmail, Shawn = Slack, Arka = Notion, Nivesh = Confluence, Riju = Airtable, Padma = Google Sheet.

**Zoom walkthrough:** https://zoom.us/rec/share/y775YKGeagHP0rYa7jHx8u_jo64lxlRt1DoW8z_QBUpcgT7cOa1HRO9SDds_u0Q4.Mnbd00W4oeqoDgre?startTime=1746453591000 (Passcode: nY5a^.GD)
**Sample MCP project:** https://remix-dev.remixlabs.com/e/edit/wc_tools/mcp_tools

## Common OAuth Patterns

**Callback URI:** `https://{ENV}.remixlabs.com/a/x/auth/{AUTH_CONFIG_NAME}/callback` — Production: `https://auth.remixlabs.com/a/x/auth/{AUTH_CONFIG_NAME}/callback`. Envs: `remix-dev`, `remix-beta`,
`remix`, `remix-india`. Add separate URI per env.

**OAuth Manager App:** `https://{ENV}.remixlabs.com/oauth_manager/home` | **OAuth Tools App:** `https://remix.remixlabs.com/oauth_tools/configs`. Auth config name is immutable once set — must match
callback URI.

---

## Notion

Project: https://remix-india.remixlabs.com/e/edit/notion_tools | Auth: **Internal integration token** (not OAuth)

**Capabilities:** Search pages/databases, get page contents, create/update pages, add/retrieve comments.

**Setup:** (1) Create integration at notion.so/profile/integrations → type=Internal → copy secret. (2) On each target page: ··· menu → Connections → select integration (grants access to page +
children). (3) Install Notion tools from Remix catalog → restart Claude → navigate to `notion_tools/manage_token` → paste secret.

---

## Gmail

Projects: [gmail_tools](https://remix-dev.remixlabs.com/e/edit/gmail_tools) | [gmail_mcp](https://remix-dev.remixlabs.com/e/edit/gmail_mcp) | Auth: **OAuth 2.0** (GCP) | Config name: `gmail_connect`

**Capabilities:** Create drafts, send emails. **Scope:** `googleapis.com/auth/gmail.compose`

**Setup:** (1) GCP console → New Project. (2) Enable Gmail API. (3) Configure OAuth consent screen. (4) Add `gmail.compose` scope. (5) Create OAuth client ID (Web app) → add redirect URIs:
`https://auth.remixlabs.com/a/x/auth/gmail_connect/callback` + dev variants → copy Client ID + Secret. (6) OAuth Manager App → New → fill form with `gmail_connect` name + credentials.

---

## Google Sheet

Project: [google_sheets_connector](https://remix-india.remixlabs.com/e/edit/google_sheets_connector) |
Connector: [sheets_connect screen](https://remix-india.remixlabs.com/e/edit/google_sheets_connector_oauth/sheets_connect) | Auth: **OAuth 2.0** (GCP) | Config name: `sheets_connect`

**Capabilities:** Full spreadsheet CRUD — get/create/rename spreadsheets and tabs, append/update/clear rows+cells, insert/delete rows+columns, charts, conditional formatting, freeze/unfreeze,
merge/unmerge, sort, filter, functions, copy tabs, protect ranges, notes.

**Scopes:** `googleapis.com/auth/userinfo.email` + `googleapis.com/auth/spreadsheets` + `googleapis.com/auth/drive`

**Setup:** (1) GCP → New Project. (2) Enable Google Sheets API. (3) Configure OAuth consent screen. (4) Create OAuth client ID → redirect URIs with `sheets_connect`. (5) Add all 3 scopes. (6) OAuth
Manager App → name=`sheets_connect`. (7) Configure connector screen constants (`auth_server`, `auth_config_name`, `redirect_url`). (8) Test: click sign-in → verify token via `get_token` agent.

---

## Airtable

Project: https://remix-india.remixlabs.com/e/edit/airtable_connecter | Auth: **Personal Access Token (PAT)** — no OAuth

**Capabilities (14 agents):** `list_bases` | `list_tables_in_base` | `get_table_details_by_id` | `create_single_new_record` | `list_records_in_table` | `get_record_by_id` | `update_record_by_id` |
`delete_record_by_id` | `create_new_table` | `update_table_by_id` | `create_field` | `update_field` | `create_multiple_records` (up to 10) | `search_record_in_table`

**Setup:** (1) Airtable → profile → Builder Hub → Personal access tokens → create + copy. (2) In Remix project constants, paste PAT → Make (publish).

---

## Confluence

Project: https://remix-india.remixlabs.com/e/edit/confuence_connector | Auth: **Atlassian API Token + email** — no OAuth

**Capabilities:** Fetch page content by ID (`get_confluence_page` agent).

**Setup:** (1) Atlassian → manage-profile/security/api-tokens → create + copy. (2) In connector page (`confuence_connector/get_confuence_api_key`): enter API token as `confluence_api_key`, email as
`user_email`. (3) Grant page/space access in Confluence (Settings → Permissions). (4) Extract `page_id` from URL → pass to agent.

---

## Slack

Auth: **OAuth 2.0** (Slack App) | Config name: `slack_oauth` | Requires Slack admin permissions

**Capabilities:** Post messages, create/update canvases, list users/channels/files.

**Bot token scopes:** `channels:read`, `channels:history`, `canvases:read`, `canvases:write`, `chat:write`, `chat:write.customize`, `chat:write.public`, `files:read`, `users:read`,
`users.profile:read`, `users:read.email`. Apply same for User Token Scopes.

**OAuth config:** Auth URL=`https://slack.com/oauth/v2/authorize` | Token URL=`https://slack.com/api/oauth.v2.access` | User Info URL=`https://slack.com/api/auth.test` | Token Extras=`authed_user`

**Setup:** (1) api.slack.com/apps → Create New App → From Scratch. (2) OAuth & Permissions → add redirect URIs with `slack_oauth`. (3) Add all bot + user token scopes. (4) OAuth Tools App → New
config → paste Client ID + Secret from Basic Information → name must match `slack_oauth` → fill Auth/Token/UserInfo URLs + scopes + Token Extras → Save.