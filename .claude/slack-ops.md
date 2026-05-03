# #ops Slack Channel — Remix Labs

**Coverage:** Feb 1 – May 2, 2026
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

## Mar 26–Apr 3 Promotion Logs

| Promotion PR     | Posted | Kicked Off | Mobile Beta | Notes                                                                                                                                       |
|------------------|--------|------------|-------------|---------------------------------------------------------------------------------------------------------------------------------------------|
| harmony/pull/368 | Mar 26 | Apr 2      | 2723 → 2749 | Arka approved Mar 31. Arvind's node-boundary fix (turntable/pull/11888) added to beta Apr 3 before prod promotion. Next beta to test: 2749. |
| harmony/pull/370 | Apr 2  | (pending)  | —           | Posted for next week. Team asked to comment if blocking issues.                                                                             |

**Turntable CI failing [Apr 2, Didier]:** CI blocked for a few hours — couldn't merge anything. Self-resolved.

**TT `update-users` job failing [Mar 31, Tyler]:** CircleCI job failing repeatedly. Chris: likely dependency mismatch; next mix-rs auto-update expected to fix.

## Apr 14–17 Promotion Logs

| Promotion PR     | Posted | Kicked Off | Mobile Beta | Notes                                                                                                                                                                                                  |
|------------------|--------|------------|-------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| harmony/pull/371 | Apr 14 | Apr 16     | 2788        | Superseded harmony/pull/370 — added turntable/pull/11920 to beta before promoting. Kicked off beta→release Apr 16; dev held (builder issues on dev). Mobile beta 2788 pushed as Lumber-targeted build. |
| harmony/pull/372 | Apr 17 | (pending)  | —           | Posted for next week. Team asked to comment if blocking issues.                                                                                                                                        |

## Apr 18–25: Stdlib ID Mismatch Blocks TT Deploys

**[Apr 22, Simon/Gerd]** Simon's latest TT merge to main triggered a CI failure:

```
Stdlib ID mismatch: protoquery = turntable — stopping build here
```

This blocked **both** Desktop deploy and remix.app deploy (remix.app promotion waits for Desktop to complete first). Result: nothing deployed that day. Simon requested decoupling remix.app from
Desktop's stdlib ID check.

**Gerd's response:** Not easily fixable in the current CI setup. Root fix: turntable should not need to know the stdlib ID at all — simplify the dependency graph. (See also:
`turntable stdLibId coupling` in engineering process notes.)

**Simon's follow-up:** No per-PR CI failure notifications available — would like notifications for own PR failures on main without noise from all other branches/PRs. Not implemented.

## Apr 25–May 2, 2026 Additions

**Weekly promotion [Apr 30, Chris]:** Kicked off weekly promotion — **skipping `beta.remix.app`** due to a component-mismatch bug (turntable/rmx-remix webcomp double-embed issue; ref: #bugbash). Will
find a nearby build with correct component set; otherwise defer.

**Desktop `release` channel: `0.11350.0` [May 1, Chris]:** New release pushed to desktop `release` channel — fixes the version incompatibility issues (including the `file.upload` FFI gap reported Apr
29).
