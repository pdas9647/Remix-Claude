# Standup Notes — May 2026 (Part 2)

**Coverage:** May 11–15, 2026
**Format:** Condensed bullets per standup. Google Doc link on each date for on-demand full transcript fetch.
**See also:** [standup-notes-may-2026.md](./standup-notes-may-2026.md) — May 1–8 | [standup-notes-apr-2026-3.md](./standup-notes-apr-2026-3.md) — Apr 27–30

---

## 2026-05-15 — [Google Doc](https://docs.google.com/document/d/1xTyb9awc7A7KIQZ5gslbQjMZQymf_1Q6XuZUB1L2gV4)

- **Snowflake Summit (Jun 1-4, Moscone Center SF)**: Final prep underway.
- **Decision: Private Snowflake marketplace listing** (not public). Under Stripe approval; leveraging earlier public listing work. Chris to complete within 2-day turnaround.
- **Target: Mid-market** — 250–5,000 employees using data lakes. Two personas: technical (data ops, engineering) + business (sales, marketing, RevOps, CS).
- **Core offering**: "Analytic apps" — enterprise decision-making with Remix + AI.
- **Demo themes**: Customer 360 stories, custom mobile widgets, dashboard quotes (Lumber-style), narrative insights (automated reports + takeaways), CDP→digital experience pipeline.
- **Campaigns**: (1) Co-hosted networking event with USD on Jun 2 at Local Edition (~40 people; USD = services firm with Snowflake practice in healthcare/finserv/retail, NDA in place); (2) expo
  booth; (3) targeted campaign to 20-40 companies from AE list.
- **LinkedIn 2-wave paid ads**: Wave 1 (7 days live) = market pain points, business + technical persona ad sets. Wave 2 (~1 week out) = summit-specific messaging.
- **Twilio approval**: Awaiting brand verification from Twilio + Aegis (3rd-party agency). Awaiting Starlight Twilio approval too.
- **Lumber status**: Testing/approval phase. Manish: internal approval pending with Greg. Plan: 2-hour Remix Studio hands-on with Manish + Arbend + Vijay to unblock reporting.

---

## 2026-05-14 — [Google Doc](https://docs.google.com/document/d/1oZ7rEBXhr00XRTns508LQusraM20xJkkJhZv8fQnSbo)

- **Production deploy without QA**: Wilbur unavailable, QA scripts not run; decision: deploy anyway (nobody negatively impacted).
- **DECISION: Discontinue waiting for Lumber team input** — no feedback for a week. Vijay frustrated; Reza tried semi-weekly sync, unclear progress.
- **Architectural framework (Vijay)**: Goal isn't "page builder" — it's clean architecture using service agents + facets + tables for rapid app dev.
- **STRATEGIC Parquet emphasis**: Snowflake querying expensive; market shifting to Parquet. Service agents enable fine-grained access control on Parquet (hard to replicate inside Snowflake).
- **Service agent alias mechanics**: Aliases wrap base agents + apply fixed params (compute settings, roles, DB tables). CTE pattern: construct SQL by applying facets to base queries. Compositional,
  static params. Alias creation = admin function in workspace. No recursive aliasing (prevents logical loops).
- **Alias use case**: Hide private keys / account IDs from devs.
- **Workspace tooling (John)**: Drill into agents, manage aliases, clone for refined permissions, set static SQL params.
- **UX decision**: Tooling launch via dedicated menu (not tiles on scene graph).
- **AI-assisted alias generation (John, prototype)**: Anthropic API parses SQL → generates parameter shapes. Goal: integrate into page builder for inline alias definition.
- **DECISION: Builder vs Runtime separation** — avoid conflation. Builder view "spectacular" for creating/testing; runtime separate for end users.
- **Strategic positioning**: Platform as "middleware" — federation layer across Snowflake, DataDog, etc. Live disposable microservices in <5 min = competitive moat.
- **Auditability (Reza)**: Admins need permission tracking. Have metadata APIs + deployment logs; need UI for version history + audit logs to readable directory (e.g., CloudFront).
- **Decision: Agent Alias framework past ideation** → execution + productization phase.

---

## 2026-05-13 — [Google Doc](https://docs.google.com/document/d/1tjGQx9BJXjWi7ueBDTAOERlLB9gyDzOIXBSlPey1Hm4)

- **Agentic search demo (Padma)**: Crackle fashion dataset for US team's e-commerce retail-customer demo. ~500 data points in remote DB. NL → structured query (category, product name, availability
  fields). Examples: "show me blue apparel for woman", "I need shoe suggestions for a dinner party".
