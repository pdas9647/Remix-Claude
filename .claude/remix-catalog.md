# Remix Catalog — Widgets, Libraries, Connectors, GTM

> See also: [federated-servers.md](./federated-servers.md), [remix-snowflake.md](./remix-snowflake.md)

---

## Widget Catalog

> [Notion](https://www.notion.so/2721d464528f8075a7abfdd7f94a2f68)

### Snowflake Widgets (iOS/Android, API key, AI: Anthropic)

All share configurator: database, schema, table, human prompt, widget title. Project: `remix-india.remixlabs.com/e/edit/snowflake/`

| Widget    | Output                                                          |
|-----------|-----------------------------------------------------------------|
| List      | AI-generated title + list of rows with selected columns         |
| Aggregate | AI-generated title + aggregated values (prices, salaries, etc.) |
| Details   | AI-generated title + single row details                         |
| Chart     | AI-generated title + bar chart (e.g. top 3 products by price)   |

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

**Key findings:** No comparable native iOS/Android widget approach in HubSpot ecosystem — Remix is unique. HubSpot's own mobile widgets: only 2 (Recently Called, Upcoming Calls). Marketplace: only 50
apps have 10k+ installs (3% of 300k customers); top = Gmail 392k, Google Cal 245k; Databox (most comparable) = 17k (#33). Getting to 1% penetration = top 100 apps.

---

## remix_labs Library Rules

> [Notion](https://www.notion.so/2791d464528f8032b76cfb5e3be51176)

1. All assets must be in `remix_labs` — no dependency on other libraries
2. Replace or unlink external assets and publish to `remix_labs`
3. Use `draft` tag for assets under review
4. Goal: start purely from library assets and get things working

### ZenDesk Setup Pattern

**Service agents** (deploy to workspace): `set_secret` (save `anthropic_api_key`), `get_secret` (retrieve), `generate_zendesk_query` (AI→ZenDesk API params).

**OAuth:** Auth server `remix-india`; ZenDesk = subdomain-per-org → OAuth config per org in `oauth_manager`. Set `auth_config.config_name` in Constants. Pull `zendesk_connect` + `confirmation` screens
from library.

**Constants:** Add `auth_config` + `central_workspace` (tags: `constants`, `data_model`) + ZenDesk `colors` palette (tag: `zendesk`).

---

## Connectors (MCP Tools)

> [Notion](https://www.notion.so/1e61d464528f80508ef4c3df63b25a5f)

**Common pattern:** Local Mixer + `remix_toolbox` + Claude/MCP client. OAuth callback: `https://{ENV}.remixlabs.com/a/x/auth/{NAME}/callback` (prod: `auth.remixlabs.com`). Configs via OAuth Manager
App.

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

> [Notion — GTM Home](https://www.notion.so/98cc07f3fac14e73a8b3513576160cd9) | [GTM Strategy](https://www.notion.so/2001d464528f803293aaebc066f1acb4)

### May 19 Remix DT Release

> Website content detail: [Notion](https://www.notion.so/1ed1d464528f806abd03e3b73af5c876)

Target: **IT/Builder + Biz Ops persona** at mid-to-large enterprise. Potential customers: Alteryx, Guidewire, Zendesk, UKG, Freshworks, OptHealth.

**Work streams:**

- Website content: builder project, text/graphics for all pages, 2 blog pieces (SaaS sprawl problem; AI imperative + Remix role), contact form (→ HubSpot + Slack)
- Cohort 1 campaign: compiled invitee list in HubSpot, demo slides, demo artifacts (analytics/dashboard, connector, Remix DT), demo scripts, email outreach
- Cohort 2 campaign: connector-specific invitee lists (Airtable, Big Query, Notion, Confluence, Quickbooks, HubSpot)
- Self-serve product trial: use cases by connector

Invitee spreadsheet: [Google Sheets](https://docs.google.com/spreadsheets/d/1822XGm_nJx_5rht5VbL6PQmdoQdvVzYmizH-tAsiUI4/edit)

**Core messaging (SaaS sprawl → AI → Remix DXP):**

1. **Problem — SaaS sprawl**: 1000+ apps in large enterprises, 250+ in mid-size; fragmented data silos; vendor-specific AI compounds fragmentation; shadow IT worsens it
2. **Catalyst — AI tipping point**: Unified context+data needed · Build>Buy compelling again · Rapid AI adoption · CX personalization at scale now expected
3. **Solution — Remix DXP**: De-siloed data · Any LLM, works at the edge · Content library of agentic apps · Deployable everywhere (mobile, browser, web, private cloud) · Enterprise-grade security

**Website home page sections:** (1) SaaS Sprawl Iceberg graphic → (2) AI Catalysts Bridge graphic → (3) Remix DXP solution graphic → (4) CTA: Request a demo

**Example use cases for website:**
- *Actionable analytics*: Query Account 360 (HubSpot + Zendesk + Confluence + Jira) in natural language → draft personalized emails to high-CSAT accounts
- *Marketing campaign*: AI queries warm-lead list (100 CIOs) → distributes LinkedIn enrichment tasks to team → team sends personalized invites via Gmail/Outlook + SDR follow-up

**Blog themes:** "SaaS Silos & AI: The Hidden Costs You Can't Afford" · "Beyond Build vs. Buy: Why 'Build' is Back in the AI Era"

### Demo Scenarios (2025 GTM Working Docs)

> [Notion](https://www.notion.so/1f81d464528f80f9a60bca23f49e28f7)

Four demo scenario categories:

1. **Actionable Analytics**
    - Large dataset (parquet/CSV, e.g. NYC taxi) analyzed with Remix + AI → dashboard
    - Cross-silo enterprise data (Customer-360: Billing, CSAT, Support, Usage) analyzed w/ AI
    - Cross-silo data + connector + workflow (e.g. Customer-360 → Gmail connector → personalized emails)
    - Analysis artifacts sent to systems-of-communication (e.g. chart → Slack)
    - Personal use data (e.g. Strava + HealthKit → VO2max / half-marathon prediction)

2. **Digital Business Profile**: Use template → auto-configure 'splash'/home screen with AI → preview → save → publish as App Clip URL

3. **Quiz from content**: Topic or PDF → quiz AppClip with shareable URL (e.g. Blackjack rules)

4. **Chrome Extension**
    - Focused crawling: market research → collect articles → send to AI → Notion doc
    - AI captures browsing activity/calendar → lists actions in extension (e.g. LinkedIn crawl → add to doc)
    - Add from Gmail: query Gmail → automated task (e.g. respond to customer support ticket)

### Demo Script — Lead-Gen at Events (Booth-Hub Flow)

> [Notion](https://www.notion.so/1c31d464528f80faa37dc7472f9ce9b8)

Step-by-step demo flow for trade-show/event lead generation:

1. **Lead capture**: Booth-hub app → camera (iOS) scans badge or looks up lead list (pre-loaded from CRM/attendee list); add new lead manually if needed
2. **Lead qualification**: Customizable survey (company size, vertical, role) → event staff determines which journey fits the visitor
3. **QR code + DBP Clip**: Instantly generate QR code → visitor scans → receives personalized Digital Business Profile (value prop, marketing content, CTAs: video/case study/meeting scheduler)
4. **Push notifications** (iOS only, ~8h window): Send targeted notifications to visitor's device while still at event — e.g. VIP dinner invite, additional content, follow-up meeting request
5. **Meeting scheduling**: Send calendar link (Google or other) integrated into the DBP Clip
6. **Continued engagement**: Respond to requests for case studies, ROI calculator (custom-built), order forms, NDAs for signature — all via the Clip
7. **Analytics + CRM**: Each DBP screen has a beacon → view which screens opened, how many times, per lead. Export all leads to CRM (direct integration or CSV)

### DXP Launch Use Cases (2025)

> [Notion](https://www.notion.so/1dd1d464528f806c8323c8fe2f846d4e) — Marketing Plan for Remix DXP launch (old)

Target: **Dept/LOB Lead, Ops Lead** (non-technical, process KPI owners) + **IT/Builder** persona.

Three flagship use cases with step-by-step flows:

**1. HubSpot Stage 4 Pipeline ("Show me my Stage 4 sales pipeline from HubSpot using Remix")**
- Claude checks for HubSpot connector → prompts install if missing → user signs in
- Connector displays deals in table/markdown → user asks for timeline view
- Remix opens timeline dashboard → prompts to share/export/deploy as mobile app flow or widget

**2. Quiz App from PDF ("Give me a quiz app for the attached content using Remix")**
- Remix DT: upload PDF → set question count + difficulty (beginner/intermediate/expert)
- Claude generates quiz app → displayed in Remix DT with QR code to test
- User enters edit mode (questions/answers/images) → deploy as mobile flow or widget
- Bonus: admin view for responses/leaderboard

**3. Digital Business Profile ("Give me a Digital Business Profile using Remix")**
- Remix DT collects: website URL, LinkedIn profile URL, YouTube video, Google Calendar sign-in, case study PDF
- Claude generates DBP app → displayed in Remix DT with QR code
- User enters edit mode → deploy as mobile flow or widget
- Bonus: Superapp framework + push notifications to send additional screens

**Content marketing assets planned:** Blog (weekly), email templates, landing page with step-by-step wizard + runtime embed + download links for Remix DT + Claude DT, pricing, Terms of Use, subscription agreement.

**Campaign structure:** Cohort v1/v2/v3 contact lists → email outreach → LinkedIn targeted campaign → website with contact-us flows.

### Sales Kickoff Portal (2025 GTM Concept)

> [Notion](https://www.notion.so/15e1d464528f80ac828ad03a7518b982)

Remix transforms the SKO (Sales Kickoff) experience with a suite of digital experiences:
1. Built from compiled content assets (agenda, case studies, battle-cards, playbooks, ROI calculators, videos, quizzes)
2. Deployed as app-experiences that persist beyond the SKO event
3. Used daily for sales productivity

**App experience for sales reps:** Sign in → access content assets (agenda, bookmarks, session surveys, marketing assets, lead-gen tools). Reps can browse/request assets and access lead-gen + prospect
engagement tools.

**Content asset types:** SKO event agenda w/ venue/speakers/surveys, case studies (by vertical/keyword/product), competitor battle-cards (pros/cons/objections), product brochures, sales playbooks by
persona, DBP for sales reps, ROI calculator, marketing calendar, qualification criteria, videos, SME directory, training assessments/quizzes.

### Landing Page — Field Sales / Lead-Gen Positioning (2025)

> [Notion](https://www.notion.so/1361d464528f80689883f1f06ef9fb79)

**Tagline:** "Transform Lead Capture and Engagement with Rich, Frictionless Digital Experiences"

Key claims: 3× marketing qualified leads · 2× lead conversion · 10× productivity for booth staff, inside sales, account execs.

**Digital Experience Stream flow:**
1. Customized DBP of company value proposition → prospect activates at trade-show / social / text
2. Prospect engages: requests info, sets meeting, shares micro-app with stakeholders
3. Sales rep responds via pre-configured micro-apps → mobile notification to prospect

**Content templates:** Video links, case studies, contact form, personalized contact info, CRM integration (Salesforce/HubSpot), calendar/meeting scheduler, SSO, additional as needed.

**Pricing (2025):** 2-week free trial · $20/user/month · Free for external users · No setup fees (except custom dev).

**Testimonial:** "With Remix our lead capture from trade-shows has improved significantly" — Ethan Alexander, Netradyne.

### GTM Sub-Pages

- [Connector Recipes](https://www.notion.so/1f31d464528f80d98814de11ffafd6a5), [Demo scenarios](https://www.notion.so/1f81d464528f80f9a60bca23f49e28f7)
- [Snowflake x Remix GTM Plan (Feb 2026)](https://www.notion.so/2fb1d464528f80dd9bddeba7672de70b)
- [Content for Website + Landing Pages (May 19)](https://www.notion.so/1ed1d464528f806abd03e3b73af5c876)
- [Alteryx AI Workflow List](https://www.notion.so/2021d464528f80c8b164d3fe71991960), [Remix User Personas](https://www.notion.so/1f41d464528f8068895ad839e01e65ab)
- [Customer Support Domain Research](https://www.notion.so/2961d464528f8070b287e1117d5a4cda), [Zendesk Customer Survey](https://www.notion.so/2481d464528f8027bb04f0bd83bf662c)
- [Project Management (Tasks & Projects)](https://www.notion.so/2941d464528f807894c3c966a0a9c623) — Notion DBs: Projects, Tasks, Timeline

### GTM Tactics

**Collaborative AI warm leads:** Person builds ~100 account target list via LinkedIn clipping → construct search URLs → distribute as smart tasks to network → recipients find 1-2 degree connections →
system builds network graph → AI ghost-writes intro messages → additional tasks for LinkedIn posts. High value for regional events; Alteryx interested.

**Reusable asset pattern:** Generic CRUD services → AI-specialized instances (e.g. "Save Entity" → "Save Lead"). Pipeline: generalized components → libraries → build → deploy at scale. Live artifact
saves to well-known location → tools submit to shared workspace.

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

**Prerequisites:** Add `remix_labs` library URL (`https://agt.remixlabs.com/ws/remix_labs`) to searchable libraries. Save API tokens via `set_secret`
agent ([asset](https://remix.app/remix/asset?source=https://agt.remixlabs.com/ws/remix_labs/QvzOgJPsKHQgaGQwgiM4BQfCfXMkWwH8)) or `save_secret` helper screen.

| Agent                               | Provider  | Description                                    |
|-------------------------------------|-----------|------------------------------------------------|
| `anthropic_messages`                | Anthropic | Fully configurable (secret_key + body)         |
| `anthropic`                         | Anthropic | Pre-configured with artifact for setup         |
| `openai_completions`                | OpenAI    | Fully configurable (api_key + org_id + body)   |
| `openai_json/text/image/embeddings` | OpenAI    | Specialized: JSON, text, image gen, embeddings |
| `gemini_json/text`                  | Gemini    | JSON or text response                          |

OpenAI/Gemini agents have no artifacts — require manually updating API key fields.

### Zendesk Customer Research (Aug 2025)

> [Notion](https://www.notion.so/2481d464528f8027bb04f0bd83bf662c) — Surveying customers of Zendesk re use cases

Two prospect conversations conducted Aug 2025:

**Simpplr (Jai Valluri, VP Customer Support — ~500+ people, Aug 1 2025)**
- 18 support engineers, email-only via Zendesk; no chat/voice; exploring chat as future channel
- Support is existing-customer-only (pre-sales handled by separate team)
- Plans: enhance KB + vector search, replace Sixsense with Thoughtspot for analytics dashboards, integrate Snowflake (full ticketing data, still in progress), evaluating Service Cloud beyond Zendesk
- Pain: improving self-service, efficiency, response times (current SLAs: 1h premium / 4h standard for P1)
- AI interest: embed AI within agent interface for case summaries, next-best actions, relevant KB articles
- In-product AI: plan to embed support features directly in-product
- Next: Vijay + Mukund to meet Jai; evaluate beyond Zendesk to Service Cloud

**Opt Health (Chris Collins, Customer Success — ~25+ people, Aug 7 2025)**
- Meeting notes brief; context in field trial report → [customer-projects-other.md](./customer-projects-other.md#opt-health)

### Customer Support Domain Research (Oct 2025)

> [Notion](https://www.notion.so/2961d464528f8070b287e1117d5a4cda) — Customer Support Domain - Research/Interviews

**Meeting: Kris Bhandare, VP Customer Support, DataStax-IBM — Oct 24, 2025**
(Participants: Vijay + Mukund / Remix)

**DataStax context:** SaaS database (Cassandra-based, MongoDB competitor). ~150 SaaS product cases/month (20% of total). Post-IBM acquisition — large enterprise procurement complexity. Kris previously at Nutanix (60-70k cases/year, built custom automation).

**Key pain points:**
- 80–90% of cases are repetitive (latency, error messages) — human reads graphs and describes what's happening; AI can now do this
- No efficient duplicate/similar-case surfacing → support engineers re-answer identical questions
- Manual KB article creation is time-intensive; training content becomes stale
- "Proving a negative" problem for case deflection metrics

**AI opportunities Kris highlighted:**
- Real-time Slack-based support with direct API access + role-based control (uses Thena.ai)
- Automatic duplicate case detection + surface past solutions to agents
- Auto-generate KB articles from recurring cases
- Podcast-style training from docs (using Notebook LLM); quiz generation for assessment
- Bite-sized TikTok-style training for just-in-time learning (DataStax eliminated training dept using AI)
- Simulation systems running parallel to human agents for quality assurance

**Kris's target:** 50% automation of specific case types → 10% monthly labor reduction

**KPIs monitored:** NPS, CSAT, case volume, first response time, time-to-resolution. Dashboard: Salesforce + Tableau.

**Enterprise buyer requirements (critical):**
- Fast time-to-value: POC within **one week**, on real tickets in real environment (not simulations)
- Minimal operational disruption; easy integration with existing CRM/ticketing/DB
- IBM security review = 10+ people — complex procurement post-acquisition
- Kris would have bought Remix at DataStax pre-IBM or at Nutanix; post-IBM = harder

**Remix action items from meeting:**
1. Rapid POC capability (within 1 week) on real tickets
2. Lead with NPS/CSAT improvement, not just cost reduction
3. Build rich educational content (videos, interactive checklists, not just static docs)
4. Target mid-market first (less procurement friction than large enterprise)
5. Pursue intros to Kris's SVP boss and **Sean Ryan** (former Nutanix, "very good at these things")

**Key quotes:**
> "80, 90% of the cases… the human does is look at graphs, look at charts and says, here is what's going on. That part of the interpretation can be done through AI now."
> "Give me a value, time to value within three [weeks]… six months, a million dollars — no one's going to give you that."
> "My customer is getting better responses, and faster responses… NPS, CSAT… anytime the NPS and CSAT is higher is when you respond very quickly."

**Competitive tools mentioned:** Thena.ai (Slack/support integration), Notebook LLM (podcast gen), Tableau/Salesforce (dashboards).
