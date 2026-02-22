# Connectors — Integration Setup Guides

Notion source: https://www.notion.so/1e61d464528f80508ef4c3df63b25a5f

All connectors are MCP-based tools that run via the **remix_toolbox** setup on Remix Desktop (Local Mixer).
Prerequisites for all: Remix Desktop installed and running, Claude (or MCP-capable app) connected to the MCP server.

## Common Patterns

### OAuth callback URI format

```
https://{ENV}.remixlabs.com/a/x/auth/{AUTH_CONFIG_NAME}/callback
```

- Production: `https://auth.remixlabs.com/a/x/auth/{AUTH_CONFIG_NAME}/callback`
- `{ENV}` options: `remix-dev`, `remix-beta`, `remix`, `remix-india`
- Add a separate URI for each environment

### OAuth Manager App (for creating/editing auth configs)

- dev: `https://remix-dev.remixlabs.com/oauth_manager/home`
- beta: `https://remix-beta.remixlabs.com/oauth_manager/home`
- remix-india: `https://remix-india.remixlabs.com/oauth_manager/home`
- Note: auth config name cannot be changed once set; must match `{AUTH_CONFIG_NAME}` in callback URI

### OAuth Tools App (alternative)

- `https://remix.remixlabs.com/oauth_tools/configs`

### Zoom walkthrough recording

Wilber's MCP tool setup walkthrough: https://zoom.us/rec/share/y775YKGeagHP0rYa7jHx8u_jo64lxlRt1DoW8z_QBUpcgT7cOa1HRO9SDds_u0Q4.Mnbd00W4oeqoDgre?startTime=1746453591000
Passcode: nY5a^.GD

---

## Notion

Notion page: https://www.notion.so/1ea1d464528f8046bd7feb94aa1137fc
Remix project: https://remix-india.remixlabs.com/e/edit/notion_tools
Auth type: **Internal integration token** (not OAuth)

### Capabilities

- Search pages/databases
- Get page contents
- Create new pages
- Update existing pages
- Add and retrieve comments

### Setup

#### 1. Create a Notion integration

1. Navigate to https://www.notion.so/profile/integrations
2. Click "New integration" or select an existing one
3. Select your associated workspace; set type to **Internal**
4. Click Save → "Configure Integration Settings"
5. Copy the **Integration Secret**
6. Default capabilities (Read, Update, Insert content) are sufficient; adjust as needed

#### 2. Connect pages to the integration

1. Open the Notion page you want the integration to access
2. Click the ··· menu (top-right) → "Connections" → select your integration
3. This grants access to that page and all its child pages

#### 3. Connect tools in Remix

1. Install the Notion tools from the Remix catalog (personal catalog)
2. Restart Claude (or your MCP app)
3. Navigate to: https://remix-india.remixlabs.com/notion_tools/manage_token
4. Enter your Integration Secret and click "Save Token"

---

## Gmail

Notion page: https://www.notion.so/1e41d464528f80bab43bc79f02116d05
Remix projects: https://remix-dev.remixlabs.com/e/edit/gmail_tools | https://remix-dev.remixlabs.com/e/edit/gmail_mcp
Auth type: **OAuth 2.0** (GCP project)
Auth config name: `gmail_connect`

### Capabilities

- Create email drafts
- Send emails

### Required OAuth scope

- `https://www.googleapis.com/auth/gmail.compose`

### Setup

#### 1. Create a GCP project

1. Go to https://console.cloud.google.com
2. Project dropdown → "New Project" → name it (e.g. "Gmail Connect")

#### 2. Enable Gmail API

APIs & Services → Enabled APIs & services → Enable APIs → search "Gmail API" → Enable

#### 3. Configure OAuth Consent Screen

APIs & Services → OAuth consent screen → fill in app info (app name: "Gmail Connect")

#### 4. Add scopes

OAuth consent screen → Data Access → "Add or remove scopes" → add `gmail.compose` → Update

#### 5. Create OAuth credentials

1. APIs & Services → Credentials → Create Credentials → OAuth client ID
2. Application type: **Web application**
3. Name: "Gmail Connect"
4. Authorized redirect URIs:
    - Production: `https://auth.remixlabs.com/a/x/auth/gmail_connect/callback`
    - Dev: `https://remix-dev.remixlabs.com/a/x/auth/gmail_connect/callback`
5. Click Create → copy/download Client ID and Client Secret

#### 6. Create auth config in Remix

