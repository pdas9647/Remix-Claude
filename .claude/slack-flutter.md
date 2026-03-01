# #flutter Slack Channel — Remix Labs

**Coverage:** Jan 1 – Feb 28, 2026
**Channel ID:** CPNGC1UQ4
**Purpose:** All things mobile app (iOS + Android)

---

## Release Log & Key Changes

### [Jan 6] TF 2475 — Widgets restored on iOS (Didier)

- Widgets working again on iOS using **federated query** to discover all existing widgets
- No longer relies on amp to get list of DBs
- No longer requires packaging records from other DBs
- Uses platform **1.45** (prod was still on 1.44 at the time)

### [Jan 5] DB list discovery — native filesystem approach (Didier)

- Old approach (`listWidgets` with dbname) replaced: now traverses filesystem to get DB names from `vers.json` files
- iOS implemented (TF 2475); Kotlin (Android) version to follow
- `listWidgets` in mixrun → replaced by `listDbs` approach; agent queries each DB for widgets per DB
- Benedikt exploring more general mixer FFI exposure — lower priority now that mobile requirement is met

### [Jan 10] TF 2492→2496 — Both iOS + Android (Didier)

Two major changes:

1. **Platform version 1.45** — apps must be published from machines updated to 1.45 (or use new explicit version field in Arvind's packaging plugin)
2. **Federated search** for flows and widgets now on both iOS and Android

**Breaking packaging change:** New tools & settings plugin uses **1 db == 1 .remix** format. `.remix` files packaged with the new tool will not import correctly in older app versions (flows and
widgets won't appear).

Auth server + workspace env vars were mixed up in 2492 → fixed in **2496** (use 2496+).

### [Jan 20] `/run?__rmx__url=` deeplink support (Didier)

Flutter app now properly handles `open /run?__rmx__url=<url of .remix>` — downloads and runs a .remix from URL.

### [Feb 6] TF 2554 — Widgets + config params on iOS + Android (Didier)

- Widgets with config params confirmed working on both platforms
- Snowflake dynamic widget use case verified by John (fresh install)
- Use any version >= 2554

**Pending store submission:** Blocked by bug — flows with a widget record don't appear in the app (only show under "Manage Content"). Flows-only .remix files work fine. Repro confirmed by Arka. Root
cause: suspected userspace issue with agent info setup or packaging. Arvind assigned to investigate.

### [Feb 20] TF 2601 — Deeplink `/run-remix-file` fix (Didier)

- Wilber found: `/run-remix-file` deeplink (auto-download + run a .remix) was broken in latest mobile app
- Fixed in 2601

---

## Channel Tone & Patterns

- Didier is sole author of all release announcements; low-traffic
- Testing delegated to Mukund, Arka, John; Chris handles TestFlight pushes
- Bugs cross-referenced to #bugbash threads