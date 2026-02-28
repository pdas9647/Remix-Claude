# Engineering — Continuous Integration (CI)

> Source: [Continuous Integration (CI)](https://www.notion.so/11f1d464528f80a0b618ca8d6b28dd95)
> Parent: Engineering process → Platform Engineering (Internal) Home
> Last updated: 2026-02-24

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

Custom images in [`ci-images`](https://github.com/remixlabs/ci-images) repo, built by CI on `main`. Stored in AWS ECR (public). Built for both `arm64` and `amd64`. Debian-based, `circleci` non-root user with passwordless `sudo`.

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