1. Open OAuth Manager App for your environment
2. Tap "New +" → fill out the form with Client ID, Client Secret, and the `gmail_connect` name
3. Auth config name must match `gmail_connect` exactly (cannot be changed later)

---

## Google Sheet

Notion page: https://www.notion.so/1eb1d464528f809f90e3f5d0ec35450b
Remix project: https://remix-india.remixlabs.com/e/edit/google_sheets_connector
Connector screen: https://remix-india.remixlabs.com/e/edit/google_sheets_connector_oauth/sheets_connect
Auth type: **OAuth 2.0** (GCP project)
Auth config name: `sheets_connect`

### Capabilities

- Get/create/rename spreadsheets and sheet tabs
- Append, update, clear rows and cells
- Read full spreadsheet or individual sheet content
- Insert/delete rows and columns
- Create charts, apply conditional formatting, freeze rows/columns
- Merge/unmerge cells, set row/column heights and widths
- Sort, filter, apply functions
- Copy sheet tabs, protect cell ranges
- Retrieve and clear all notes

### Required OAuth scopes

- `https://www.googleapis.com/auth/userinfo.email`
- `https://www.googleapis.com/auth/spreadsheets`
- `https://www.googleapis.com/auth/drive`

### Setup

#### 1. Create a GCP project

Go to https://console.cloud.google.com → project dropdown → New Project → name (e.g. "Google Sheet Connector")

#### 2. Enable Google Sheets API

APIs & Services → Enabled APIs → search "Google Sheets API" → Enable

#### 3. Configure OAuth Consent Screen

APIs & Services → OAuth consent screen → fill in app info

#### 4. Create OAuth credentials

1. APIs & Services → OAuth Consent Screen → "+ Create credentials" → OAuth client ID
2. Name: "Google Sheets Connector OAuth Client"
3. Authorized redirect URIs:
    - Production: `https://auth.remixlabs.com/a/x/auth/sheets_connect/callback`
    - Dev variants: `https://{ENV}.remixlabs.com/a/x/auth/sheets_connect/callback`
        - `{ENV}`: `remix-beta`, `remix-dev`, `remix`, `remix-india`
4. Click Create → copy Client ID and Client Secret

#### 5. Add scopes

Click the created OAuth client → Data Access → "Add or remove scopes" → add all 3 scopes above → Update

#### 6. Create auth config in Remix

1. Open OAuth Manager App for your environment
2. Tap "New +" → fill form; name must be `sheets_connect`

#### 7. Configure Connector Screen

