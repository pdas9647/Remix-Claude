# Standup Notes — Mar 2026 (Week 1)

**Coverage:** Mar 2–5, 2026
**Format:** Condensed bullets per standup. Google Doc link on each date for on-demand full transcript fetch.
**See also:** [standup-notes-mar-2026.md](./standup-notes-mar-2026.md) — Mar 10–13

---

## 2026-03-05 — [Google Doc](https://docs.google.com/document/d/1NQONGS79R6Uy0vcWyUpByEX71fG9J-VLrJfmUAtSNpw)

- **Preferences UI for home app (John Dismore + Didier)**: Building simple UI for power end users to set personal/sync preferences on mobile and extensions. Leveraging "preps" — JSON name-value pairs
  hosted in workspace, auto-synced to builder/mobile/extension. Use case: global preference lists (e.g., catalogs to sync for search). Desktops must also respect these prefs the same way.
- **Facet refactoring → standard library (Mark)**: Integrating Snowflake facet functionality into Lumber final flows. Lumber is the first project to consume facets as standard library items.
  Coordination needed between John, Vijay, Mukund, and Wilbur. Demo will use HubSpot data set.
- **HubSpot agent/widget separation into 3 apps (Wilbur Claudio)**: Splitting HubSpot agents + widget setup into 3 distinct applications: (1) web app for configuring service agent aliases to query
  HubSpot, (2) mobile app for configuring widget records, (3) separate project for service agents. Web setup experience nearly complete, working through bugs. Sync-up needed with John + Vijay on table
  configuration (not on Mark's plate).
- **2026 strategic initiatives (Vijay)**: Three thematic initiatives for the year: add value to HubSpot users, Zendesk users, and Snowflake users. Strategy: leverage these orgs as channels if
  possible, else go direct. Good headway with Snowflake and Zendesk already.
- **Lumber as key pattern (Mukund)**: Lumber project represents a pattern that must be implemented correctly to unlock other opportunities.
- **Platform scale concerns (Didier + Gerd)**: Platform has become a "monster" — fighting with build server. AI concern: people may believe AI can do everything, reducing need for platform tools.
  Questioning whether new clients would adopt such a large platform ecosystem.
- **Lumber client sensitivity (Didier)**: Lumber engineers view Remix as a threat — implies their internal team failed at reporting. Complex internal schemas. Not everyone at Lumber is receptive.
  Quick progress eventually speaks for itself, but situation is delicate.
- **Next steps**: Arvind to post about web components follow-up in #announcements. Vijay + John + Mukund + Wilbur sync on Lumber coordination + table config. Arvind + Vijay to discuss headless web
  components.

---

## 2026-03-04 — [Google Doc](https://docs.google.com/document/d/1j9U40Wg4YT0mOEHUGy3dQdjry9UoKAodAGLYsc4HnlM)

- **Windows menu deep-nested bug (Fred + Chris)**: Deep-nested issue in Windows menuing system traced to Muda code base (not Remix code). No new dependency release available; `cargo` file unchanged
  for ~1 month. Fred will attempt Tauri upgrade from v2.9 → v2.10. If dependencies don't fix it, Chris suggested bisect debugging to isolate the breaking change.
- **Lumber facets (Sirshendu)**: Created two new drop-down facets — (1) improved table drop-down from Snowflake facet with better error checks and UI, (2) simpler static drop-down picker.
- **Lumber project management reset (Didier)**: Meeting with Manish, Oleg, Anand. Key issues: client treating Remix as "shrink-wrapped" product rather than customizable platform; no PM ownership or
  capacity allocation on client side for Snowflake schema design and ETL. Client's Anand was running feature-checklist evaluation — not productive.
- **Phased delivery agreed**: Replaced unrealistic Feb 27 go-live with phased approach. Phase 1 = end-to-end thread on a few reports, demonstrated on Remix servers. Anand acknowledged Snowflake
  schema/ETL is client's responsibility.
- **Oleg owns requirements**: Significant win — Oleg (not Anand) now owns requirements definition. Follow-up call scheduled: Oleg + John + Arvind + Vijay to walk through first 3–4 concrete reports (no
  lengthy requirements doc; reports illustrate capabilities).
- **Internal productization track (Didier)**: Run parallel internal track — productize widgets, facets, charts using HubSpot data set (leveraging Wilber's sync work). Differentiates general components
  from client-specific consulting items. Doubles as demo material for partners (e.g., HubSpot personnel).

---

## 2026-03-03 — [Google Doc](https://docs.google.com/document/d/1Ehuv1xAFGWOuKHECLSudHKfGZOF_KapKkNo95P0dyC0)

- **Desktop release for Lumber**: Plan to release desktop version for internal testing before Lumber handoff. PR for handling invalid characters on input approved; may need minor adjustments before
  merge. Target: something out by end of day for India QA team (back tomorrow). Chris to tell Fred when to cut corresponding Windows build after Mac beta push.
- **Arrow key / error key input bug (Tauri)**: Unresolved issue with error keys on input — likely deep Tauri framework bug (other users reported, no permanent fix). Workaround: JavaScript event
  listener on keydown to suppress default handling at text input boundaries (beginning→left, end→right). Didier approved as workable. Affects text inputs "everywhere" including runtime — significant
  UX pain.
- **Lumber integration blocked by client**: Engineering deliverables largely complete, but Lumber hasn't provided clean testing environment. `QA remix.lumberfi.com` URL not accessible from QA admin
  panel. Proposed solution: provide Lumber a test HTML page + files to host, requiring just a button redirect to a Remix-controlled page ("Hello world") to confirm integration.
- **Environment variable overhaul (Gerd + team)**: Current env variables (backend/frontend base URLs) are confusing — originated from AMP world where one URL covered assets + APIs, no longer accurate.
  Gerd has PR to remove many confusing functions from env. Design needed: only two variables required (one for backend/Mixer, one for assets). Key user needs: links to builder, links to running app,
  API prefixes for app metadata. Must deprecate old variables, not change meaning. Chris + Simon + Gerd to figure out what Arvind needs and expose proper functions.
- **Theme/style revamp deployed**: Themes now treated as collections of styles (not executable). Changes auto-propagate to dependent apps — no more manual `make`/rebuild. Themes are separate .remix
  apps that get synced. Vijay questioned why not library assets; answer: .remix files enable computations + builder assets. Future consideration: push computed theme results to dependency-free library
  assets (JSON in settings).
- **Builder UX consistency (Didier)**: "New" button updated — now acts as button with search + description. Inline help for module types, consistent keyboard navigation/shortcuts for L1/L2.
- **Core sync agent updates in customer workspaces (Arvind + Vijay + John)**: Updating core service agents (`_rmx_sync`, `_rmx_prefs`, `_rmx_search`) across customer workspaces is unmanageable
  manually as workspace count grows. Debate: automatic vs manual updates. Security concern (John + Vijay): dynamically fetching executables is risky. Decision: manual one-click update by workspace
  admin for now. Tooling: management app showing which core apps are out of sync, admin initiates update with own permissions.
- **Table primitives (Tyler + Arvind)**: Tyler added native table primitives to builder + web components. Non-negotiable requirement (Arvind): any subtree (header cell, row) must be convertible to a
  list and piped into appropriate primitive as container — essential for building tables dynamically from data via loops. Tyler to test list-binding functionality. Also added inline ungroup button
  convenience feature.
- **Lumber reports ready for testing (Arvind)**: Reports app, catalog app, and Lumber home app are far along. Report lifecycle: copy .remix file from draft → published state within single workspace.
  Staging (draft/review/publish) should stay in one workspace — cross-workspace tooling for firewalled environments would be very difficult.
- **Chrome extension homepage**: Not started yet; Arvind planning a starter template.
- **File copy/overwrite API issue (Arvind)**: V1 API `copy` fails if destination exists. Team decided to keep current error-returning behavior (more features); workaround = delete-then-copy, managed
  via component.

---

## 2026-03-02 — [Google Doc](https://docs.google.com/document/d/1UIeN8IGXMnsnqmL3IC1laqEKFkhNistbkVEbq8olrz4)

- **Desktop bug fixes + mixer HTTP/protocol fix (Benedikt)**: Spent prior week on desktop bug squashing. Merged configurable HTTP port PR (mix-rs #948). Side-effect fix: when mixer exposed as HTTP,
  internal custom protocol URLs (`remix://localhost/a`) were surfacing externally — now external URLs are valid HTTP URLs. Improving app install procedure using E10 mechanism to check if
  reinstallation is actually needed, preventing unnecessary reinstalls of workspace apps.
- **MCP bridge revamp (Benedikt)**: Removing auto-start behavior where Remix launched automatically when Claude ("claw") started. New model: users must open Remix Desktop first; MCP tools are only
  available if Remix is already running. Previous auto-config was problematic.
- **Subprocess shell from External Actions (Vijay + Chris)**: Vijay proposed adding subprocess/shell invocation from desktop external actions to talk to CLI tools (Gemini CLI, cloud code). Chris:
  technically not difficult but security model needs careful design — what constraints to apply. Vijay: high value because many users have pre-built CLI tool setups on their desktops. No decision yet;
  security model is the blocker.
- **Lumber EKS deployment template (Chris)**: Built EKS (Amazon managed Kubernetes) template for deployed mixer setup — internal service + deployment components provided to Lumber team, who will wire
  into their cluster. Next step: determine what's needed to expose for external testing. Snowflake runtime still has `runtime.json` access issue (ongoing).
- **Windows release cut (Fred)**: Completed bug fixing round; cut latest Windows release on Friday (Feb 28). Desktop bug fixes done.
- **Build server for cloud agents (Gerd)**: Extended build server to support building cloud agents. Also improved build experience error handling for incremental builds with temporarily invalid code
  modules. Next: test everything, then create a plugin for selecting workspace service agents for deployment.
- **Parser refactoring for LSP (Oliver)**: Refactoring parser to separate type creation from pure syntax parsing — complex task involving expressions that mix types, statements, and phrases. Goal:
  greater flexibility and better control over Language Server Protocol (LSP) functionality. Long-term LSP discussion: Gerd noted it would require new Monaco editor wrapping; Didier emphasized LSP
  server should work with any editor (VS Code, Vim) and not require a custom web component.
