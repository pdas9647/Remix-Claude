# Engineering Process — CI, Releases, Deployment, Operations

> Notion: [Engineering process](https://www.notion.so/11f1d464528f80dcaf4ee38f27af640e)
> Subpages: CI, Deployment channels and release process, Tech debt notes, QA Processes, Planning and task management tools, ChrisDB

---

## Continuous Integration (CI)

> Notion: [Continuous Integration (CI)](https://www.notion.so/11f1d464528f80a0b618ca8d6b28dd95)

- **CircleCI** — [dashboard](https://app.circleci.com/pipelines/github/remixlabs)
- Config: `.circleci/config.yaml` per repo
- New repo setup: add config file + set up in [CircleCI projects dashboard](https://app.circleci.com/projects/project-dashboard/github/remixlabs/)
- CI status shows on GitHub PRs; repos can require specific checks to pass before merge

### Secrets Management

- Uses CircleCI [contexts](https://circleci.com/docs/contexts/) for GCP, AWS, Slack credentials
- GCS artifact bucket auth: two service accounts (read-only, read-write) in GCP project `rmx-artifacts`
    - `rcm-reader` context → `RCM_CI_READER_KEY`
    - `rcm-editor` context → `RCM_CI_EDITOR_KEY`
    - Activate: `rcm script authorize-cloud RCM_CI_READER_KEY rmx-artifacts`
- `slack-alerts` context for Slack failure notifications (used by `slack` orb)

### CI Base Docker Images

- Managed in [`ci-images`](https://github.com/remixlabs/ci-images) repo
- Stored in AWS ECR (public repos, no auth needed)
- Built for `arm64` + `amd64` (except Android images: `amd64` only)
- All Debian-based, `circleci` user with passwordless sudo

| Image                    | Contents                                                                            |
|--------------------------|-------------------------------------------------------------------------------------|
| `rmx-ci-base`            | Node v20.8.1, zx, gsutil, common utils (no rcm)                                     |
| `rmx-ci-rcm`             | base + rcm                                                                          |
| `rmx-ci-standard`        | rcm + Go 1.21.6, goyacc, docker CLI, websocat                                       |
| `rmx-ci-ocaml`           | base + rcm + OCaml/opam toolchain, websocat                                         |
| `rmx-ci-rust`            | rcm + Rust/cargo, bindgen-cli, cargo-wasi, wasm32-wasi target, WABT 1.0.33, sccache |
| `rmx-ci-java`            | base + rcm + Adoptium temurin-17-jdk (amd64 only)                                   |
| `rmx-ci-android-base`    | java + Android SDK/NDK/CLI tools (amd64 only)                                       |
| `rmx-ci-android-flutter` | android-base + rcm + gradle 7.6.3, fastlane, Flutter 3.16.8                         |
| `rmx-ci-android-go`      | android-base + rcm + Go 1.21.3, gomobile                                            |
| `rmx-ci-android-rust`    | android-base + rcm + Rust + Android targets (aarch64, armv7, x86_64)                |

---

## Deployment Channels & Release Process

> Notion: [Deployment channels and release process](https://www.notion.so/11f1d464528f806c8735f8c46301bcb0)

### Versioning

- Each component has a **semantic version** (e.g., `0.1.0`) + incrementing **build numbers** (assigned by CI)
- **Platform version**: assembled from all component builds; immutable once assigned
    - Minor bump: compiler ABI or FFI set extended (programs compiled with new version can't necessarily run on older)
    - Major bump: ABI/FFI removal (lack of backward compatibility). Has never happened; avoided as much as possible.

### Three Deployment Channels

| Channel   | Purpose                                              | Deployment                                                               |
|-----------|------------------------------------------------------|--------------------------------------------------------------------------|
| `dev`     | Latest builds, auto-deployed on any component change | `remix-dev.remixlabs.com/e`, `dev.remix.app`, `agt-dev.remixlabs.com`    |
| `beta`    | Next release candidate, manually promoted from dev   | `remix-beta.remixlabs.com/e`, `beta.remix.app`, `agt-beta.remixlabs.com` |
| `release` | Production                                           | `remix.remixlabs.com/e`, `remix.app`, `agt-uk.remixlabs.com` etc.        |

- Customer deployments should be tied to `release`
- App Store/Play Store mobile app corresponds to `release`; TestFlight builds for `dev`/`beta`
- Built/packaged apps stored as `.remix` files in GCS or R2 (depending on use case) with platform version metadata
- Stable internal apps: develop on beta, publish to `remix.remixlabs.com`

### Which Build to Use

- **Users/customers**: release
- **Internal Remix Labs** (building/testing): beta (use release when building production customer assets)
- **Platform developers**: dev

### Core Flows Channels

- Three flow channels: `draft`, `test`, `public`
- Each flow build gets an incrementing version number on publish
- **Core Flows Version**: `versions.json` collecting build numbers for each core flow
- Organized in GCS by platform major/minor version, then draft/test/public status
- A new platform version only needs testing with the latest core flow version
- A new core flow version should be tested against all currently supported platform versions

### Release Cadence — Every Thursday Morning ET

**Step 1: Promote beta → release** (if no known issues):

1. Update `harmony` latest `release` build to current `beta` → updates amp servers via `roadie`
2. Release current beta mobile app builds to App Store / Play Store
3. Update `release` branch in `remix.app` repo (turntable build)
4. Update web components on `remixlabs.com` in `website` repo
5. Update `release` branch of `M` for mixer on `agt-uk`

**Step 2: Promote dev → beta** (if no known issues):

1. Update `harmony` latest `beta` build to current `dev` → updates amp servers via `roadie`
2. Release TestFlight build to `beta` group; submit to stores
3. Update `beta` branch in `remix.app` repo
4. Update `beta` branch of `M` for mixer on `agt-beta`

- Coordination via `harmony` PR (use `&template=promotion.md` query param)
- Fixes deployed by Monday are eligible for Thursday promotion; otherwise wait

### Hot Fixes

- Goal: smallest out-of-band change necessary to avoid substantial service degradation
- Prefer highest-level fix (application > platform)
- For urgent issues: hot-fix branch off current release build's branch for the affected component
- Test on beta environment before deploying to production if possible

### Auto-Update Chain (dev builds)

Merge to `main` triggers `update-users` CI job, cascading:

```
groovebox → mix-rs (mixer, mixcore, mixrun) → [protoquery, M] → amp → turntable → [remix.app, flutter-runtime]
```

Deploying repos: `harmony` (amp), `remix.app` (web runtime), `flutter-runtime` (iOS), `M` (mixer)

---

## QA Processes

> Notion: [QA Processes](https://www.notion.so/1421d464528f802a8675d4c52258257d)

- Two primary QA focuses: mobile apps and web+cloud-agent apps (e.g., restaurant use case)
- Mobile app / app clip testing: test 3 app clips for the build, test basic features (sharesheet, widgets), submit if good
- Restaurant tests documented separately

### iOS App Store Submission Steps

1. Open App Store Connect → select "Build in TF list"
2. Add app clip to list, test app clips from TestFlight list
3. Install main app, check draft and public versions
4. Create new release (+), same name/build number as previous
5. Check promotional text, add release notes
6. Click "Apply on app clip", add build number, save
7. Submit for review

### QA Vision (target state)

- Maintain a small number of standard test apps in a known location
- For each new platform version: run all test apps through codegen+build pipeline (check if codegen changed)
- Built apps installed, run, and tested across surfaces
- Service agents: deployed and tested automatically
- Web/mobile flows: tested with human QA scripts
- Tracking: `remix-issues#141`

### Strategic Context (as of Feb 2026)

- Platform is "close to feature complete"; content is "not at all close"
- 2026 is the "year of GTM" — 5 customers now, headed to 20
- Platform needs bug fixes and incremental improvements, not new releases
- Moving off amp builder toward Desktop Studio is a priority
- Service-agent-only apps should be built on desktop, in the dev channel
- Missing: "sharable core app" (centralized libraries + build recipes on agent servers, not laptops)

---

## Operational Runbooks ("ChrisDB")

> Notion: [ChrisDB](https://www.notion.so/2f01d464528f807da58ecafbde85d3c4)
> "A (hopefully shrinking) list of things only Chris knows how to do"

### Weekly Promotion Commands

1. Create harmony PR: `bin/create-promotion-pr [--include-dev]` (flag = also promote dev → release)
2. Update `remix.app` and `M`: `bin/do-promotions` (outputs instructions for beta update in `flutter-runtime`)
3. Update nginx in roadie: `bin/update-nginx-in-roadie TAG`

### Agent Server Disk Usage

- Configure kubectl: `source kube-config.sh prod|poc|beta|dev` (in `M` repo)
- Check disk: `kubectl exec <pod> -- df -h` (use `kyolo` helper for single-pod clusters)

### AWS Accounts

| Account             | Number         | Purpose                                                               |
|---------------------|----------------|-----------------------------------------------------------------------|
| `figly` (top-level) | `250233190882` | Organization root (admin: John). Has old ECR private repo `remixlabs` |
| `remix-internal`    | `544862508450` | Public CI images in ECR                                               |
| `remix-dev`         | `066861643289` | Dev account                                                           |

### Amp Rollbacks

- **Full disk rollback** from snapshot:
    1. Create disk from snapshot
    2. Edit deployment to mount at a different directory
    3. Copy raw DB files (db name → directory name via hash; use `bin/encode-app-name` in amp)
    4. Can swap out the `v2` directory if no one is using the app
- **Screen-level rollback** (for node corruption / builder cleanup issues):
  `https://remix.remixlabs.com/e/edit/rollback/rollback`

---

## Planning & Task Management

> Notion: [Planning and task management tools](https://www.notion.so/2e81d464528f80748dd4ee31f348f1f5)

### Current State (as of Feb 2026)

- **Sprint cadence**: 2-week sprints. Planning meeting first Monday before standup; wrap-up/retro the following Friday before standup.
- **PM roles**: Chris = engineering PM (feature requests flow through GitHub board); Reza = content PM
- Bug reports start as Slack threads (`#bugbash` or component channels like `#flutter`), goal is to create GitHub issues for visibility/tracking
- Daily standups; first part is prioritizing new/unprioritized tasks
- Diagrams: standardized on draw.io (connected to Google Drive)

### GitHub Projects

- Main board: `github.com/orgs/remixlabs/projects/2/views/1`
- Tracking issues view: `github.com/orgs/remixlabs/projects/2/views/6`
- Content/Flows/Feedback: `github.com/orgs/remixlabs/projects/3`
- How-to: [How to use the Github Projects board](https://www.notion.so/3091d464528f805bbdb3e7d54898c4a7)

#### Board Columns (main board)

| Column           | Meaning                                                        |
|------------------|----------------------------------------------------------------|
| Icebox           | Backlog bankruptcy; low-priority, pull items out as needed     |
| Backlog          | Old issues, candidates for future sprints                      |
| New              | Auto-added issues/PRs; sort during next sprint planning        |
| Ready to pick up | Unassigned tasks ready to start                                |
| In progress      | Actively worked on (must have assignee)                        |
| Ready to review  | Completed work awaiting reviewer pickup (generally unassigned) |
| In review        | Actively being reviewed (must have reviewer)                   |
| Parked           | Partially done but blocked or de-prioritized                   |

- "Ready to pick up" + all in-progress/review (not parked) = current sprint scope
- Filter by assignee to see "your stuff"

### Standup Rotation

- 1 GTM day, 2 content days, 2 platform days per week
- Platform days may split as "desktop" and "server/infra"

### Day-to-Day Dev Conventions

- PRs should be linked to a GitHub issue if work takes more than a day
- Issues/PRs should get appropriate labels and milestones
- Feature requests should flow through Chris (eng PM), not directly to individual engineers

### Product/Project Breakdown

- **Desktop and friends**: desktop app, Chrome extension, builder and web runtimes
- **Server/infra**: self-host mixer, Snowflake mixer, core components (groovebox/mixquery/protoquery), build server/pipeline, customer ops
- **Remix ops**: devops, security, QA, planning, documentation

### Active Milestones

- Desktop app v1
- Chrome extension v1
- Self-host server v1
- Snowflake-hosted server v1

---

## Asset Submission & Review (Tasks Tool)

> Notion: [Tasks - Asset Submission Guide](https://www.notion.so/2111d464528f8031a05fd76e687b6379)

**Purpose**: Submit artifacts (flows, builder assets, MCP tools) for review and potential publishing to Remix catalog or library.

### Setup

1. Install the **Tasks** app via **Workspace Tools → App Hub**
2. Access Tasks tool:
    - Prod US: `remix.remixlabs.com/tasks/start`
    - Prod India: `remix-india.remixlabs.com/tasks/start`
3. Configure to point to your workspace via **Set workspace**

### Supported Artifact Types

| Type                   | Description                                    |
|------------------------|------------------------------------------------|
| **Auth Configuration** | JSON payload of an auth configuration          |
| **MCP Tools**          | .remix file with agents accessible through MCP |
| **Builder Asset**      | Builder node (screen, agent, component, etc.)  |

### Workflow

1. **Create** — New task with name + description (starts as "backlog")
2. **Attach** — Add artifacts via corresponding buttons
3. **Complete** — Mark task as complete
4. **Submit** — "Submit for review" with reviewer-facing name/description

### Key Rules

- All submissions reviewed in the **`remix_review`** workspace
- MCP .remix files must include MCP records with `_rmx_type = mcp_tool`
- Don't submit the same artifact multiple times unless necessary

---

## GTM Customer Categories

> Notion: [T1/T2 Customers](https://www.notion.so/20d1d464528f80608571fc691f0bf69f)

Four reusable solution patterns for customer engagements:

1. **Informational Awareness** — Ad hoc queries, NatLang, widgets; info consumption at point of use
2. **Human in the Loop / Distributed Data Gathering** — Beacons + Chrome extension → data back to Snowflake; rapid workflow deployment across surfaces
3. **Agentic Use Cases** — Generalized MCP; IT-focused, potential end-user config; inline prompting in L2/template tools
4. **Digital Experience at Scale** — CDP+ETL = DBP; educational sale; self-serve quiz/template creation and deployment

### GTM Prospect Pipeline (as of mid-2025)

| Prospect              | Priority | Categories              | Notes                               |
|-----------------------|----------|-------------------------|-------------------------------------|
| Funda                 | High     | —                       | Live customer, LinkedIn+AI+DBP      |
| Guidewire             | High     | —                       | Customer 360, contract audit        |
| Alteryx               | High     | —                       | "We are solution; push towards LLM" |
| Opt Health            | High     | Info Awareness, Agentic | Requested proposal                  |
| Snowflake             | Medium   | —                       | Partnership/integration target      |
| Freshworks            | —        | —                       | —                                   |
| Cisco                 | Low      | —                       | —                                   |
| Next Frontier Capital | Low      | —                       | —                                   |

_Note: Anthropic listed as "not a customer" (reference/partner context)._
