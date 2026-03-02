# #design Slack Channel — Remix Labs

**Coverage:** Dec 20, 2025 – Feb 28, 2026
**Channel ID:** C4BLR910S
**Purpose:** Design systems, visual IDE, icon and widget libraries

**Key participants:** Arvind (design lead), Didier (builder/runtime), Simon (UX/tester), Tyler (webcomp/desktop), Sirshendu/Padmanabha (bug reports)

---

## Key Decisions & Proposals

**Library Sync False-Positive [Feb 10]:** Changing left-side binding values after library sync always shows diff. Fix: for components, ignore exterior binding values in diff — use `in_param` values as canonical. Accepted for components; no solution for other node types.

**Theme/Styles Packaging Overhaul [Feb 14-17]:** No more `make` for themes — styles reflect immediately in builder after save. Removed code modules, make/rebuild/preview/open from theme menu. Theme only exportable with builder assets (no standalone executable). Arvind approved.

**TUI Grid Custom Cell Renderers [Feb 18-20]:** Tyler confirmed `rmx-base` webcomp works as custom renderer. Limitations: explicit row height/column width required; card data must be in column options. POC done; production TBD.

**TUI Grid Theme Loss in Nested .remix [Feb 7-22]:** Shadow DOM isolation breaks TUI styles inside `rmx-remix` webcomp. Fix: `no-shadow=true` attribute disables shadow DOM. Available on dev; needs prod rollout.

**L0 Desktop Organization [Feb 12]:** Proposal: simplify to "my local projects" + "my workspace projects". Git discussion: Gerd says git as-is won't work (JSON assets, custom checksums); needs tighter integration with git-like plugin. No sign-off.

**html2card Naming [Feb 11]:** Two components in `agt/remix-libraries`: (1) "Convert HTML to Live Card Node" — builder-time, interactive, bindings at L2. (2) "Convert HTML to Static Card Structure" — runtime, non-interactive, for AI-generated HTML.

**MIGRATE OFF `_rmx_style` [Feb 26] @all:** `_rmx_style`/`_rmx_styles` deprecated, not auto-loaded, won't sync to Desktop. All cards/components must use `rmx_tailwind`. All team members tagged.

**L1 constants modal: Cancel stays at L1 [Feb 26]:** When `make` errors from outdated-constants modal, Cancel wrongly goes to L0. Fix in progress (Simon).

**Repository DB rename deferred [Feb 28]:** Rename to `_rmx_repository` deferred to post-Lumber. Needs tooling for existing DB renames.

---

## Open Design Items

**Tailwind text color gap [Dec 20]:** Only `text-em-500` available; needs 600+ for contrast with buttons.

**Color binding → theme lookup [Dec 20]:** Regular color bindings should support theme lookup with custom hex override. Agreed, not built.

**Binding change triggering upload [Dec 20]:** Consumer edits (e.g. currency) trigger "upload changes" — no way to distinguish consumer edit vs structural change. No clean solution.

**Screen state path picker [Feb 3]:** Paths typed manually as string arrays; no visual picker. Proposal: derive shape from Set State invocations. No resolution.

**Library search ranking [Feb 10]:** Exact match appears below partial match (alphabetical sort, no ranking). Agreed: exact match must rank first. Open.

---

## UX Bug Reports

**Const picker unreachable [Feb 21]:** Can't reach const picker in card editor properties panel. Filed, unresolved.

**+ ADD FIELD buried [Feb 13]:** Button far below long object in component out params. Needs max-height with overflow. Arvind agreed.

**Multi-select chevron overflow [Feb 12]:** Duplicate components in `remix_labs` and `remix-forms` cause confusion. Action: consolidate into `remix_labs`.

**h-xls/h-xs invalid classes [Feb 12]:** Copy-paste error (all set to 0). Fixed by Arvind; pushed to india/dev/beta.

**my-6 missing [Feb 9]:** Typo (dash omitted). Fixed by Arvind; pushed.

**Desktop zoom missing [Jan 29]:** No Ctrl+/Ctrl- support in Tauri. Growing list of browser features missing from Desktop.

**Number decimal alignment [Dec 22]:** No component for tabular alignment. Workaround: `font-variant-numeric: tabular-nums` + right-align.

**Output area not resizable [Dec 23]:** AI response area fixed height. Arvind agreed to fix.

---

## Channel Patterns

- Arvind drives proposals, Didier implements; Simon probes rationale
- Screenshots + builder links on every UI report
- "shipitt" = approved; no formal sign-off ceremony
