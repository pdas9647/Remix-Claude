# Standup Notes — Jan 2026

**Coverage:** Jan 23–30, 2026
**Format:** Condensed bullets per standup. Google Doc link on each date for on-demand full transcript fetch.

---

## 2026-01-30 — [Google Doc](https://docs.google.com/document/d/1ncuCkA6mcMKrdUWCU4lvGdY7dQSlr4jnax-v6qgkLCk)

- **Bomisco extension**: New version reduces blank extract error. Workaround: manual zip install + reload button. LinkedIn probing for extensions causes false errors; customer's Windows + slow EU
  internet compounds issue.
- **GTM prospects**: Op Health (chat for 2.5–3K customers), Starlight (financial services, SMS/RCS chat — RCS dependency).
- **Lumber schema**: 532 tables from AWS SQA. Strategy: dashboard atop **denormalized schema** using materialized views (not duplicating Postgres). Reza targeting one report by Monday.
- **Desktop blockers**: White screen freeze (suspected Tauri). Auth workflow fix (RMX auth app) deploying.

---

## 2026-01-29 — [Google Doc](https://docs.google.com/document/d/1vCcRlwX92enDij8K4S8rD5FV_yk4vg8NyCiWsHhB15o)

- **Scripted chat demo**: Wilbur demoed pre-recorded, non-AI chat flow for public use. Experiences within chat are interactive. Adding intro screen + lead-gen ending.
- **Lumber components**: John modifying agents for SQL generation + alias records. Mark: scatterplot, notebook (more templated), Parquet-in-Snowflake as data source.

---

## 2026-01-28 — [Google Doc](https://docs.google.com/document/d/1lF3PaJaSUmsZuz26vFqXveGj4qLh7n1bhuA_BFqCiA0)

- **Lumber timeline**: Owed to customers; CEO following up with Vijay. Go-live depends on desktop stability + auth. Reza getting up to speed.
- **Desktop OAuth ~50% failure rate**: Missing base URL in redirect (race condition). Didier + Benedikt pair-debugging.
- **Landing page standardization**: Must use latest Tailwind config (Arvind) + only `remix_labs` library.
- **Snowflake agent flow**: Padmanabha built schema+table+prompt → data retrieval (relevant for Lumber).

---

## 2026-01-27 — [Google Doc](https://docs.google.com/document/d/1z3RlPjMu2FbmRBKT8Cibr_R7b9UpQolxS4L9XY2XCAQ)

- **AI strategy**: AI "paints us into a corner" — bespoke platform vs AI building standard React/Node. Pivot: maintainability tooling. Policy: $20/mo individual AI tool expense, no company-wide
  license.
- **Lumber architecture**: Postgres (AWS) → denormalize → Snowflake. Mixer as Docker in Lumber AWS. Error registry must be co-hosted (app with agents, not directory). Standard deployment endpoints
  needed. Train 3–4 people for self-service reports, mid-Feb go-live. Chris + Da own solution architecture.
- **Desktop**: File upload 0 KB bug (XRS, blocking). O workflow broken. Web worker crashes not caught on main thread.
- **Builder**: Groups in catalog, paste scene graph, unified login consistent across Desktop/AMP/Mobile.
- **Web components**: RMX2E chart + grid (Tyler). Monorepo consolidation (Turborepo). Race condition bug (direct binding without `set value` node).
- **Engineering process**: Need consistent work-tracking tool (past Notion/GitHub/own-app attempts failed).

---

## 2026-01-26 — [Google Doc](https://docs.google.com/document/d/129K2WIMWaZ02QDV84J4NYRSZDyHH6uIIUEuCKNGp-7Y)

- **Desktop blockers**: File upload broken (needs Elm change, affects builder + runtime). UI grayed out (Wilbur). Loading ~20s to L2 (missing cache-control headers from mixer).
- **Blob URLs**: Gerd demonstrated bypassing scene graph size limit (>10 MB) for images. Client-side only.
- **Rust CI**: Aggressive optimization flags → 3 min link time; reduced to 2 sec with lighter flags.
- **runtime.json hotfix**: Wrong file open flags when overwriting broke it in production.
- **Mix query**: Save behavior fix (outputs=inputs). Return remix values instead of records. New `key` function.
- **Compiler**: Oliver types ready for review. `but` match pattern started.
- **Desktop identity question**: Builder-only vs compact surface application? Deferred.

---

## 2026-01-23 — [Google Doc](https://docs.google.com/document/d/1WvX1itbVflCivCXSqgxOt982TfojmerGKUbN6PjW0gA)

- **Lumber**: Flat-rate analytics deal (not per-customer). Workforce Junction **ditched** (too complicated). Snowflake tabular reporting. 2–3 weeks to finalize.
- **Funda**: LinkedIn scraping of 600+ profiles **failed** (4 accounts blocked). Decision: members self-maintain. Adequate for volunteer group.
- **Bomisco**: Remnant issues to close next week.
- **Cohort 1**: Fivestar Music + Orderly signed/paid. Orderly at 1 hotel, plan to expand.
- **GTM**: New Store (legal content search, in negotiation). Op Health (chat, more funding). Snowflake marketplace ~2 weeks + SI partner. Zendesk Labs follow-up. Salesforce LWC (iframe→events).
- **App Clip "killer model"**: Push notifs, NFC tag launch (Airbnb scenario). remix.app record not self-service. Android TBD.