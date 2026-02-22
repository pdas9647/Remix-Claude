# #design Slack Channel — Remix Labs

**Coverage:** Dec 20, 2025 – Feb 23, 2026
**Channel ID:** C4BLR910S
**Purpose:** All things design: design systems, visual IDE, icon and widget libraries

---

## Key Decisions & Proposals

### [Feb 10] Library Sync False-Positive Problem (Arvind)

**Problem:** Changing left-side binding values on an asset after syncing from library always shows a diff, even when no real component change occurred. Users can never tell if sync indicator is
genuine.
**Proposal (Arvind):** For *components only*, ignore L2 exterior binding values when computing diff — use internal `in_param` values as the stable canonical version. Author behaviour change required:
set default left-side values inside the component's `in_params`, not on the exterior binding.
**Discussion:** Wilber flagged inconsistency with non-component nodes. Didier outlined 3 options: (1) introduce dual values everywhere (too complex), (2) remove dual values from comps (rules out), (3)
current proposal — accepted as pragmatic. Service Agent Connect nodes have no L3 so can't adopt dual-value approach.
**Status:** Proposal accepted for components; no solution yet for other node types.

### [Feb 14–17] Theme/Styles Packaging Overhaul (Didier)

**Change:** Removing the need to `make` a theme. After saving the styles module, theme changes reflect *immediately* in builder L2 view (like changing local styles). Only `.remix`/preview/webview
still requires a `make app` to update.
**Additional:** Removing code modules from themes (no demand; if function needed, define in styles module). Removing `make`, rebuild, preview, open from theme menu. System modules should not be
deletable — bug to file.
**Decided:** Arvind approved ("shipitt"). Theme is only exportable with builder assets (no standalone executable).

### [Feb 18–20] TUI Grid Custom Cell Renderers (Arvind + Tyler)

**Question:** Can Remix "send cards" into TUI grid cells using custom renderers?
**Answer:** Yes — Tyler confirmed working using `rmx-base` webcomp as custom renderer. Limitations: must set explicit row height and column width in grid options; card data must be in column options (
painful). Improvement path: use Vijay's json-data→card webcomp.
**Status:** Proof of concept done; production integration TBD.

### [Feb 7 → Feb 22] TUI Grid Loses Theme in Nested `.remix` (Arvind + Tyler)

**Problem:** TUI grid loses all styling when inside a `rmx-remix` webcomp loaded via `.remix` file inside another `.remix`-based catalog app (shadow DOM isolation breaks styles).
**Affects:** Lumber catalog table configurator, chat demo Experiences.
**Fix:** Tyler added `no-shadow` attribute to `rmx-remix` node. Set `no-shadow=true` to disable shadow DOM and restore TUI styles. Working on dev.remix.app and dev builder.
**Status:** Fix available on dev; needs production rollout.

### [Feb 12] L0 Desktop Project Organization (Arvind)

**Proposal:** Simplify L0 to two sections — "my local projects" (top) and "my workspace projects" (below). Hide "other projects" if empty. Remixers (multi-workspace) are a special case.
**Git discussion:** Gerd: git as-is won't work — assets are JSON-based with custom checksums; 80% of work is mapping DB↔storage, git only addresses 20%. Vijay: libgit with proper file/folder mapping
could help. Current db concepts leak into file hierarchy. Needs tighter integration with Gerd's git-like plugin.
**Status:** Proposal discussed, no explicit sign-off.

### [Feb 11] html2card Naming Finalised (Didier)

Two distinct components in `agt / remix-libraries`:

- **"Convert HTML to a Live Card Node"** — builder-time, interactive, produces node with bindings at L2 (old html2card)
- **"Convert HTML to a Static Card Structure"** — runtime, non-interactive, invoked like an action/agent and returns a static card binding (new, for displaying Claude-generated HTML)

### [Dec 20] Tailwind Text Color Coverage (Simon + Arvind)

**Issue:** Only 1 visible font color option in the theme, not enough contrast vs white backgrounds and doesn't match button style. Tailwind text colors use the `text-em*` prefix — Simon found only `text-em-500` available; needs 600 weight for sufficient contrast with buttons.
**Status:** Open gap in tailwind text color range; broader `text-em-600+` not yet added.

### [Dec 20] Color Binding → Theme Lookup (Simon + Arvind)

**Proposal:** Regular colour bindings should be able to do a theme lookup, with a custom hex override when needed. Arvind: agreed, need to build this mechanism.
**Status:** Open design item.

