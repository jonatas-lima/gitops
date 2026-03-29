# CLAUDE.md — gitops

## Overview

Reusable GitHub Actions workflows for release automation. Currently contains a single release workflow powered by semantic-release.

## Structure

```
.github/workflows/
  release.yml       # Reusable workflow (workflow_call) — runs semantic-release
```

## How It Works

Consumer repos call the reusable workflow via `workflow_dispatch`:

```yaml
jobs:
  release:
    uses: <owner>/gitops/.github/workflows/release.yml@main
    permissions:
      contents: write
      issues: write
      pull-requests: write
    secrets: inherit
```

The workflow parses Conventional Commits, bumps semver, updates CHANGELOG.md, creates a git tag and GitHub Release. It exposes `new_release_version` and `new_release_published` as outputs.

IaC update logic (e.g. updating image tags in deployment manifests) should live in the **caller workflow**, not here. This keeps the reusable workflow generic and allows per-repo customization.

## Commit Conventions

This repo and all consumer repos follow [Conventional Commits](https://www.conventionalcommits.org/):

- `fix:` = patch bump
- `feat:` = minor bump
- `feat!:` or `BREAKING CHANGE:` footer = major bump
- `chore:`, `docs:`, `ci:` = no release

## Gotchas

- The `.releaserc.json` config is created inline in the workflow, not committed. Changes to semantic-release config happen in `release.yml`, not a separate file.
- The branch must be `main` — semantic-release is configured for `["main"]` only.
- `persist-credentials: false` in checkout is required — semantic-release authenticates via `GITHUB_TOKEN` env var, not git credentials.
- The repo must be **public** for other repos to use the reusable workflow (GitHub limitation for personal accounts; orgs can share private reusable workflows).
