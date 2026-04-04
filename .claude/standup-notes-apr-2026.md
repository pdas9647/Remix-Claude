# Standup Notes — Apr 2026

**Coverage:** Apr 1–3, 2026
**Format:** Condensed bullets per standup. Google Doc link on each date for on-demand full transcript fetch.
**See also:** [standup-notes-mar-2026-3.md](./standup-notes-mar-2026-3.md) — Mar 24–31

---

## 2026-04-03 — [Google Doc](https://docs.google.com/document/d/1C7KPRTLPpbeXdbjBcrZJMW2SsgN5XljJfSykFbueaoc)

- **Sales doubled → offsite back on agenda**: New engagement revenue ~$800/month (highest in ~5 cycles, comparable to Lumber). John: revenue less important than proving the RCS solution for future
  sales.
- **RCS / federal credit unions (30-day trial)**: RCS solution in 30-day trial with federal credit unions for marketing campaign conversion, outreach, and engagement. Demos complete; setting up
  experience for client. Team positioned to become default interface for these interactions.
- **Intercom integration plan**: Allow clients to send RCS from Intercom. Short-term: client uses Remix conversation view. Vijay: significant future revenue via AI-automated RCS experiences.
- **RCS submission to Twilio complete**: Submitted; Sumal credited for helping with required details. Campaign + landing page + value story being built for RCS marketing.
- **Snowflake marketplace submission**: Very close (Chris: platform in good state). Remaining: SPCS environment quirks, packaging demo data, onboarding flow design. Streamlit app discussed as
  front-end for permissions/data setup — not blocking submission.
- **Sales pipeline**:
    - Starlight USD (SI with Snowflake experience): expected to close soon.
    - Zendesk partner discussions progressing; next meeting next week.
    - Meeting with lead of Google's RCS team secured in ~10–12 days.
- **Priorities (Vijay)**: Push hard on pipeline. Urgent: resolve Lumber desktop delivery ASAP. Demo of Vijay's work (Arvind to present) deferred to Monday.

---

## 2026-04-02 — [Google Doc](https://docs.google.com/document/d/1Zwo02MIMhrldH_TT8-oA392tEOvG1vYN7qLxaBAliQo)

- **RCS flow status (Wilber Claudio)**: Service agent app for calendar picking mostly complete; scheduling flow nearly done. John to update cards post-standup.
- **Twilio agent tooling**: New tooling to accelerate Twilio agent setup — covers all Remix edge cases, auto-deploys agents + creates aliases from Twilio info. Some manual Twilio console config still
  required.
- **System plugin versioning decision**: Cache-control header conflict between dev/beta and prod Cloudflare environments — no single file upload solution covers all environments. Decision: version
  system plugins (separate versions for dev, beta, prod). Two tasks: (1) short-term fix, (2) longer-term move system plugin registration to preferences for workspace admins.
- **TUI charting components (John Dismore)**: New components: bubble scatter, column stack, column, bar — all with numeric filter. No loading state yet (all refresh at once). All in library; Lumber
  library asset updated.
- **Chart architecture**: Config-driven — base SQL uses standard field names; chart config maps X/Y/label. Three parts: aggregator (base query + facets), SQL runner, charting component (reshapes
  data). Max height calculated client-side; stack sum calculated by chart.
- **Lumber (Mark Loevenstein)**: Bug fixing + table cleanup; supporting various facet types.
- **Post-standup lumber huddle**: Reza, Mark, John, Vijay, Arvind to review Reza's lumber proposal + timeline + task assignments.
- **RCS factory enablement**: Vijay: focus on completing "first of a kind" experience before enabling factory flow. John: RCS component is independent of experience — only sends message + link to
  agnostic .remix file.

---

## 2026-04-01 — [Google Doc](https://docs.google.com/document/d/1pnKfuBBbLreaCMxPwlSKw3aCo9c-QWxVe4G69Z4nzyw)

- **Lumber (India team)**: No new updates; Padmanabha Das confirmed nothing to report.
- **Lumber SQL/CTE rethink (Reza Mohsin)**: Current approach needs rethinking for better report functionality. Reza to work through examples first, then bring in Vijay, John, Mark.
- **SPCS Snowflake Snowpark widget preview flow (Padmanabha Das)**: Goal — user selects table, enters human prompt → Cortex generates + runs SQL from table metadata → widget preview in app. Blocked:
  cannot obtain table DDL in consumer-side application. Chris investigating workaround. Fallback: user provides DDL manually.
- **JSX/embedding follow-up (Vijay)**: Smaller group meeting with Simon, Didier, John on JSX-side work. John to schedule.
- **Personal**: Nivesh Mishra getting married Apr 25; relocating to Bangalore.