### [Dec 20] Binding Change Triggering Upload (Simon + Didier + Arvind)

**Issue:** Using a foreseen binding edit point (e.g., changing currency on a control) triggers an "upload changes" prompt, implying the user is an author/collaborator.
**Root cause:** Static binding values are stored with catalog asset — no way to distinguish "expected consumer edit" vs. "structural component change."
**Discussed:** Simon suggested config objects should live *inside* component (not exposed externally); Arvind: "If an assembler wants to push changes, that's a variant." Didier: problem exists for all
tiles, not just components. No clean general solution found.
**Status:** Open design problem, no resolution.

---

## UX Issues & Bug Reports

### [Feb 21] Const Picker Unreachable (Simon)

Can't reach const picker in card editor properties panel for a text node. Bug filed, no thread resolution.

### [Feb 16] Node Number Repetition in L2 (Simon)

Repeated node IDs shown when "show system ids" is on. Clarified by Didier: these are internal stable IDs for group nodes, introduced to support storing groups in the catalog and reconnecting nodes
post-sync (PR #11588). Not a bug.

### [Feb 13] + ADD FIELD Buried Under Long Object (Padmanabha)

When a long object flows through component out params, the `+ ADD FIELD` button is far below all content. Needs max-height constraint with overflow or nested scrollview. Arvind agreed.

### [Feb 12] Multi-Select Menu Chevron Overflow (Padmanabha)

Dropdown chevron pushed outside the box when multiple items are selected. Root cause (Arvind): duplicate `string - multi select menu` components exist in both `remix_labs` and `remix-forms`
workspaces — causes naming confusion. Action: consolidate `remix-forms` assets into `remix_labs` (Wilber + John).

### [Feb 12] h-xls / h-xs Classes Invalid (Sirshendu)

Tailwind library had `h-xls`, `h-xs` height classes (copy-paste error) all set to 0. Arvind deleted them and pushed standard height classes (`h-24`, `h-32`, etc.) to india boxes, dev, beta.

### [Feb 10] Library Search Ranking (Sirshendu)

Exact asset name match appearing below a partial match due to alphabetical sort + no ranking algorithm. Proposal: implement proper ranking (drop server+collection grouping for search results). Arvind:
ranking is better UX. Wilber: exact match must always rank first. Grouping by workspace useful for exploration but not for search.
**Status:** Open improvement.

### [Feb 9] Tailwind `my-6` Missing (Sirshendu)

`my-6` (margin-y 6) missing from tailwind library — typo (dash omitted). Fixed by Arvind; pushed to india boxes + dev/beta. Desktop auto-update pending.

### [Feb 3] Screen State Path Picker (Arvind)

Screen/app state paths must be typed manually as string arrays — no visual picker. Proposal: derive known shape from Set State invocations at runtime and surface a visual picker. Design question; no
resolution yet.

### [Jan 29] Desktop Zoom (Sirshendu)

No zoom/scale support in Remix Desktop. Ctrl+/Ctrl− not supported. Tyler notes growing list of browser features missing from Desktop: bookmarks, tabs, beforeunload, zoom, browser title updates.
Arvind: zoom not a deliberate feature. Tyler: reason to reconsider browser-based builder.

### [Jan 2] Refresh vs Projects Tab (Simon)

"Refresh" in Studio only refreshes the libraries tab (redownloads catalog libs), not the project list. Clarified by Didier — expected behaviour, not a bug.

### [Dec 23] Output Area Not Resizable (Padmanabha)

AI response output area is fixed height — hard to follow long responses. Arvind agreed to fix.

### [Dec 22] Number Decimal Alignment (Simon)

No component for tabular number alignment. Gerd suggested `font-variant-numeric: tabular-nums` + right-align; Didier suggested fixed-width font + right-align.

### [Dec 20] Tailwind Group Cascade for Icons (Simon)

Tailwind group styles cascade to child text fields but icon sizes may not follow. Arvind: surprised, everything should cascade. Font colors do cascade correctly.

---

## Channel Tone & Patterns

- Primary participants: Arvind (design/platform lead), Didier (builder/runtime), Simon (UX observer/tester), Tyler (webcomp/desktop), Sirshendu/Padmanabha (users surfacing bugs)
- Format: screenshots attached to every UI report, specific builder links provided for bugs
- Arvind drives proposals, Didier implements; Simon probes rationale as power user
- Tyler often surfaces Desktop-vs-browser tension
- Decisions stated directly; "shipitt" = approved. No formal sign-off ceremony.