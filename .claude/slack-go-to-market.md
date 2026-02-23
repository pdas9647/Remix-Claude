# #go-to-market Slack Channel — Remix Labs

**Coverage:** Dec 20, 2025 – Feb 23, 2026 (+ earlier GTM context; channel is very low-traffic)
**Channel ID:** C7HGRVDL3
**Purpose:** GTM updates/status; target application & process scenarios, incl. for demo workflows

---

## Recent Activity (Dec 2025 – Feb 2026)

### Feb 5, 2026 — Atomicwork CRM meeting (Mukund)

Detailed meeting summary from call with Atomicwork (Vijay Royapathi, Ameet D/AD):

- **Company profile**: 10 AEs (5 US + 5 India), HubSpot CRM, Gong, Clay, Revenue Hero, Smartlead, 6sense
- **GTM motion**: Account-based selling, MEDDIC qualification, deal stages: Discovery Scheduled → Discovery Complete → Demo Complete → POC → Negotiation → Win/Loss
- **Primary objectives**:
    1. Real-time sales enablement — "action board" for pre-customer pipeline, proactive mobile/desktop widgets, automated nudges on stale deals
    2. Workflow automation — stale deal alerts, pipeline movement notifications, custom AE widgets
    3. Data challenges — HubSpot analytics insufficient, data enrichment quality poor vs Apollo/6sense, no data warehouse yet
- **Key pain points**: Sales activity scattered across 8–9 tools with bidirectional syncs; lifecycle event timestamps not fully captured in HubSpot; dependency on GTM engineer (AD) for all config
- **Success criteria**: Enable AD to rapidly build widgets; multi-channel delivery (mobile widgets, Slack, Chrome extension Copilot); role-specific views
- **Next steps**: Working session with AD + Remix (Feb 5); pilot with simple widget family on HubSpot data; future data lake architecture (BigQuery/Snowflake)
- **Assessment**: Strong fit for widget/workflow automation platform; AD is key champion

### Dec 15, 2025 — Snowflake pitch day video (Mukund)

Snowflake pitch day presentation recording: https://vimeo.com/1146629723/d7b21d29cb

### Dec 4, 2025 — 5 Star Music stats (Suma)

Since go-live: 47 IVR phone calls + 12 text messages delivered.

---

## GTM Strategy & Positioning

### Oct 31, 2025 — Strategy & GTM deck (Mukund)

Shared strategy deck: https://docs.google.com/presentation/d/1MX8EM8yddi5d1QzGDbtOrQa5jreAYt0Mhjry--7E088/edit

- **Thread discussion** (18 replies): Data lake positioning debated
    - **Gerd**: Cautioned against claiming data lake capabilities prematurely — "we shouldn't claim 'we can do that already' if 'we are on the track' is more honest"
    - **Vijay**: Data lakes are where the market is headed — AI needs customer-360 context; referenced Databricks/Snowflake market caps — "we will be sitting alongside of those"
    - **Mukund**: HubSpot announced data lake at recent conference; Snowflake projecting same; strategy must target where the market is going
    - Zylo SaaS Management Index Report shared (SaaS sprawl cost-savings opportunity)

### Customer support sales slides (Oct 31, 2025)

Sales deck for support-oriented conversations (w/ Zendesk demos): https://docs.google.com/presentation/d/1PWn7U2WNsALgvL-HdgKGQFQRkr3_9EYHaVQ1m6OxJiY/edit

### Canvas: Snowflake Prep for GTM & Pitch Day (Nov 2025)

Six-area strategy:

1. AI Infrastructure → deploy Remix in Snowpark container
2. Mobile widgets → build in Studio, deploy native, Snowflake auth + sample data
3. Contextual widgets → Snowflake → Remix Copilot (Customer-360)
4. Zendesk on Snowflake → webhook embeddings, Cortex AI sentiment/search, KB creation
5. Digital experience → runtime embed, dynamically generated from Snowflake data
6. Marketplace → private listing now, public later

### Canvas: Remix Differentiators for AI Customer Support (Nov 2025)

**Solution differentiators**: Enterprise co-pilots (cross-team process context + data lake), custom digital experiences (intake, walk-throughs, CSAT), analytics (cluster analysis,
unstructured→structured), process monitoring + mobile alerts, incremental KB maintenance, data-lake-centric architecture

**Platform differentiators**: TCO, privacy/security, control/governance, time-to-value, low disruption, multi-surface/modality, AI-native

### Canvas: AI Customer Support PRD (Nov 2025)

Full PRD extracted from Duckie AI webinar (Nov 2025). Six core capabilities: autonomous resolution, agent co-pilot, knowledge management, VoC intelligence, predictive analytics, workflow automation.
Tiered pricing model (Starter $99/Professional $149/Enterprise custom per agent/month). 12-month phased roadmap. Target: mid-market to enterprise B2B SaaS (1,000+ monthly tickets). Expected 60–70% AI
resolution rate at maturity.

---

## Sales Pipeline & Prospect Conversations

