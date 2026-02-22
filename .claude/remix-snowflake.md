# Remix + Snowflake

> Parent Notion page: [Remix product](https://www.notion.so/27e1d464528f802291b6d5a093fbc10d)

---

## Remix in Snowflake/Snowpark

> Source: [Remix in Snowflake/Snowpark](https://www.notion.so/2841d464528f804ba6fbe2d8feb7a963)
> GitHub: https://github.com/remixlabs/remix-issues/issues/143

Spike on a "self-host" Remix deployable entirely within **Snowpark Container Service**, so Snowflake-using customers have a deployment contained entirely within Snowflake. Separate from the main
desktop product.

### Snowflake Workspaces

| Name                    | Workspace ID | Snowflake Account | Account URL                                |
|-------------------------|--------------|-------------------|--------------------------------------------|
| Snowflake Remix Demo    | `6Xj36DfSbF` | `REMIX_DEMO`      | `app.snowflake.com/oxfsvki/remix_demo`     |
| Snowflake Internal Demo | `vuVXGsZuxW` | `SNOWFLAKE_DEMO`  | `app.snowflake.com/oxfsvki/snowflake_demo` |

Both have dedicated Snowflake auth flows. Sign-in requires a user in the corresponding Snowflake account. Remix identity = email on Snowflake user profile; sign-in uses Snowflake credentials (
username + password/passkey).

---

## Remix Snowflake Reference Implementation (Spike)

> Source: [Remix Snowflake reference implementation](https://www.notion.so/2931d464528f80cc9da8ede348733125) — WIP spike,
> sub-page: [Alteryx scenarios](https://www.notion.so/24a1d464528f808ca6e3d9ae5022b64d)

A spike for a self-hosted Remix deployment entirely within **Snowpark Container Services**, so Snowflake customers have a deployment contained within Snowflake. Separate from the main desktop product.

### Snowflake Product Areas in Scope

- **Unistore** — Unify transactional + analytical workloads in Snowflake
- **Cortex AI** — Primary focus for Remix+Snowflake demos:
    - Cortex Search Service (lexical + semantic/vector search)
    - Cortex Analyst (automatic semantic model generation)
    - Cortex Agents
    - Document processing pipeline (chunking + vector embeddings built-in)
    - Snowflake Intelligence (conversational AI over structured + unstructured data; queries Salesforce/ZenDesk via Snowflake connectivity)
- **Snowpark** — Code execution environments
- **Marketplace** — Third-party data sources

> Page is largely a skeleton (most sections empty). Re-fetch for any updates.

---

## Snowflake Widgets

> See also: [remix-product.md](./remix-product.md) Widget Catalog section for full widget list

Snowflake widgets (iOS/Android, API key auth, AI: Anthropic):

| Widget              | Output                                                          | Project URL                                            |
|---------------------|-----------------------------------------------------------------|--------------------------------------------------------|
| Snowflake List      | AI-generated title + list of rows with AI-selected columns      | `remix-india.remixlabs.com/e/edit/snowflake/list`      |
| Snowflake Aggregate | AI-generated title + aggregated values (prices, salaries, etc.) | `remix-india.remixlabs.com/e/edit/snowflake/aggregate` |
| Snowflake Details   | AI-generated title + row detail view                            | `remix-india.remixlabs.com/e/edit/snowflake/detail`    |
| Snowflake Chart     | AI-generated title + bar chart (e.g. top 3 by price)            | `remix-india.remixlabs.com/e/edit/snowflake/chart`     |

**Configurator fields:** database, schema, table, human prompt, widget title

---

## Snowflake Connectors

> See also: [remix-product.md](./remix-product.md) Connectors section for full connector inventory

| Connector                | Auth Method                        | Auth Config Name    | Capabilities                                        |
|--------------------------|------------------------------------|---------------------|-----------------------------------------------------|
| Snowflake (OAuth)        | Snowflake custom OAuth integration | `snowflake_connect` | Run queries as signed-in user (PUBLIC role default) |
| Snowflake (Service Acct) | Key-pair JWT auth                  | —                   | Programmatic API access, key-pair auth              |

---

## Snowflake — Cortex Search (Catalog Guide)

> Source: [Snowflake — Cortex Search](https://www.notion.so/2aa1d464528f8016ab74d92bc328935b)
> Parent: Remix Documentation > Remix Catalogs and Toolkits

Step-by-step guide for storing data in Snowflake and enabling **Cortex Search** (semantic + keyword hybrid search).

### Prerequisites

1. Add `remix_labs` library (Find → Config → Add URL). Agents are in the **Snowflake** collection.
2. Create or use an existing Snowflake table with a unique column for upserts.
3. Grant privileges to **PUBLIC** role on database/schema/table.

### Cortex Search Service

Create an AI-powered search index on your table using the Cortex Service creation worksheet template (re-fetch Notion page for full SQL). Key parameters:

| Parameter                                      | Description                                                             |
|------------------------------------------------|-------------------------------------------------------------------------|
| `service_name`                                 | Name of the Cortex Search service (referenced in `cortex_search` agent) |
| `database_name` / `schema_name` / `table_name` | Target table                                                            |
| `embedding_columns`                            | Columns used as embeddings in searches (comma-separated)                |
| `returned_columns`                             | Columns returned in search results (comma-separated)                    |
| `target_lag`                                   | How up-to-date the index stays (e.g. `1 hour`, `1 minute`)              |
| `embedding_model`                              | Default: `snowflake-arctic-embed-l-v2.0`                                |

### Agents

| Agent           | Description                                                                          | Library Asset                                                                                                          |
|-----------------|--------------------------------------------------------------------------------------|------------------------------------------------------------------------------------------------------------------------|
| `upsert_record` | Takes raw data, inserts or updates a row keyed by unique ID                          | [asset](https://remix.app/remix/asset?source=https://agt.remixlabs.com/ws/remix_labs/pTJneCrLzA7lhz6v75LoXzcaMwSPfcC9) |
| `cortex_search` | Takes query string + data projection + limit, returns top results by relevance score | [asset](https://remix.app/remix/asset?source=https://agt.remixlabs.com/ws/remix_labs/eUyIoYFTHkceAvdsBcVgWVhUHnUlGJ45) |

### Typical Workflow

1. **Create table** in Snowflake (ensure unique column for upserts, grant PUBLIC role privileges)
2. **Create Cortex Search Service** using worksheet template → creates AI-powered search index
3. **Upsert data** using `upsert_record` agent
4. **Execute searches** using `cortex_search` agent → semantic + keyword hybrid search over Snowflake tables

---

### Snowflake OAuth SQL

```sql
CREATE OR REPLACE SECURITY INTEGRATION snowflake_connect
  TYPE = OAUTH
  OAUTH_CLIENT = CUSTOM
  OAUTH_CLIENT_TYPE = 'CONFIDENTIAL'
  OAUTH_REDIRECT_URI = 'https://<SUBDOMAIN>.remixlabs.com/a/x/auth/<AUTH_CONFIG_NAME>/callback'
  ENABLED = TRUE
  OAUTH_ISSUE_REFRESH_TOKENS = TRUE;
```

---

## Snowflake OAuth Integration Setup

> Source: https://www.notion.so/20e1d464528f8057b143d055514d6cb5

Runs queries as a signed-in user with the default `PUBLIC` role.
Auth config name: `snowflake_connect`

**Prerequisites:** Snowflake org account; `ACCOUNTADMIN` role or `CREATE INTEGRATION` privilege.

### 1. Create a worksheet in Snowflake

Navigate to your Snowflake account at `https://app.snowflake.com/<ORG_ID>/<ACC_ID>` → Projects → Worksheets → "+" → name it `snowflake_connect setup`.

### 2. Create the security integration

```sql
CREATE OR REPLACE SECURITY INTEGRATION snowflake_connect
  TYPE = OAUTH
  OAUTH_CLIENT = CUSTOM
  OAUTH_CLIENT_TYPE = 'CONFIDENTIAL'
  OAUTH_REDIRECT_URI = 'https://<SUBDOMAIN>.remixlabs.com/a/x/auth/<AUTH_CONFIG_NAME>/callback'
  ENABLED = TRUE
  OAUTH_ISSUE_REFRESH_TOKENS = TRUE;
```

Replace `<SUBDOMAIN>` with `remix` and `<AUTH_CONFIG_NAME>` with `snowflake_connect` for standard setup.
Ref: https://docs.snowflake.com/en/user-guide/oauth-custom

### 3. Retrieve client credentials

```sql
SELECT SYSTEM$SHOW_OAUTH_CLIENT_SECRETS('<SECURITY_INTEGRATION_NAME>');
```

> **`<SECURITY_INTEGRATION_NAME>` must be ALL CAPS** (e.g. `SNOWFLAKE_CONNECT`).

Returns the Client ID and Client Secret needed for the Remix auth config.

### 4. Get authorization and token endpoints

```sql
DESC INTEGRATION <SECURITY_INTEGRATION_NAME>;
```

> **No quotes** around the integration name here.

Outputs `OAUTH_AUTHORIZATION_ENDPOINT` and `OAUTH_TOKEN_ENDPOINT`.

### 5. Scopes

OAuth scopes map directly to Snowflake roles. Default scope for this guide:

```
session:role:PUBLIC
```

Replace `PUBLIC` with any other role as needed. Ref: https://docs.snowflake.com/en/user-guide/oauth-ext-overview#scopes

### 6. Create auth config in Remix

Use the OAuth Manager App for your environment (see Common Patterns in connectors.md):
1. Tap "New +" → fill in Client ID, Client Secret, auth/token endpoints, and scopes
2. Name must be `snowflake_connect` (cannot be changed after creation)

---

## Snowflake Service Account Setup

> Source: https://www.notion.so/2a81d464528f80de8febcf0b590be206

Key-pair (RSA) JWT authentication for programmatic API access — no OAuth flow.

**Prerequisites:** `ACCOUNTADMIN` access (or sufficient privileges); OpenSSL installed locally.

### Step 1: Generate RSA key pair

```bash
# Generate private key
openssl genrsa 2048 | openssl pkcs8 -topk8 -inform PEM -out rsa_key.p8 -nocrypt

# Generate public key from private key
openssl rsa -in rsa_key.p8 -pubout -out rsa_key.pub
```

Ref: https://docs.snowflake.com/en/user-guide/key-pair-auth#configuring-key-pair-authentication

### Step 2: Prepare public key (strip headers)

```bash
grep -v "BEGIN PUBLIC" rsa_key.pub | grep -v "END PUBLIC" | pbcopy
```

Copies the bare public key content to clipboard (needed for Step 3).

### Step 3: Create the service account in Snowflake

Run in a Snowflake worksheet:

```sql
-- Create the service account user
CREATE USER <SERVICE_ACCOUNT>
  TYPE = 'SERVICE'
  DEFAULT_ROLE = 'PUBLIC'
  DEFAULT_WAREHOUSE = 'COMPUTE_WH'
  COMMENT = 'General purpose service account with CRUD permissions.';

-- Create open network policy (allows connections from any IP)
CREATE NETWORK POLICY open_network_policy
  ALLOWED_IP_LIST = ('0.0.0.0/0')
  COMMENT = 'Open network policy allowing all IPs for a service account';

-- Assign network policy to service account
ALTER USER <SERVICE_ACCOUNT> SET NETWORK_POLICY = 'OPEN_NETWORK_POLICY';

-- Set the RSA public key (paste the key content from Step 2)
ALTER USER <SERVICE_ACCOUNT> SET RSA_PUBLIC_KEY='<paste_public_key_here>';

-- Get the public key fingerprint
DESC USER <SERVICE_ACCOUNT>
  ->> SELECT SUBSTR(
        (SELECT "value" FROM $1
           WHERE "property" = 'RSA_PUBLIC_KEY_FP'),
        LEN('SHA256:') + 1) AS key;
```

> **Note:** `open_network_policy` allows all IPs (`0.0.0.0/0`). Review before production use.
> Ref: https://docs.snowflake.com/en/sql-reference/commands-security#network-policy

### Step 4: Generating JWT tokens

Four parameters required:

| Parameter | Where to get it |
|---|---|
| `account_id` | Snowflake → account name (bottom-left) → hover "Account" → "View Account Details" → copy Account Identifier |
| `user` | The `<SERVICE_ACCOUNT>` name from Step 3 |
| `public_key_fingerprint` | Output of last query in Step 3 |
| `base64_key` | Run command below |

Get the base64-encoded private key:

```bash
cat rsa_key.p8 | sed 's/^[[:space:]]*//;s/[[:space:]]*$//' | base64 | pbcopy
```

(Trims whitespace, preserves headers, encodes to base64, copies to clipboard.)

Use the **Snowflake Key Pair JWT Token Creator** asset to generate tokens:
https://remix.app/remix/asset?source=https://agt.remixlabs.com/ws/remix_labs/d5cyzPdtcaN6hLNIBJUayTrwoWZOGddB
