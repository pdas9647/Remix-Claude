# HubSpot Toolkit

> Source: [HubSpot toolkit](https://www.notion.so/1371d464528f809b9f51cf5b4337a137)
> Parent: Remix Documentation > Remix Catalogs and Toolkits
> See also: [remix-catalog.md](./remix-catalog.md) (HubSpot widget research, connector inventory)

---

## Overview

The Remix HubSpot toolkit is a set of assets, templates, tools and tutorials for building HubSpot-integrated Remix applications. Examples:

- Custom mobile app for sales reps
- IVR / WhatsApp customer interaction channels that update HubSpot
- Embedding HubSpot data as widgets in internal or customer-facing systems

---

## Toolkit Components

> Source: [HubSpot toolkit assets](https://www.notion.so/1371d464528f80c6b609c3589d263c8b)

Four deliverables:

| # | Component                    | Description                                                                                                                                                                      |
|---|------------------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| 1 | **HubSpot Connector app**    | Deployed as a Remix app in a workspace. Server-side set of agents that connect to a HubSpot instance. Remix applications connect to this Connector to fetch/update HubSpot data. |
| 2 | **HubSpot Configurator app** | Application to configure the HubSpot Connector.                                                                                                                                  |
| 3 | **HubSpot library**          | Library assets to accelerate development. Hosted at `https://agt.remixlabs.com/ws/remix-libraries/_rmx_hubspot`. Not versioned — updated on an ongoing basis.                    |
| 4 | **Example application**      | Simple app showing how all toolkit assets work together.                                                                                                                         |

---

## Asset Versions & Downloads

All .remix files downloadable from `agt.files.remix.app/remix-libraries/files/_rmx_hubspot/`. Deploy via [Cloud Workspace Tool](https://remix.remixlabs.com/cloud_workspace/workspaces).

### HubSpot Connector

| Version  | Notes             | Download URL                                                        |
|----------|-------------------|---------------------------------------------------------------------|
| 20241204 | Latest release    | `agt.files.remix.app/.../20241204/hubspot_connector-20241204.remix` |
| 20241120 | Support for OAuth | `agt.files.remix.app/.../20241120/hubspot_connector-20241120.remix` |
| 20241111 | First release     | `agt.files.remix.app/.../20241111/hubspot_connector-20241111.remix` |

### HubSpot Connector Configurator

- **Hosted version:** `https://remix.remixlabs.com/hubspot_connector_config/configure`
- .remix download: `20241112/hubspot_connector_config-20241112.remix` (first release)

### HubSpot App Configurator

- **Hosted version:** `https://remix-beta.remixlabs.com/hubspot_app_config/hubspot_admin`
- .remix download: `20241204/hubspot_app_config-20241204.remix` (first release)

### HubSpot Example App

| Version  | Notes                                    |
|----------|------------------------------------------|
| 20241204 | Separate app config screens into own app |
| 20241120 | Support for OAuth                        |
| 20241111 | First release                            |

### HubSpot Widgets Example

| Version  | Notes           |
|----------|-----------------|
| 20241204 | Initial release |

---

## Installation

> Source: [Installation instructions](https://www.notion.so/13b1d464528f80d29831d063eaf7b365)
> Re-fetch for detailed step-by-step screenshots.

For each HubSpot instance you want to connect:

1. **Step 1:** [Create a connected app in HubSpot](https://www.notion.so/1551d464528f80929400eee5f15a0365) — register an app in HubSpot Developer portal
2. **Step 2:** [Deploy the Remix HubSpot Connector app](https://www.notion.so/1551d464528f80df886bc2bb89afbf12) — deploy .remix file to workspace
3. **Step 3:** [Configure the Remix HubSpot Connector app](https://www.notion.so/1551d464528f80aabe01e82333c75953) — configure via Configurator

Then use the example apps as a starting point for building your own app.

---

## OAuth Setup

> Source: [Set up OAuth in HubSpot](https://www.notion.so/1441d464528f8005a611c39fc2f76cc3)
> Re-fetch for full 18-step screenshot walkthrough.

### Key Details

- **Requires:** HubSpot Developer Account (choose "A custom integration" during setup)
- **Redirect URL:** `auth.remixlabs.com/a/x/auth/hubspot_oauth/callback`
- **Auth config name:** `hubspot_oauth`

### Required OAuth Scopes

```
crm.objects.companies.read
crm.objects.companies.write
crm.objects.contacts.read
crm.objects.contacts.write
crm.objects.deals.read
crm.objects.deals.write
crm.objects.owners.read
crm.objects.users.read
oauth
```

### Optional Scopes

Add any other scopes as **optional** based on your use case (e.g. tickets, leads, conversations).

---

## Use Case Scenarios

> Source: [Example scenarios](https://www.notion.so/1371d464528f80eaa4a9e94ffb518ad5)
> Page is a draft/placeholder — content not yet written.

Planned scenario categories:

- HubSpot tiered pricing forcing app usage
- Edge engagement points → HubSpot updates (IVR, TripAdvisor reviews, WhatsApp)
- Internal system integration (e.g. HubSpot ticketing ↔ Jira)
- Workflows
- AI agent integration
- Interactive surveys with AI

Named use cases in sidebar:

- Updating a ticket based on edge engagement
- Creating a ticket from a submitted review
- WhatsApp integration
- Creating a ticket
- Slack integration

---

## HubSpot Library

> Source: [HubSpot library](https://www.notion.so/1371d464528f80599defe5ccbc7d1e9c)
> Page is a placeholder — no content yet. Installation and documentation sections TBD.

Library URL: `https://agt.remixlabs.com/ws/remix-libraries/_rmx_hubspot`
For access setup, see [Accessing Remix libraries](https://www.notion.so/13c1d464528f8039beb0c08aa8413722) (stored in [federated-servers.md](./federated-servers.md)).

---

## Creating a HubSpot Widget (Desktop Guide)

> Source: [🧩 How to Create a HubSpot Widget](https://www.notion.so/30d1d464528f80aea1b5fda08278ad59) — standalone page; last updated 2026-02-21

4-phase workflow for building, configuring, and deploying a HubSpot widget via Remix Desktop + HubSpot Configurator.

**Prerequisites:**

- [Remix Desktop](https://remix.app/remix/desktop-releases) installed
- [HubSpot Configurator](https://remix.app/remix/hubspot_configurator/install) installed

### Phase 0 — Create Project

1. Create a new project in Remix Desktop
2. Copy widget asset [`hubspot_widget_oauth_template`](https://remix.app/remix/asset?source=https://agt.remixlabs.com/ws/remix_labs/yKWEG8wayWlGvh5N8ifRZRDd0yqmDOI2) and paste into the project
3. Rename the template widget (always set name before creating the widget record)

### Phase 1 — Sync Object Schemas

1. Open **HubSpot Configurator** → **Schema Sync** section
2. Sign in, select HubSpot objects (e.g. Contacts, Deals, Companies)
3. Tap **Sync** and wait for confirmation

> Re-sync whenever HubSpot object structure changes — stale schemas cause incorrect AI query generation in Phase 2.

### Phase 2 — Configure the Component (AI-assisted)

1. In **HubSpot Configurator** → click **"Configure Component"**
2. Select the component type and the HubSpot object (must match a synced schema from Phase 1)
3. Enter a natural-language prompt describing the query:
    - *"Show the 10 most recently updated open deals"*
    - *"Total number of active leads"*
    - *"Total number of deals by stage"* (bar chart)
4. Review generated query; refine prompt if needed
5. Click **Copy Component** — keep clipboard ready for Phase 3

### Phase 3 — Build the Widget

1. Open the widget from Phase 0; locate the designated component slot (see annotation node)
2. Paste the copied component
3. Connect 3 required bindings:

| Binding    | Purpose                                    |
|------------|--------------------------------------------|
| **Token**  | HubSpot authentication token               |
| **Invoke** | Triggers data query on load/refresh        |
| **Card**   | Maps returned fields to visual card layout |

4. Test in builder: tap **on load** in the in-params node; verify data, layout, filters

**Common issues:**

- No data → check token binding and HubSpot connection
- Wrong data → revisit Phase 2 prompt
- Broken card layout → verify card binding field mapping

### Phase 4 — Package & Deploy

1. Inside the widget, use **Create Widget Record** tool (see annotation node); fill in metadata
2. Package via **Workspace Tools** plugin → navigate to workspace → **Export Widgets**
    - Package must include the widget records from step 1
3. Install the packaged `.remix` file to the target device via the plugin-provided URL
4. On device home screen: **Add Widget** → search "remix" → select size → add widget → long-press to edit and select the widget
