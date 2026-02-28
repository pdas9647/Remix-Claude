# #general Slack Channel — Remix Labs

**Coverage:** Dec 20, 2025 – Feb 27, 2026
**Channel ID:** C02BD0B8B

---

## Team Structure (from standup bot)

Daily standups Mon–Fri via https://meet.google.com/hrz-onty-twp

| Day       | Team                             | Members                                       |
|-----------|----------------------------------|-----------------------------------------------|
| Monday    | Platform I (backend/core)        | Benedikt, Chris, Fred, Gerd, Oliver, Shritesh |
| Tuesday   | Platform II (frontend/runtimes)  | Simon, Arvind, Didier, Tyler                  |
| Wednesday | Content I (Toolkits & Libraries) | Arka, Arpita, Nivesh, Padmanabha, Riju, Shawn |
| Thursday  | Content II                       | John, Mark, Wilber                            |
| Friday    | Go-to-market                     | Mukund, Reza, Suma, Vijay                     |

---

## Key Events

### Dec 26, 2025 — Company strategy

- Vijay delivered informal 2026 company strategy in standup ("No slides, no videos — just a great off the cuff discussion")

### Jan 5, 2026 — Chrome extension "Copilot" rename

- Tyler published new extension version: right-click sign-out added; runtime sends signal on bad token to force re-auth
- Simon raised Microsoft trademark concern on name "Copilot"; Mukund confirmed "Remix Copilot" is acceptable (multiple apps use the word in Chrome store)

### Jan 12, 2026 — Platform tech: blob URLs + stdlib fix

- **Gerd**: Fixed "stdlib ID mismatch" error — new desktop build is now blocked when upstream repos differ in stdlib ID
- Created amp-free rebuild scripts that recompile the 10 most recent projects as part of CI flows
- Added `blobCreate` FFI for blob URL management from Mix to work around 10MB scenegraph message limit (replaces long `data:` URLs for computed images)
    - PRs: protoquery #2226, groovebox #457, mix-rs #935

### Jan 15, 2026 — Repository Tool + scatterplot

- **Gerd**: Published Repository Tool walkthrough in Notion: https://www.notion.so/Repository-Tool-2e91d464528f800985eee93ec9fd5842
- **Mark**: regl-scatterplot web component working example at https://remix.remixlabs.com/e/edit/scatterplot/home
  — accepts parquet file as `data` in-binding; point selection returns metadata via out-bindings; deeper visualizations (bar charts) being added

### Jan 16, 2026 — GTM

- Mukund + Vijay: Actively trying to close 2–3 deals; John demoed to Salesforce

### Jan 20, 2026 — Repository Tool live

- Repository Tool available: https://remix-dev.remixlabs.com/e/edit/Repository
- Discussion: need a "publish read-only builder view" feature — not yet built; current read-only DB access not exposed in builder

### Jan 22, 2026 — Debug tip

- Gerd: Shared screenshot of how to enable "track messages" in JS log (requires setting in desktop app)

### Jan 23, 2026 — Chat + digital experiences demo

- Mukund: Demo recording: https://drive.google.com/file/d/14rMkRdtZL3OnBgL65kHSVRvminyxlICP/view
- Chris: Added entire team to standup calendar event → Gemini notes + recordings now auto-shared with team

### Jan 25–26, 2026 — India holiday

- India team out Jan 26 for Republic Day

### Jan 29, 2026 — Action item: _rmx_auth rollout

- Didier + Benedikt debugged and modified base `_rmx_auth` app on prod
- **Action item**: `_rmx_auth.remix` must be updated on every workspace; John Dismore and Wilber Claudio are the only ones with access to all workspaces — assigned to them

### Jan 30, 2026 — GTM pipeline update (Mukund)

- **Closing**: Lumber, Funda, Bomisco
- **New pipeline** (3 opportunities):
    - Starlight — chat → digital experiences
    - Atomic Work — analytics from HubSpot
    - Opt Health — chat → digital experiences
- **Needed**: Snowflake bootstrap experience (widgets + landing page) for Snowflake marketplace submission
- **Website**: Update underway to reflect data lake–centric messaging
- Chat demo link (Wilber): https://remix.app/run?_rmx_url=https://agt.files.remix.app/remix/_rmx_files/demo/chat.remix

