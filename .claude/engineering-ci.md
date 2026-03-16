# Engineering — CI, rcm & Operations

>
Sources: [CI](https://www.notion.so/11f1d464528f80a0b618ca8d6b28dd95) | [rcm](https://www.notion.so/11f1d464528f80d28321f9c83e3a0cbc) | [ChrisDB](https://www.notion.so/2f01d464528f807da58ecafbde85d3c4) | [Bug reporting](https://www.notion.so/3061d464528f80cdacf7eed2612bad07)
> Parent: Engineering process → Platform Engineering (Internal) Home
> Last updated: 2026-03-16

---

## Overview

CircleCI ([dashboard](https://app.circleci.com/pipelines/github/remixlabs)) runs CI for nearly all GitHub repos. Functions:

1. Run automated tests before merging to main
2. Deploy internal build artifacts on commits
3. Run deployment jobs on merges to `main` or special branches

Config file: `.circleci/config.yaml` per repo ([CircleCI docs](https://circleci.com/docs/configuration-reference/)). CI status shown on GitHub PRs; repos can require checks to pass before merge.

To add CI to a new repo: add `.circleci/config.yaml` **and** register the repo in the CircleCI [projects dashboard](https://app.circleci.com/projects/project-dashboard/github/remixlabs/).

## Secrets Management

Secrets managed via CircleCI [contexts](https://circleci.com/docs/contexts/). Key contexts:

| Context        | Env vars            | Purpose                                  |
|----------------|---------------------|------------------------------------------|
| `rcm-reader`   | `RCM_CI_READER_KEY` | Read-only access to GCS artifact bucket  |
| `rcm-editor`   | `RCM_CI_EDITOR_KEY` | Read+write access to GCS artifact bucket |
| `slack-alerts` | (Slack creds)       | Failure notifications via `slack` orb    |

GCP project for both service accounts: `rmx-artifacts`.

Activate account in job: `rcm script authorize-cloud RCM_CI_READER_KEY rmx-artifacts`

Slack failure notification (drop-in):

```yaml
steps:
  - slack/notify:
      event: fail
      branch_pattern: main
```

## Docker Base Images

Custom images in [`ci-images`](https://github.com/remixlabs/ci-images) repo, built by CI on `main`. Stored in AWS ECR (public). Built for both `arm64` and `amd64`. Debian-based, `circleci` non-root
user with passwordless `sudo`.

| Image                           | Contents                                                                                                              |
|---------------------------------|-----------------------------------------------------------------------------------------------------------------------|
| `rmx-ci-base-{amd64,arm64}`     | node v20.8.1, zx, standard CLI tools, gsutil                                                                          |
| `rmx-ci-rcm-{amd64,arm64}`      | base + `rcm`                                                                                                          |
| `rmx-ci-standard-{amd64,arm64}` | rcm + Go 1.21.6 + goyacc + gcc + docker CLI + websocat                                                                |
| `rmx-ci-ocaml-{amd64,arm64}`    | base + rcm + gcc/cmake/etc + ocamlc/opam (via [ocaml-scripts](https://github.com/remixlabs/ocaml-scripts)) + websocat |
| `rmx-ci-rust-{amd64,arm64}`     | rcm + rustc/cargo + bindgen-cli + cargo-wasi + wasm32-wasi + WABT 1.0.33 + sccache 0.5.4                              |
| `rmx-ci-java-amd64`             | base + rcm + Adoptium temurin-17-jdk (amd64 only; JDK install is flaky in Docker)                                     |
| `rmx-ci-android-base-amd64`     | java + Android SDK/NDK/command-line tools                                                                             |
| `rmx-ci-android-flutter-amd64`  | android-base + rcm + g++ + gradle 7.6.3 + fastlane + flutter 3.16.8                                                   |
| `rmx-ci-android-go-amd64`       | android-base + rcm + Go 1.21.3 + gomobile                                                                             |
| `rmx-ci-android-rust-amd64`     | android-base + rcm + rustc/cargo + WABT + sccache + Android Rust targets                                              |

**Android targets** in `rmx-ci-android-rust-amd64`: `aarch64-linux-android`, `armv7-linux-androideabi`, `x86_64-linux-android`. Android images are `amd64`-only (Android SDK not available for arm64).

---

## rcm — Remix Component Manager

> Source: [rcm](https://www.notion.so/11f1d464528f80d28321f9c83e3a0cbc)
> Repo: [github.com/remixlabs/rcm](https://github.com/remixlabs/rcm)

npm-like tool for installing, updating, and publishing versioned dependencies within the Remix platform.

### Component Model

- **Product** — shippable unit (mobile app, Docker image)
- **Artifact** — internal build product (static lib, Wasm file)
- **Component** — piece of the system with its own repo + dependency node
- **Build** — unique immutable label for component artifacts (incrementing per component)
- **Version** — semver label mapping to one or more builds

Artifacts stored in GCS: `gs://remix-dev/artifacts/{component}/{build}/{bin|lib|assets}/...`
Refs: `gs://remix-dev/artifacts/{component}/refs/{branch|semver|sha}` → build number.

### Config File (`rcm.config.mjs`)

```javascript
export default {
    "name": "harmony",
    "version": "0.1.0",
    "dependencies": {
        "mixrun": {"kind": "branch", "branch": "main"}
    }
}
```

**Dependency kinds:** `build` (immutable tag), `version` (semver, supports `^` caret), `branch`, `sha`, `local` (with optional `fallback`). All except `local` accept `"repo"` field for cargo builds.

**Lock file:** `rcm-lock.json` pins mutable versions. `rcm install` uses lock; `rcm update` fetches latest.

### Key Commands

| Command                       | Description                                   |
|-------------------------------|-----------------------------------------------|
| `rcm install`                 | Install all dependencies (respects lock file) |
| `rcm update`                  | Install all, updating to latest               |
| `rcm comp publish BUILD`      | Upload artifacts                              |
| `rcm comp publish-local`      | Install locally                               |
| `rcm dep install/update NAME` | Single dependency                             |
| `rcm dep latest-build NAME`   | Check latest build                            |
| `rcm script NAME [args]`      | Run `scripts/NAME.mjs`                        |

### Auto-Update Pipeline

On merge to `main`, CI runs `update-users.mjs`: checks downstream components, and if the update is compatible (same version or within caret range), automatically updates `rcm-lock.json` and pushes to
`main` in the downstream repo. Cascades transitively. Prevent by bumping the version to an incompatible semver.

### Adding a New Component

1. New GitHub repo with access (`admins`=admin, `figly`=write)
2. Add `rcm.config.mjs`
3. Implement `make` targets: `setup`, `artifacts`, `post-update-deps`, `integration-tests-{dep}`
4. Add `.circleci/config.yml` with `update-users` job + CircleCI contexts (`rcm-reader`, `rcm-editor`)
5. Update dependency graph in `rcm/scripts/users-of-component.mjs`

---

## GitHub Projects Board

> Source: [How to use the Github Projects board](https://www.notion.so/3091d464528f805bbdb3e7d54898c4a7)
> Parent: Platform Engineering (Internal) Home > Planning docs

### Columns

| Column               | Purpose                                                            |
|----------------------|--------------------------------------------------------------------|
| **Icebox**           | Backlog++; triaging not current priority; pull items out as needed |
| **Backlog**          | Old issues, candidates for future sprints                          |
| **New**              | Auto-added issues/PRs; sorted during next sprint planning          |
| **Ready to pick up** | Unassigned tasks ready to be picked up                             |
| **In progress**      | Actively being worked on; **must have an assignee**                |
| **Ready to review**  | Completed work ready for review pickup; generally **unassigned**   |
| **In review**        | Actively being reviewed; **must have a reviewer**                  |
| **Parked**           | Partially completed; blocked or de-prioritized                     |

### Workflow Rules

- Everything in "In Progress" **must** have an assignee; everything in "In Review" **must** have a reviewer
- "Ready to pick up" and "Ready to review" should generally be **unassigned**
- If assigned to you in "In progress": figure out next step (could be moving to backlog, parked, or re-assigning)
- Filter the board to "your stuff" by clicking your avatar on a tile or using the search bar filter
- "Ready to pick up" + everything in progress/review (not parked) = the current sprint grouping

---

## Operations (ChrisDB)

> Source: [ChrisDB](https://www.notion.so/2f01d464528f807da58ecafbde85d3c4) — last updated 2026-03-11
> "A hopefully shrinking list of things only Chris knows how to do"

### Weekly Promotion

1. Create harmony PR: `bin/create-promotion-pr [--include-dev]` — merging updates `roadie` and deployed amp environments
2. Update `remix.app` and `M`: `bin/do-promotions` — also outputs beta update instructions for `flutter-runtime`
3. Update nginx in roadie: `bin/update-nginx-in-roadie TAG`

### AWS Accounts

| Account             | Number       | Purpose                                                         |
|---------------------|--------------|-----------------------------------------------------------------|
| `figly` (top-level) | 250233190882 | Organization (John=admin). ECR `remixlabs` (old) + `mixer` repo |
| `remix-internal`    | 544862508450 | CI images (public ECR)                                          |
| `remix-dev`         | 066861643289 | Root: chris@remixlabs.com                                       |

### Agent Server Disk Usage

With GCP cluster access: `source kube-config.sh prod|poc|beta|dev` in `M`, then `kubectl exec` + `df -h`.

### Mixer DB Admin / Rollback

Mixer uses nightly snapshots (same approach as Amp). Server runs as user `remixlabs` in group `www-data` — `chgrp`/`chown` needed if SSH as root.

- **Index directories:** `<db_name>/v2/` — only current version needed, but set must match `vers.json`. Dirs not in `vers.json` are harmless and removable.
- **Interrupted flush recovery:** orphaned write-log records (index doesn't reference them, numbering off). Fix: use latest version in `vers.json` to get expected record count → truncate `db.off` and
  `db.dat` to that count → clean `vers.json` → restart server. See [ChrisDB](https://www.notion.so/2f01d464528f807da58ecafbde85d3c4) for the full recovery script.

### Amp Rollbacks

- **Full disk rollback:** Create disk from snapshot → mount on target pod → copy raw DB files (hash-named dirs, use `bin/encode-app-name` in amp) → swap `v2` directory
- **Screen node rollback:** `https://remix.remixlabs.com/e/edit/rollback/rollback` — rolls back individual screen nodes to earlier time (for node corruption)

---

## Bug Reporting Process

> Source: [Bug reporting](https://www.notion.so/3061d464528f80cdacf7eed2612bad07) — last updated 2026-03-10

1. **Ask:** Is it a bug? Check Slack / GitHub issues, or ask in `#bugbash` / `#dumbquestionsanswered`
2. **Repro:** Provide minimal repro — link to cloud builder screen, .remix file, or step-by-step. Include: product/surface, version, OS/browser
3. **Fix or file:** Engineering either fixes immediately (link PR in thread) or creates a GitHub issue. Reporter ensures one happens
4. **Mark resolution** (emoji on original Slack post): ✅ fixed, 👍 expected, 📝 ticket created, 🚧 content bug, 🤷 no repro
