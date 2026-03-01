# #dumbquestionsanswered Slack Channel — Remix Labs

**Coverage:** Feb 16–28, 2026
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
- **`compact` / garbage-collect:** Currently disabled — has issues with file interaction (mix-rs/issues/751). Less necessary now that v2 `.remix` files store binary code in files. Not exposed any other way.
- **`rename`:** NOT supported in mixer and probably shouldn't be — DB name is used as identifier in many places; renaming breaks things. Copy+delete workaround exists.
- **Clone (install-as):** Equivalent is `.remix` export + import. Missing piece is "import-as" / "install-as" feature — ability to set a new DB name and Studio display name during import. Arvind added this to the Notion task list.

---

## [Feb 23] Agent node: Params → Fields binding toggle needs manual refresh (Arvind → Simon)

**Question:** When toggling from Params bindings to Fields bindings on an agent node, why must you manually hit Refresh? Why not auto-fetch the field bindpoints?

**Answer:** Fixed — turntable/pull/11744

---

## [Feb 23] `parseJSON` fails on multi-dot version strings (Arvind → Gerd/Tyler)

**Question:** Why does `{"version": 0.9880.0}` not parse with `parseJSON`? Should it be treated as a string?

**Answer:** `0.9880.0` is not valid JSON — not a valid number, not quoted. No JSON parser supports it. Must be `{"version": "0.9880.0"}`. Gerd confirmed this will not be papered over. Root cause: something in the system was returning the version unquoted from an external action — that's the bug to fix.

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

**Question:** DnD install at L0 calls `post_install_remix_file` and gets back appmeta, but subsequent GETs to `localhost:2025/v1/ws/local/appmeta/` don't return it. Is it the builder's job to write it?

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