### Simpplr / Zendesk (Oct 2025, Mukund)

Canvas recap of conversation with Jai Valluri (Simpplr):

- 18 support engineers, email-only via Zendesk, no chat/voice
- Pain: self-service gaps, analytics (replacing Sizeense with Thoughtspot), Snowflake integration in progress
- Interested in embedding AI tools for case summaries/next best actions
- Next: Jai to meet Vijay; evaluate beyond Zendesk to Service Cloud

### Opt Health / Zendesk (Oct 2025, Mukund)

Canvas shared: Opt Health conversation with Chris Collins re Zendesk. Tagged John, Reza, Vijay.

### Alteryx (Jun 2025, Mukund)

- Contact: Vikram (IT team). Prefer Snowflake/Snowpark self-hosted deployment behind firewall; Plan B: AWS
- Enterprise OpenAI license — flows must work with OpenAI APIs
- Desktop + Mobile apps included; services for example flows setup
- 3-month free trial, then evaluate longer-term
- **Thread**: Chris asked what they want to run in Snowflake; Vijay suggested offer both Snowflake + AWS, collaborate with early customer on limitations; for Snowflake container: mostly Snowflake +
  Salesforce if possible

### TappedIn (Jul 2025, Mukund)

6-month outlook: 5,000 users, 20 businesses, $0.07/user/month introductory rate, Stripe payments planned.

- Revenue model: likely ad fee + lead gen (coupon redemptions) rather than per-active-user

### Zendesk demo recording (Jul 2025, Mukund)

Demo/meeting with Zendesk: https://vimeo.com/1102333918/4cdfeb8452

---

## MCP / Connector Strategy (May 2025)

### Zapier vs Remix connectors (Arvind)

- Shritesh: "because you're not limited by MCP"
- Vijay: "you can use Zapier alongside us" — proposed having a Zapier connector
- Arvind: "A connector to the connector!"

### LLM client partnership productisation (Arvind)

- Arvind: Not reasonable to expect partners to go through deploy motions to supercharge their LLM client — need to productise the workflow
- Shritesh: "use your existing LLM by just downloading this one app" gets people through the door; meet users wherever they are
- Arvind: "Remix is better with the client, the client is better with Remix" but need push if business users just use the web

---

## User Story Development (Apr–May 2025)

### Claude + Remix narrative stories (Arvind, Apr 2025)

Requested 5–7 bullet-point user stories from GTM team. Mukund provided 3:

1. **Sales pipeline from HubSpot**: ask Claude → connector install → sign in → table → timeline dashboard → deploy to mobile
2. **Quiz app from PDF**: upload → set difficulty → Claude generates Remix quiz → edit mode → deploy + admin view
3. **Digital Business Profile (DBP)**: provide URLs + calendar + PDF → Claude generates DBP → edit → deploy as superapp

Vijay suggested "build a dashboard on top of a parquet file" as starting point. Arvind repeatedly bumped for more concrete stories from Vijay/John/Reza; Shritesh noted priorities were agreed in a
separate call.

### SaaS de-siloing deck (May/Apr 2025, Mukund)

Shared deck: "Remix Overview — Desiloing SaaS in the AI Era"

---

## Other Notable Items

- **Jan 2025 — TappedIn live on App Store** (Shritesh): https://apps.apple.com/us/app/tappedin-events-deals/id6657850557 — exploring App Clips
- **Jan 2025 — Passkeys suggestion** (Gerd): "We should do passkeys as alternative to Google sign-in"
- **Jan 2025 — OSS licensing flag** (Simon): MIT license compliance for third-party code in builder as it gets used externally
- **Jan 2025 — Netradyne demo clip** (Mukund): https://remix.app/about/netradyne-demo — case study, ROI calc, FAQ
- **May 2025 — External docs on Notion** (Shritesh): started at https://curious-turnover-84b.notion.site/Remix-Desktop-1ee1d464528f800cb165e706911a0090 — discussed hosting at docs.remixlabs.com
- **Sep 2025 — HubSpot Marketplace Listing** (John): https://www.notion.so/Hubspot-Marketplace-Listing-c7fb69cb6a8a4d30bd6d54a6cab2ec1b
- **Sep 2025 — HubSpot preso** (Mukund): positioning, customer journey, pricing for HubSpot Inbound25
- **Nov 2025 — Breakthrough Builders** (Mukund): asked Vijay/John for notes on latest discussion with Kamal re invoices management/DBP

---

## Channel Tone & Patterns

- Very low traffic (1–3 messages per month); long gaps between posts
- Mostly Mukund posting: customer meeting summaries, strategy decks, sales materials, demo recordings
- Arvind drives product/design alignment: requests user stories, pushes for concrete deliverables, raises strategic questions
- Technical team (Gerd, Simon, Chris, Shritesh) contribute reality checks and tactical input
- Vijay provides high-level strategic framing (market trends, data lake thesis, positioning)
- Canvas tabs used extensively for detailed meeting notes and strategy documents
- Direct, informal tone; @-mentions to pull in relevant people; threads for substantive discussion