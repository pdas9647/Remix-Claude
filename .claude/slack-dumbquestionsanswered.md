# #dumbquestionsanswered Slack Channel — Remix Labs

**Coverage:** Feb 16 – Apr 2, 2026
**Channel ID:** C86KWF7MG
**Purpose:** "the dumbness of the programmer has no limits" — quick Q&A, platform clarifications

---

## [Feb 20, 61 replies] `env.runtimeURL` deprecation (Arvind + Chris + Didier)

**Question:** What replaces `env.runtimeURL(env.backendBaseURL, env.frontendBaseURL, "app_name", "screen_name", params)`?

**Current status:** Still works but deprecated. protoquery/pull/2218 will formally remove `env.runtimeURL` and deprecate `backendBaseURL` / `frontendBaseURL`.

**Replacement:**

- Use `env.baseURL` instead of `backendBaseURL` / `frontendBaseURL` (always set, works on all surfaces)
- For **in-app navigation**: use the Navigate action node — no URL construction needed
- For **widget tap targets** (native iOS/macOS): use universal link deeplink format (the container app sets `client_frontendBaseURL` via `mixrun/src/widgets.rs`)
- No stdlib replacement planned — surface-specific URL quirks make a generic abstraction more trouble than it's worth (Chris)

**Why no universal stdlib replacement:**
Each surface has different URL semantics:

- Native widgets → universal links into container app (`https://app.remixlabs.com/<app>/<screen>`)
- Desktop widgets → equivalent desktop deep links
- Web "widgets" rendered in screens → same `.remix` exe, use Navigate actions directly

**Gerd's position:** Runtime-specific URL construction should not be in stdlib — belongs in a catalog component or as a client action / env variable template.

**Conclusion (Chris):** "Problem for userspace until we have clearer portable abstractions." Removing `env.runtimeURL` will be a compile-time error, so existing code will surface naturally.

---

## [Feb 22] Desktop: equivalent of `/a/x/apps` actions (Arvind → Chris/Benedikt/Gerd)

**Question:** What is the desktop equivalent of amp's `/a/x/apps` actions (`compact`, `rename` with orig/dest params, `garbage-collect`)?

**Answers (Chris):**

- **`compact` / garbage-collect:** Currently disabled — has issues with file interaction (mix-rs/issues/751). Less necessary now that v2 `.remix` files store binary code in files. Not exposed any
  other way.
- **`rename`:** NOT supported in mixer and probably shouldn't be — DB name is used as identifier in many places; renaming breaks things. Copy+delete workaround exists.
- **Clone (install-as):** Equivalent is `.remix` export + import. Missing piece is "import-as" / "install-as" feature — ability to set a new DB name and Studio display name during import. Arvind added
  this to the Notion task list.

---

## [Feb 23] Agent node: Params → Fields binding toggle needs manual refresh (Arvind → Simon)

**Question:** When toggling from Params bindings to Fields bindings on an agent node, why must you manually hit Refresh? Why not auto-fetch the field bindpoints?

**Answer:** Fixed — turntable/pull/11744

---

## [Feb 23] `parseJSON` fails on multi-dot version strings (Arvind → Gerd/Tyler)

**Question:** Why does `{"version": 0.9880.0}` not parse with `parseJSON`? Should it be treated as a string?

**Answer:** `0.9880.0` is not valid JSON — not a valid number, not quoted. No JSON parser supports it. Must be `{"version": "0.9880.0"}`. Gerd confirmed this will not be papered over. Root cause:
something in the system was returning the version unquoted from an external action — that's the bug to fix.

---

## [Feb 23] Runner link: navigate to a specific screen (Arvind → Didier)

**Question:** For a runner link like `https://remix.app/run?_rmx_url=...`, how do I go straight to a particular screen?

**Answer (Didier):** `/run/<screenname>?_rmx_url=...`

---

## [Feb 24] Deeplink param mismatch: desktop `remix_file_url` vs Flutter `url` (Wilber → Didier/Benedikt)

**Question:** Desktop deeplink uses `?remix_file_url=` but Flutter uses `?url=` for `run-remix-file`. Intentional?

**Answer:** Not intentional — historic discrepancy. Per Notion deeplink doc, correct param is `?url=`.

