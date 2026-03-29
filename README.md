# gitops

Reusable GitHub Actions workflows for release automation.

## Release Workflow

A reusable workflow that automates semantic versioning and releases using [semantic-release](https://github.com/semantic-release/semantic-release) and [Conventional Commits](https://www.conventionalcommits.org/).

### What it does

1. Parses commit messages since the last tag following Conventional Commits
2. Determines the next [semver](https://semver.org/) version (`fix:` = patch, `feat:` = minor, `BREAKING CHANGE` = major)
3. Updates `CHANGELOG.md`
4. Creates a git tag and GitHub Release

### Usage

Add a workflow to your repo that calls the reusable workflow:

```yaml
# .github/workflows/release.yml
name: Release

on:
  workflow_dispatch:

jobs:
  release:
    uses: <owner>/gitops/.github/workflows/release.yml@main
    permissions:
      contents: write
      issues: write
      pull-requests: write
    secrets: inherit
```

Replace `<owner>` with your GitHub username or organization.

Then click **Run workflow** on the Actions tab when you want to create a release.

### Outputs

The workflow exposes outputs for downstream jobs:

| Output | Description |
|--------|-------------|
| `new_release_version` | The new version (e.g. `1.2.0`) |
| `new_release_published` | `'true'` if a release was created |

Use these to chain additional jobs — for example, updating deployment manifests in an IaC repo:

```yaml
jobs:
  release:
    uses: <owner>/gitops/.github/workflows/release.yml@main
    permissions:
      contents: write
      issues: write
      pull-requests: write
    secrets: inherit

  update-iac:
    needs: release
    if: needs.release.outputs.new_release_published == 'true'
    runs-on: ubuntu-latest
    steps:
      - run: echo "Deploying version ${{ needs.release.outputs.new_release_version }}"
```

### Commit Message Format

| Prefix | Version Bump | Example |
|--------|-------------|---------|
| `fix:` | Patch (1.0.0 → 1.0.1) | `fix: resolve null pointer in auth` |
| `feat:` | Minor (1.0.0 → 1.1.0) | `feat: add user search endpoint` |
| `feat!:` | Major (1.0.0 → 2.0.0) | `feat!: redesign auth API` |
| `BREAKING CHANGE:` (footer) | Major | Any commit with this footer |
| `chore:`, `docs:`, `ci:` | No release | `docs: update README` |

### Requirements

- The target branch must be `main`
- Repository must have `GITHUB_TOKEN` with write access (automatic in GitHub Actions)
- This repo must be **public** (GitHub requires reusable workflows to be in public repos for personal accounts)

## License

[Apache License 2.0](LICENSE)
