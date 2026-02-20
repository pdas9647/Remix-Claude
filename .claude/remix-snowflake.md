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