- Fix: mix-rs/pull/1013 (Benedikt) — align desktop to use `url`
- Notion deeplink doc: notion.so/Mobile-Desktop-deeplinks-27a1d464528f803db94ac980a0bd84eb

---

## [Feb 25] DnD install at L0: appmeta not persisted in `_rmx_admin` (Simon → Benedikt)

**Question:** DnD install at L0 calls `post_install_remix_file` and gets back appmeta, but subsequent GETs to `localhost:2025/v1/ws/local/appmeta/` don't return it. Is it the builder's job to write
it?

**Answer:** No — the endpoint should write it to `_rmx_admin` automatically. Bug confirmed (v2 .remix file). Fix: Fred's fix available in v0.9974.

---

## [Feb 20] Download latest mixer with wasm support (Vijay)

**Answer:** https://github.com/remixlabs/mix-rs/tree/main/mixer#precompiled-binaries, or use `rcm`.

---

## [Feb 19] Leftover file from `export_package` plugin (Arvind → Gerd)

Unknown file appearing in project. **Answer:** Leftover from `export_package` plugin — safe to delete. Will go away once blob URL support is complete.

---

## [Feb 17] Uppercase DB names (Arvind → Gerd)

**Question:** DB names should be lowercase? **Answer (Gerd):** No — uppercase DB names are fine. No lowercase convention required.

---

## [Feb 17] `files` vs `_rmx_files` in Lumber workspace (Arvind → Wilber)

`files` is the old DB name before it was renamed to `_rmx_files`. Found in Lumber workspace because it's an older workspace created before the naming change. Safe to delete `files`. Wilber confirmed
it's not created by current customer tooling.

---

## [Mar 2] File copy API has no overwrite option (Arvind)

**Question:** `/v1/ws/{ws}/app/{app}/file-manager/copy` returns `Exists` error when destination exists. Force/override option?

**Answer:** No overwrite option. Workaround: delete-then-copy. Already captured in standup notes (Mar 3).

---

## [Mar 5] URL issue with Gerd's workspace (Gerd → team)

Gerd hit a URL-related issue. Debugging discussion; details in thread.

---

## [Mar 6] CORS with third-party APIs — proxy through service agent (Wilber → Benedikt + Chris)

**Question:** Client→HubSpot API call fails with CORS on remix.app. Works on amp and desktop. Bug?

**Answer (Benedikt + Chris):** Not a bug. CORS is browser-enforced. Proxying through a service agent works because agent server isn't a browser — request to HubSpot comes from server, not browser JS.
Desktop also bypasses CORS (HTTP client, not browser). **Architecture pattern: proxy third-party API calls through service agents to avoid CORS on remix.app.**

---

## [Mar 6] L1 module copy puts AST in clipboard (Arvind → Didier)

**Question:** Does copying modules at L1 put the L1 code in the clipboard?

**Answer (Didier):** No — it puts the AST of every node into the clipboard. It does end up in the system clipboard but may not render in visual clipboard managers (Raycast) due to size.

---

## [Mar 6] Tools and Settings moved to plugin menu (Sirshendu → Simon)

**Question:** Is "Tools and Settings" gone from the builder?

**Answer (Simon):** Moved to the plugin menu like all other plugins. Still accessible there.

---

## [Mar 9] Lumber workspace: how to find and log in (Simon → Reza)

**Question:** What tool to find the Lumber workspace for Desktop testing?

**Answer:** Customer workspace IDs listed in the "Desktop info" tab of #desktop channel and on the Customer Success Projects Notion page. Reza confirmed: just log in with the Lumber workspace per that
page.

---

## [Mar 12] Mac Claude client unknown UI element (Arvind → Vijay/John)

Arvind shared a screenshot of something unclear inside the Mac Claude client, asking Vijay and John what it means. Vijay didn't know; deferred to John. **Unresolved.**
---

## [Mar 15] "Pack wasm code" option in export package — obsolete (Didier → Gerd)

**Question:** What does the "pack wasm code" option do in `export_package`, and when should/shouldn't it be used?

**Answer (Gerd):** It's obsolete and can go away. When binaries were stored in the DB, wasm code had separate records and the exporter needed to know whether to pack wasm or qcode. Now that binaries
are stored in the filesystem, the distinction is gone. Gerd will check how to remove it.

