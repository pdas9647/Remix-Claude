# Remix Product — First Release Scope & Personas

> Source: [Scope of first release, mental model, key terms](https://www.notion.so/26f1d464528f80c68d5ff6d663522db5)
> See also: [remix-product.md](./remix-product.md) (task list, desktop design, pricing/SKU, sub-page index)

---

## First Release — Scope, Mental Model, Key Terms

### Alignment Points

- **Foundational release** — primary purpose is to get something installable at a customer site with daily self-serve value for target personas
- No ambition for full self-serve surface area in first release — initial setup done between Remix Customer Success and customer's IT team
- GTM sales kit sells a vision (operational visibility, actionable insights, agentic workflows); some demo scenarios not supported in first release

### Three Personas (First Release)

**IT:**

- Remix = software vendor with a `Remix instance` on a `Remix server` made of 1+ `workspaces`
- One `central workspace` with `auth server` + central `service agents`
- Uses `IT admin tools` for user management, service agent management, workspace management

**Contributors:**

- Work in `Remix Desktop`, connect to central server/workspace to login
- Access `libraries` with `iOS/Android widget templates` and `Chrome Extension templates`
- `Configure` templates → `publish` widgets/flows back to the library

**End Users:**

- Install `Remix mobile app` and/or `Remix Chrome Extension`
- Login with org's Remix server + central workspace
- `Give access` to org systems (HubSpot etc.) via `OAuth`
- `Configure` widgets/flows locally on device/browser

### Key Terms (Customer-Facing)

| Term                    | Definition                                                                                                                     |
|-------------------------|--------------------------------------------------------------------------------------------------------------------------------|
| Remix instance          | The installation of the Remix platform for an organization                                                                     |
| Widget                  | A configured and published instance of a widget template (iOS/Android)                                                         |
| Flow                    | A configured Chrome Extension flow (not called "widget" even if informational)                                                 |
| Widget template         | A library asset with a configurator, openable in Remix Desktop                                                                 |
| Remix clients           | 3 container apps: Remix Desktop, Remix mobile app, Remix Chrome Extension. Run flows/widgets locally on user's edge device.    |
| End user surfaces       | iOS/Android widgets + Chrome Extension flows (mobile app flows in next release)                                                |
| Workspace               | Server-side namespace: databases, service agents, files, secrets. Central workspace runs auth server + central service agents. |
| Library                 | Specialized workspace for storing assets, with built-in sync/management agents                                                 |
| Widget template library | Provided by Remix CS at install, periodically updated                                                                          |
| Widget library          | Set of configured widgets available to the customer                                                                            |
| User's chosen widgets   | Subset the user has selected for their Chrome Extension (iPhone/Android selection not tracked by Remix)                        |
| Remix IT tools          | Web app for: AI service config, service agent permissions, usage monitoring                                                    |
| Remix widget mgmt tool  | Part of Remix Desktop: browse template catalog, browse/edit/delete widgets                                                     |
| Service agent           | Cloud lambda function in a workspace with an ACL — primary auth/authz mechanism                                                |
| Agentic workflow        | Business flow of 2-6 steps, one or more done by an agent                                                                       |
| MCP                     | Model Context Protocol — LLM tool/resource access, built into Remix Desktop, configurable by IT                                |

### MoSCoW Scope (First Release)

**Must Have:**

- Auth: Google, Office365, Apple (LinkedIn dropped)
- Remix Desktop on Mac + Windows
- Remix mobile app on App Store + Play Store
- Chrome Extension on Chrome Store
- 20-30 informational widget templates for HubSpot, Salesforce, ZenDesk, Snowflake
- Widget templates configurable in Desktop, publishable to workspace library
- Widgets configurable within mobile app (facets only) and Chrome Extension
- Widgets installable/manageable on iOS + Android
- OAuth-only auth for HubSpot/Salesforce/ZenDesk/Snowflake (no API keys/service accounts)
- Workspace-level user restriction (named emails or domain)
- Customer can onboard new users
- Description and tags on widgets

**Won't Have (first release):**

- SSO
- Safari/Edge extensions
- Mac/Windows widgets
- Customer-created/edited widget templates
- Customer deleting templates or self-updating from Remix central library
- Multiple workspaces or user-specific workspaces
- Multiple widget template libraries
- Widget publish approval workflow
- Dev/UAT/prod environments (single env = production)
- Sharing specific widget to specific user list

### Self-Serve vs Remix Customer Success

**Remix CS (initial setup):** workspace setup, service agent install + permissions, IT web app setup, client install links, widget template library customization + install, testing, first-user
onboarding

**Customer IT (initial setup):** system-of-record config (e.g. HubSpot app permissions), AI service config (API key, model), service agent permissions

**Sales/Marketing Ops (ongoing):** browse templates, configure + publish widgets, manage widget library, send download instructions, share configured templates, set workspace permissions

**End Users (ongoing):** pick widgets from library in Chrome Extension or mobile, configure widgets in mobile app

---

## Personas

> Source: [Personas](https://www.notion.so/27e1d464528f80ae846cf0e81fcc7f59)

Personas are based on **tasks they do** and **tools they use**. An individual person may perform tasks across personas.

| Persona          | Actual Roles                                                   | Remix Surface                                                                  | What They Do (Q4 2025+)                                                                                                                                                                                                                                                                                                                     |
|------------------|----------------------------------------------------------------|--------------------------------------------------------------------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| **IT**           | Central IT, or dept admin/config personnel                     | Remix web app (IT tools)                                                       | **Outside Remix:** gatekeeper of SoR permissions (HubSpot etc.), mobile device mgmt, Chrome browser policies, SSO admin. **Inside Remix:** workspace setup, service agent install/config/permissions, security compliance, usage monitoring. **Never:** domain expert in end-user workflows, building libraries/assets.                     |
| **Builders**     | IT team or dept personnel willing to learn Studio              | Remix Desktop Studio + Web Studio                                              | Net new asset creation + publication to shared library. Highly skilled in Remix platform. Collaborate with Assemblers via shared libraries. Usually don't have deep domain knowledge — provide reference implementations. Maintain quality through rigorous testing. In almost all implementations = Remix Labs services arm.               |
| **Assemblers**   | —                                                              | Desktop Studio or enhanced live artifact                                       | Business domain experts. Assemble apps from library assets. Configure assets for business needs, create specialized assets. Not expected to be Remix platform experts. Contact Builder for missing assets. Combine library assets into screens + service agents, heavily AI-augmented. May create specializable templates for Contributors. |
| **Contributors** | Marketing ops, sales person (creates survey, SOCL query, etc.) | Desktop live artifacts, Mobile live artifacts, Extension variants & data tiles | Contribute assets back to library for IT/Assemblers/automated processes to include in apps. Employees of the business. Runtimes set up for them by IT with training.                                                                                                                                                                        |
| **End Users**    | —                                                              | Web, App Clips (NOT live artifacts or Studio)                                  | Participate in workflows and move on. Apps and service agents serve their workflows.                                                                                                                                                                                                                                                        |
