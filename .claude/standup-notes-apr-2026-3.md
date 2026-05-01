# Standup Notes — Apr 2026 (Part 3)

**Coverage:** Apr 27–30, 2026
**Format:** Condensed bullets per standup. Google Doc link on each date for on-demand full transcript fetch.
**See also:** [standup-notes-apr-2026-2.md](./standup-notes-apr-2026-2.md) — Apr 13–24 | [standup-notes-may-2026.md](./standup-notes-may-2026.md) — May
1+ | [standup-notes-apr-2026.md](./standup-notes-apr-2026.md) — Apr 1–10

---

## 2026-04-30 — [Google Doc](https://docs.google.com/document/d/1aOAipN1iUpXFieMiJeGzBqYLMtETpz-iBQQLCGihMGI)

*(Two of three Apr 30 sessions unrecorded — Gemini transcription failed.)*

- **Lumber QA channel consolidation**: Chris fixed release bug; new beta + release builds shortly. Decision: shift Lumber QA branch to single combined release build. India team to run combined
  Starlight + Lumber QA before promoting to single release for all customers.
- **Multicolumn text search facet (Mark)**: Integrated into facet builder; users select multiple columns + operation (e.g. "contains"). Server-side rerun (not client-side). UI challenge:
  distinguishing multi-name facets.
- **Starlight (Wilber)**: Twilio submission at brand-verification stage; ~2+ weeks expected.
- **Next**: Onboard Nat onto desktop to send RCS messages to testers.
- **Assembly service issues**: Mark hit issues using Vijay's assembly service to build a simple report; follow-up scheduled.

---

## 2026-04-29 — [Google Doc](https://docs.google.com/document/d/1In22sQqzYhmBhd2bpmg6oVBjuztxQEO5eYiIgXzvry8)

- **App sync data-loss issue**: Setting up sync record without builder assets DELETES local builder assets (default = delete-app-before-install). Decision: change default to "don't override". Need
  sync-status UI on desktop (which apps syncing/failed/succeeded + warnings).
- **Vibecode — front-end composition layer (Vijay)**: Rust-based system compiles builder assets into standalone JS web components. Output ~50 KB uncompressed HTML with inline web-component code; full
  compressed experience under 10 KB. No virtual DOM; hard-binds reactivity. Transpiles function nodes to JS for front-end-only use cases. Repo on GitHub.
- **Targets**: Landing pages, simple dashboards, RCS/WhatsApp app-like experiences, blogs, webinar pages. Embeddable in Snowflake container as composition surface.
- **Compiled-component features**: Interactive elements (decision tiles, multi-select pills with reorderable options, conditional display).
- **Tyler**: Posted web-component version supporting server-side sort with agent demo. Rich cells / card renderers blocked until custom table primitives ship.
- **Lumber evaluation (Craig, design PM)**: Doing eval over couple days — need more documentation so he doesn't get stuck building reports.

---

## 2026-04-28 — [Google Doc](https://docs.google.com/document/d/1zSaMzqsa8L-flNPUdqfpgS3QhVSRViWoIYQ03LikDBg)

- **Repo consolidation decision (4-repo, 2-level)**: Products repo (starting with desktop from mix-rs) + 3 component repos: **core** (Mix core + Groupbox + Mix Run), **turntable**, **protoquery**.
  Component repos have no build-time deps on each other; only feed into products. Reduces dep graph from 5-6 levels to 2.
- **New CI flow**: Component PR triggers branch+PR in product repo for integration tests; component PR can't merge until product CI passes. Integration tests on final merge only, not every commit.
- **RCM Go port**: Replaces unmaintainable TypeScript/CX-script version; enables G-Cloud in CI via Google's native lib; reduces image size ~1 GB.
- **RCM extension**: Proposed test-only deps (no lock entry, like branch entries).
- **Custom table primitives (Tyler + Arvind + Didier)**: Abandoning 2E table web component — can't pass entire card/cell. Decision: build own primitives starting from Gerd's previous performant
  solution. Column resizing lost short-term.
- **Simon**: Standalone website to run .remix file → led to OAuth flow fixes.
- **Didier**: Login flow for embedded use cases; race condition fixed. Open: framework vs screen responsibility for Lumber→M token exchange.

---

## 2026-04-27 — [Google Doc](https://docs.google.com/document/d/1GXKJ0spT41YlxxoX8RmKHOFHwPGXL9oNe6TXgOAp5Nk)

- **STRATEGIC: Parquet backend priority (Vijay)**: Productization focus shifts to consuming Parquet on backend (already supported). Vendor-neutral, cost-effective, analytical workload. Not enabling
  for HD/HD-dev (compute-heavy → safer behind firewall). Targets: companies on Snowflake/BigQuery for distribution.
- **Snowflake listing rejected**: Misconfigured permission grants in setup flow → fresh installs fail. Chris to rework OAuth story for desktop+web; sync with Wilbur on workspace setup flow; document
  for Lumber team.
- **RMX Remix unification (Didier + Simon)**: Light version (dynamic mix-core link) needed for embedded cases like Lumber where Remix doesn't control the page. Plan: unify versions in next few days;
  deploy assets on remix.app.
- **Lumber update**: Greg (UX designer, NOT originally intended first user) used UI faster than Oleg. Manish asked if Greg should be first to build a report. Lumber in assessment phase, not billed
  yet. Competing with Metabase on Postgres; may not move to Snowflake.
- **Polish list (Arvind)**: Productization-task list to capture generic polish items needed for both Lumber + Snowflake submission.
- **Native editing**: VS Code suggested as starting native editor; detailed discussion deferred.
- **Gerd**: Repo block plugin v2 fully tested; new dashboard plugin available (needs turntable enable). Open: codegen during compilation when cloning repo to desktop + entering app directly.
- **Oliver**: IST + type trigger replaces old ping mechanism; ready to merge pending G approval.
- **Fred**: Bug fix from Bird; one test failed due to optimization issue.
- **Benedikt**: Panic-backtrace logging PR for Windows desktop; core-integration refactor for mixed assets (recurring CI error).