---

## [Mar 18] Workspace repo pull permissions (Arvind → Gerd)

**Question:** For workspace repos — if you know the workspace name, can any signed-in user list and pull repos, or are special permissions needed?

**Answer (Gerd):** You need **"run" permission** on the following agents in the workspace:
`list_projects`, `get_project`, `list_commits`, `count_commits`, `get_commit`, `list_assets`, `get_asset`, `read_asset`, `read_assets`

---

## [Mar 20] Anchor links (`#<anchor>` URLs) — shipped (Gerd → Tyler)

**Question:** Is there any way to set anchors so you can jump to specific positions in a text via `#<anchor>` URLs? (Needed for the stdlib viewer.)

**Answer (Tyler):** Not supported before, but Tyler merged turntable#11834 (Mar 21) — adds an `id` property to any DOM node in the card editor. Builder no longer auto-generates `id` for card internal
actions/webcomps; uses `data-rmx-cid` instead. No uniqueness enforcement (user-space). Arvind follow-up: needs **smooth scroll** support — if not optional, make it the runtime default.

---

## [Mar 20] Projecting through ref field in query node (Wilber → Simon)

**Question:** Is projecting through a ref field in the query node no longer supported? (e.g. accessing `from.entity` when `from` is a ref.)

**Answer (Simon):** Still works — just type `assigned.name` (or the equivalent field path) directly. Simon demonstrated with a screenshot.

## Mar 24–Apr 2 Additions

**L1 screenshots in repo push/pull [Mar 24, Arvind → Gerd]:** Screenshots: not included. Positions: included but not tracked as changes. Decision: don't push screenshots; preserve existing local ones
on pull (don't null them).

**Navigate to screen in rmx-remix webcomp [Mar 24, Arvind → Didier]:** No URL-only mechanism. Workaround: 2 bindings (source + screen-name) or encode both in URL and split with a function tile. Arvind
chose to update data model to store screen-name explicitly.

**Missing agents in table builder clone [Mar 24, Padmanabha → Mark]:** `get_current_data_source` lives in Facet Builder repo. Fix: pull `facet_builder` from Lumber workspace.

**Open Link + anchor scroll [Mar 25–26, Gerd → Simon]:** Open Link errors on anchor-only URLs in Desktop. HTML links ≠ JS scrollTo. Fix: mix-rs/pull/1079 — aligns Desktop to browser. Merged.

**FFI fallback when FFI absent [Mar 26, Chris → Benedikt]:** Use Mix FFI fallback syntax (see Mix Syntax Notion doc). Amp: compile-time check. Mixer/JS VM: runtime error catchable.

**.remix export size regression [Mar 28–30, Arvind → Wilber/Gerd]:** Publish plugin stored temp .remix files in project Files without cleanup → 90-100MB export (was ~800KB). Fix: use
`binary.blobCreate(appstate, bin, content_type)` — in-memory blob, no filesystem write. Wilber needs to update plugin.

**Binding result types from wasm [Mar 28–30, Vijay]:** No clean solution in spreadsheet. Pattern: `result.withDefault` + `state.lastError` (Gerd). Simon's monadic codegen draft:
`remix-dev.remixlabs.com/e/edit/simon/monadic2`. Open.

**base64 → unzip in Mix [Mar 30, Didier]:** `asset_preview > binary.ofBase64 > binary.decompress(deflate-raw) > binary.toString > string.parseEscJSON`. `binary.decompress` recently added by Gerd.

**File upload CORS on agt-dev [Mar 31–Apr 2, Vijay → Chris/Wilber]:** `file.upload` FFI works on dev; presigned URL Cache-Control not yet on beta/prod. Wilber reverted to client action. Fix:
Cache-Control + CORS fix to beta Apr 2 (Chris).

**"Component refresh" event [Apr 1, Wilber → Arvind]:** Renamed from "on load" inside components. Fires on init + when comp params receive new values. Fires immediately in REPL. Key for plug-and-play
assembly components.

**`text-md` not a Tailwind class [Apr 2, Wilber → Arvind]:** Correct class is `text-base`. Arvind: will add `text-md` alias in theme.
