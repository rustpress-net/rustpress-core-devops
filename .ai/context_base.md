# rustpress-core-devops — AI Context

> **Purpose**: Orient an AI agent to this repo without reading the whole tree. Pair with the RustPress organisation context in `rustpress-core-base/.ai/context/CONTEXT_BASE.md`.

## Project

`rustpress-core-devops` is the **centralised GitHub Actions repository** for the `rustpress-net` organisation. It holds nine reusable **composite actions** that every other RustPress repo wires into its `.github/workflows/ci.yml`. This is the only repo in the org that ships no runtime code — it ships YAML.

Other repos call into it like this:
```yaml
- uses: rustpress-net/rustpress-core-devops/actions/ci-rust@main
```

If a build pipeline behaves oddly anywhere in the org, the fix usually lives here.

## Tech stack

- **GitHub Actions composite actions** (YAML, bash steps)
- Targets: Linux/macOS/Windows runners, Rust 1.75+, Node 18/20, Docker Buildx
- No external build system — composite actions execute the steps inline

## Directory layout

```
rustpress-core-devops/
├── actions/
│   ├── ci-rust/           # cargo check + test + clippy + fmt
│   ├── ci-node/           # npm ci, lint, test, build
│   ├── build-rust/        # cross-target binary builds
│   ├── build-node/        # tsup/vite/webpack node builds
│   ├── build-theme/       # package a RustPress theme tarball
│   ├── version-bump/      # conventional-commits → semver decision
│   ├── create-release/    # gh release create with artifacts
│   ├── cleanup-releases/  # purge prereleases/drafts
│   └── docker-build/      # buildx + push to ghcr.io
├── README.md              # public usage docs (246 lines, accurate)
├── LICENSE-MIT
└── LICENSE-APACHE
```

Each action directory contains a single `action.yml`. There are **no shared bash scripts** — duplication across actions is intentional to keep each one self-contained and forkable.

## Public API / what this repo exposes

Nine composite actions consumable by `uses: rustpress-net/rustpress-core-devops/actions/<name>@main` (or `@v1` once we cut a stable tag). The inputs/outputs are declared in each `action.yml`. The README has copy-pastable usage examples for `ci-rust` and `ci-node`.

Action authors should treat the input/output schema as a **public API**: changing it is a breaking change for every downstream repo.

## How to build / test

This repo has nothing to build. The intended local-test workflow is:

```bash
# Lint all action manifests
gh actionlint

# Run an action in a real CI run by referencing the branch from any
# downstream repo's workflow:
#   uses: rustpress-net/rustpress-core-devops/actions/ci-rust@my-feature-branch
```

There is **no integration test workflow yet** — see Known Issues.

## Cross-repo dependencies

- **Depends on**: nothing — leaf node of the dependency graph
- **Depended on by**: all 15 other public RustPress repos (every CI workflow references at least `ci-rust` or `ci-node`)
- Because it's transitive on everything, **breaking changes here ripple across the org**. Tag stable versions (`v1`, `v1.1`, etc.) and pin downstream consumers to a tag rather than `@main` when going GA.

## Conventions

- **License**: `MIT OR Apache-2.0` (both LICENSE files at repo root)
- **Commits**: Conventional Commits (`feat:`, `fix:`, `chore:`, `docs:`)
- **Branching**: `main` is live; feature branches off `main`; PR-merge
- **Action versioning**: float on `@main` during alpha; cut `@v1` once downstream repos pin

## Status

- Release readiness: **🟡 ALMOST READY** for v1.0 (see `AUDIT.md`)
- LICENSE files: ✅ added (recent commit `065b898 chore: add LICENSE files`)
- Phase: alpha hardening — feeds Phase 1 of the v1.0 plan

## Known issues / TODOs

From the v1.0 audit (`AUDIT-core.md` section 3):

- **P0**: No integration test workflow (`test-actions.yml`) — actions are not exercised together; a breaking change in `build-rust` may not be caught until a downstream repo's CI explodes.
- **P1**: Sparse input/output docs in each `action.yml` — every action should have a `description:` and an examples block in the README.
- **P1**: Cut a `v1` release tag and start pinning downstream consumers; today every workflow uses `@main`.
- **P1**: Document the action versioning strategy and the deprecation policy in the README.

## When working in this repo

- Treat each `action.yml` as a public API. Any input rename / removal must update every downstream repo's workflow in the same PR.
- Never `cd` inside an action's `runs:` steps without preserving `${{ github.workspace }}` — composite actions can be called from arbitrary repos.
- Bash steps must work on `ubuntu-latest`, `macos-latest`, AND `windows-latest` (PowerShell) unless the action is explicitly Linux-only.
- Avoid adding new third-party Marketplace actions without org review — prefer scripting against `gh` / `cargo` / `npm` directly.
