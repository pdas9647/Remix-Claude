# #dumbquestionsanswered Slack Channel — Remix Labs

**Coverage:** Feb 16–22, 2026
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

No answer yet — open question.

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