1. Navigate to the Connector Screen: https://remix-india.remixlabs.com/e/edit/google_sheets_connector_oauth/sheets_connect
2. Go to [app constants](https://remix-india.remixlabs.com/e/edit/google_sheets_connector_oauth/Constants) and fill in:
    - `gmail_connect.auth_server`: your auth server environment
    - `gmail_connect.auth_config_name`: `gmail_connect`
    - `redirect_url`: URL where users land after auth
3. Optional OAuth button states: `signin`, `signout`, `invalid_token`

#### 8. Test

Click the sign-in button → OAuth flow completes → verify token via the `get_token` agent

---

## Airtable

Notion page: https://www.notion.so/1eb1d464528f80d6a1dacd85a0708175
Remix project: https://remix-india.remixlabs.com/e/edit/airtable_connecter
Auth type: **Personal Access Token (PAT)** — no OAuth

### Capabilities (14 agents)

| Agent                      | Description                                                                |
|----------------------------|----------------------------------------------------------------------------|
| `list_bases`               | Lists all bases in your account                                            |
| `list_tables_in_base`      | Lists all tables in a base (`base_id` required)                            |
| `get_table_details_by_id`  | Gets table details (`base_id`, `table_id`)                                 |
| `create_single_new_record` | Creates one record (`base_id`, `table_id`, `record`)                       |
| `list_records_in_table`    | Lists all records in a table (`base_id`, `table_id`)                       |
| `get_record_by_id`         | Gets a single record (`base_id`, `table_id`, `record_id`)                  |
| `update_record_by_id`      | Updates a record (`base_id`, `table_id`, `record_id`, `record`)            |
| `delete_record_by_id`      | Deletes a record (`base_id`, `table_id`, `record_id`)                      |
| `create_new_table`         | Creates a new table (`base_id`, table schema object)                       |
| `update_table_by_id`       | Updates table metadata (`base_id`, `table_id`, updated schema)             |
| `create_field`             | Adds a field/column (`base_id`, `table_id`, field schema)                  |
| `update_field`             | Updates a field (`base_id`, `table_id`, `field_id`, field schema)          |
| `create_multiple_records`  | Creates up to 10 records (`base_id`, `table_id`, array of records)         |
| `search_record_in_table`   | Searches records by field value (`base_id`, `table_id`, value, field name) |

### Setup

#### 1. Create a Personal Access Token in Airtable

1. Open your Airtable account → click profile (top-right) → Builder Hub
2. Go to "Personal access tokens" → Create new token
3. Add required permissions (scopes) for your use case
4. Copy the generated PAT securely

#### 2. Configure PAT in Remix

1. Navigate to the Airtable connector constants page in the Remix project
2. Enter the PAT in the constants field
3. Make (publish) the project

---

## Confluence

Notion page: https://www.notion.so/1eb1d464528f80cd828bd5aea15d505d
Remix project: https://remix-india.remixlabs.com/e/edit/confuence_connector
Connector page: https://remix-india.remixlabs.com/e/edit/confuence_connector/get_confuence_api_key
Auth type: **Atlassian API Token + email** — no OAuth

### Capabilities

- Fetch Confluence page content by page ID (`get_confluence_page` agent)

### Setup

#### 1. Create an Atlassian API Token

1. Navigate to Atlassian Account API Tokens (https://id.atlassian.com/manage-profile/security/api-tokens)
2. Click "Create API token" → give it a name (e.g. "Confluence Connector")
3. Copy the generated API Token securely

#### 2. Configure in Remix

1. Navigate to the connector page: https://remix-india.remixlabs.com/e/edit/confuence_connector/get_confuence_api_key
2. Enter the API token in the field **`confluence_api_key`**
3. Enter your Atlassian account email in **`user_email`**

#### 3. Grant page/space access

1. Go to the Confluence space or page you want to access (e.g. `https://your-site.atlassian.net/wiki/spaces/TS`)
2. For a **space**: Space Settings → Permissions → add your Atlassian user with View/Edit
3. For a **page**: click the lock icon (top-right) → set restrictions to allow your user
4. Extract the `page_id` from the URL (e.g. in `.../pages/131095/AI+Trends...`, `page_id` = `131095`)

#### 4. Fetch page content

Pass `page_id` to the `get_confluence_page` cloud agent.

---

## Slack

Notion page: https://www.notion.so/1ee1d464528f80b2bdefe231f4280888
Auth type: **OAuth 2.0** (Slack App)
Auth config name: `slack_oauth`
Requirement: Slack administrative permissions for your org's Slack instance

### Capabilities

- Post messages to channels
- Create and update canvases
- List users and channels
- List files/canvases in channels

### Required bot token scopes

- `channels:read`, `channels:history`
- `canvases:read`, `canvases:write`
- `chat:write`, `chat:write.customize`, `chat:write.public`
- `files:read`
- `users:read`, `users.profile:read`, `users:read.email`

Apply the same scopes for **User Token Scopes** as well.

### OAuth config fields

| Field         | Value                                   |
|---------------|-----------------------------------------|
| Auth URL      | `https://slack.com/oauth/v2/authorize`  |
| Token URL     | `https://slack.com/api/oauth.v2.access` |
| User Info URL | `https://slack.com/api/auth.test`       |
| Token Extras  | `authed_user`                           |

### Setup

#### 1. Create a Slack App

1. Go to https://api.slack.com/apps → "Create New App" → "From Scratch"
2. Choose a name and workspace → Create

#### 2. Configure OAuth & redirect URIs

1. Left nav → "OAuth & Permissions" → scroll to "Redirect URLs"
2. Add redirect URIs:
    - Production: `https://auth.remixlabs.com/a/x/auth/slack_oauth/callback`
    - Dev: `https://{ENV}.remixlabs.com/a/x/auth/slack_oauth/callback`
        - `{ENV}`: `remix-dev`, `remix-beta`, `remix`
3. Click "Save URLs"

#### 3. Add bot token scopes

Scroll to "Scopes" → "Bot Token Scopes" → "Add an OAuth Scope" → add all scopes listed above.
Repeat for "User Token Scopes".

#### 4. Create auth config in Remix

1. Open OAuth Tools App: https://remix.remixlabs.com/oauth_tools/configs
2. Click "+" → create new configuration
3. From Slack "Basic Information" screen, copy Client ID and Client Secret
4. Fill in all fields:
    - **Name**: include the word "slack"; must match `slack_oauth` (same as `AUTH_CONFIG_NAME`)
    - **Auth URL / Token URL / User Info URL / Scopes / Token Extras**: as listed in the table above
5. Click Save