### Feb 3, 2026 — Engineering planning meeting (Gemini summary)

- Chris proposed a planning process using GitHub issues, product-themed milestones (e.g., Desktop V1), and high-level product pages for leadership visibility
- **Biggest blocker**: lack of clear, consistent product goals from Vijay and John — constant direction changes and non-ticketed requests
- **Gerd**: primary high-level goal is to secure **Series A funding**, which creates ambiguity in product definition to appeal to investors
- **Gerd + Tyler**: skeptical the team can self-manage without a dedicated project manager
- **Agreed next steps**:
    - Chris + Didier to meet John, Vijay, Reza to establish top-down product planning + buy-in
    - Engineering team to use GitHub issues for any task >1 day (include labels, assignees)
    - Chris to organize GitHub board around product themes
    - Long-term: shift to separate bi-weekly sprint planning + retrospective meetings (not in daily standup)

### Feb 6, 2026 — Mixer API Swagger docs

- **Chris**: Mixer API now has full Swagger/OpenAPI spec: https://agt-dev.remixlabs.com/v1/swagger/
    - Non-v1 endpoints migrated to `v0` prefix (old endpoints still work but deprecated)
    - New runtime URL pattern: `/<screen>[/...]` serves `runtime.html` (similar to amp-backed web apps)
    - Simon PR turntable #11690: API cleanup; mix-rs PR #989: signals deduplication fix

### Feb 9, 2026 — Prod disk incident

- **Chris**: Prod agent server (500GB disk) filled up ~10PM ET Friday Feb 7
    - Cause: excessive activity logging in `remix` workspace starting ~3PM ET Friday (spike at ~6PM, full at ~10PM)
    - Disk resized; possible residual bad state in some apps
    - Period of concern: 10PM ET Fri Feb 7 – 11AM ET Mon Feb 9
- Benedikt out Feb 9–18
- Reza: Planning session with Manish at Lumber (Manish was travelling, call scheduled for Feb 10)

### Feb 23, 2026 — Oliver: Menhir parser decision

- Oliver explored menhir (parser-generator) for better parse error messages in the Mix compiler
- **Decision**: not worth switching — menhir has a different API and requires too much extra effort; will revisit only if more specific error messages become a concrete requirement

### Feb 25, 2026 — Chris: Strategic roadmap + GitHub board triage

- Post-Feb-24-standup summary from Chris with three concrete delivery tracks:
    1. **Desktop + mixer → Lumber**
    2. **Chrome extension → Orderly + Bomisco**
    3. **Mixer-in-SPCS + mobile app → Snowflake Marketplace**
- Post-delivery: use those experiences as proxies for next 20 customers
- **Longer-term focus**: content engineering lifecycle — builder UX, build server, platform robustness, engineering processes for component libraries
- **GitHub board**: https://github.com/orgs/remixlabs/projects/2/views/1
    - Engineering homework: move anything planned for this sprint (this week + next) to "ready to pick up"; triage "in progress" → active / close / "parked" column
- **Notion task list** (Chris maintaining): https://www.notion.so/1d71d464528f80a69c47d21033bc498c?v=1d71d464528f808199ff000ce98e1252
- Gemini summary of Feb 24 standup (recommended by Chris): https://docs.google.com/document/d/13Jxgr6nUeaQpI4W9O07aWXEpXV2Y7VZhsNK5C0gLmcE/edit?tab=t.cx0cp69o18m7

### Feb 27, 2026 — Gerd: Desktop update re-enabled

- Gerd announced: "Updating desktop is again possible (and recommended)" (🎉 party parrot x3)
- Implies desktop auto-update had been blocked/disabled previously and the blocker is now resolved

---

## Channel Tone & Patterns

- Direct, async, low formality; no formal sign-offs
- Standup bot posts daily Mon–Fri with Google Meet link; people post brief absence/late notices before the call
- Technical updates shared inline (PR numbers, Notion links, demo URLs, screenshots)
- GTM updates posted by Mukund on Fridays; customer pipeline status visible to whole team
- India team posts holiday notices a day ahead