- **Chat integration (John)**: Chat client exposes search; chat AI generates NL phrase → shopping experience via Padma's agents.
- **Arka**: Investigating Snowflake's semantic layering service.
- **Website (Sirshendu)**: Component fixes + Snowflake marketing campaign collaboration.
- **DECISION: Deploy current website phase immediately**; Phase 2 = pure HTML / VJS tool (simpler, avoids overloading). Arvin to review one remaining item.
- **Personal**: Nivesh back in Bangalore after wedding.

---

## 2026-05-12 — [Google Doc](https://docs.google.com/document/d/1K0Sce3ylpLltJ_EO36uvGm0jvpkPqXL9mnK1gypj4SY)

- **RCM2 CI integration**: Most protoquery CI works with new code. Critical remaining: `update users` flow. Plan: port ≥3 repos before full RCM rename (drop RCM2 name; only one binary).
- **Decision: stick with standard JSON** for config (compat with tools). Future: `--config <file>` CLI flag for alt configs. Tyler's MJS-comment-out for local deps not replicated; only Tyler uses
  RCM's local-dir feature.
- **Next repos**: Mix RS or Products (Windows compat key driver). Benedikt has open PR for Products → desktop open with RCM2.
- **DECISION: RCM2 Windows binary first priority** — must be built in CI, placed in RCM repo. Fred owns.
- **DECISION: Public curl download for RCM2** binary — simplifies Windows CI bootstrap (includes Google SDK, no prior auth needed). Replaces gsutil + g-cloud SDK + Python.
- **AMP REMOVAL DEBATE**: Two parts — (1) AMP builder removal: tractable, internal reliance minor; (2) AMP runtime: mobile depends, significant effort, not high priority. Consensus: focus on AMP
  builder transition (becoming unsupported).
- **Transition plan**: Phased popup warning. Fred: 2 weeks. Didier: 60 days. Migration mechanisms: download .doremix → drag-drop into mixer-pointed builder (slow); OR desktop plugin connecting to AMP
  DB to copy edited databases (Gerd's idea).
- **Centralized sharing instance**: Use `dev.remix.app` builder → `agdev` server as collab replacement. Gerd: one mixer instance (like remix.app) as default sharing point.
- **React wrapper (Didier)**: Generalized React component wrapping rmx-remix; helps Lumber integration. Publishing to NPM blocked — `@Remix_Labs` namespace claimed internally but credentials lost. NPM
  90-day token renewal pain.
- **Simon**: New asset prefix in (fixes stdlib pack on remix.app). New pagination node (built-in, no function node needed). L2 ripple-response regression fixed by reverting + new response handler (
  doesn't break pub-sub).
- **Tyler**: Image primitive replaced with web component supporting fallback URL. New `DB key suffix` attribute on web component prevents same .remix source opening different DBs when embedded
  multiple times on a page.

---

## 2026-05-11 — [Google Doc](https://docs.google.com/document/d/1o7d05DOwbETRG6dP6VPDrW3v2M40s7p7wXnWQW-g1NU)

- **Repo restructuring (Benedikt)**: Creating `products` repo. CI build-ID conflict — potential fix: bump major version to restart minor counter (could land at v1). Channel switching now requires
  confirmation step.
- **S3 API hotfix (Chris)**: Latent bug in upstream Rust S3 library — file deletion didn't set content-length header → signature check failed. Pinned to recent PR commit. Hotfixed to main + production
  server.
- **Lumber QA channel cleanup (John)**: Concern about clients on QA vs release channel. Didier: current install link = release channel. Reza to switch any Lumber users still on QA.
- **Lumber weekly test script (John + Reza)**: Build script (like Starlite project) for India team to run weekly; synced with engineering release timeline.
- **Snowflake testing (Chris)**: Listing edge cases on initial install; tested account in different region. Open: push more sophisticated feature into public listing — Vijay + Chris discussion
  post-standup.
- **RCM2 GA (Gerd)**: Tested with one repo. Written in Go, Windows compat, removes gsutil dependency. Repos must switch `rcm.config.mjs` → `rcm.config.json`. Critical for Windows desktop CI builds.
- **Fred**: Rewriting aggregation function to accept different param kinds (ordering + individual groupings). Will enable Windows desktop builds in CircleCI once RCM2 validated.
- **JSON printer PR (Gerd)**: Formatted JSON output with indentation + mode to minimize line feeds.
- **De-ampification of compiler (Gerd)**: Protoquery tests run without AMP now. Keeping AMP tests for mobile only. Oliver: switching old mixed parser → new one.
