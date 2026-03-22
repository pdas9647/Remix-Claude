# #ops Slack Channel — Remix Labs

**Coverage:** Feb 1 – Mar 21, 2026
**Channel ID:** C07P57B89
**Purpose:** Weekly production release promotion workflow (beta → production)

---

## Channel Pattern

- **Chris Vermilion** manages all promotions. Workflow:
    1. Posts a "Next week's promotion PR" linking to `github.com/remixlabs/harmony/pull/NNN` — asks team to comment if they see issues in beta
    2. The following week: posts "kicking this off" to start the promotion
- **Arka Bhattacharya** handles mobile beta QA (TestFlight builds); files issues in #bugbash
- Promotion PRs are against the `remixlabs/harmony` repo (infra/release staging)
- Issues blocking promotion get discussed inline; once cleared, Chris submits the mobile build and posts a new beta number

---

## Promotion Log (Feb–Mar 2026)

| Promotion PR     | Posted | Kicked Off | Mobile Beta | Notes                                                                                                                                                                                                                          |
|------------------|--------|------------|-------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| harmony/pull/354 | Feb 5  | Feb 5      | 2532 → 2552 | Android CI failure (retried, resolved); Arka filed #bugbash issue                                                                                                                                                              |
| harmony/pull/356 | Feb 6  | Feb 12     | —           | No mobile issues noted                                                                                                                                                                                                         |
| harmony/pull/358 | Feb 12 | Feb 19     | 2577 → 2594 | Mobile issue found; confirmed userspace (not platform); 2577 submitted, new beta 2594                                                                                                                                          |
| harmony/pull/359 | Feb 19 | Feb 26     | —           | Kicked off Feb 26; new beta pushed Feb 28 including Gerd's fix                                                                                                                                                                 |
| harmony/pull/363 | Feb 28 | (pending)  | —           | Posted for "next week or whenever we want to promote beta→release"                                                                                                                                                             |
| harmony/pull/364 | Mar 4  | Mar 5      | 2646        | Mobile beta 2646 pushed; iOS widget bug still present; promotion kicked off without mobile submission. Simon found annoying bug (turntable/pull/11785), asked for dev→beta bump.                                               |
| harmony/pull/367 | Mar 12 | (pending)  | 2674        | New beta 2674 pushed. **No mobile submission this week** (no beta promotion last week). Arka asked to test 2674 when possible. Chris: main Lumber deliverables are content demos now; full deployment likely weeks/months out. |

**Desktop startup failure on agt.remixlabs.com 502 [Mar 3]:** Simon couldn't start Desktop — `_rmx_sync` failed when agent server returned 502. Self-resolved after a few minutes. Gerd: should ignore
sync errors and allow startup (important for offline use). Arvind: avoid scary system error; show friendly dismissible message. Benedikt: offline use blocked by token verification requirement. Filed:
mix-rs/issues/1043 — sync failure should not be fatal.

---

## Notes

- Android CI failures are rare but happen; retry resolves them
- "Mobile issue in userspace" (Feb ~18): issue found in beta 2577 was traced to app-level code, not platform — cleared for submission
- The prod agent server disk incident (Feb 7–9) caused a missed Slack notification for Arka's report, delaying the CI retry
- Beta promotion cadence was paused ~1 week (no promotion week of Mar 7) due to Lumber delivery focus; resumed Mar 12

---

## Channel Tone & Patterns

- Entirely operational; no design or architecture discussions
- Chris is the sole release driver; Arka is the sole mobile QA voice
- Very short messages; threads only for blockers or build hand-offs

| (no harmony PR) | Mar 19 | Mar 19 | — | Chris kicked off a promotion but held dev→beta due to OPFS regressions not fully settled. |
