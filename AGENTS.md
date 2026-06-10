# Agent Instructions — calavia-org/release-github

## Project Overview

GitHub composite action for creating releases with semantic versioning and automatic maintenance branches. Published as `calavia-org/release-github`.

## Repository Layout

```
.
├── action.yml              # Main composite action definition
├── .pre-commit-config.yaml # Linting hooks
├── README.md               # Usage documentation
└── LICENSE                 # MIT
```

## What This Action Does

1. Creates annotated git tag using `mathieudutour/github-tag-action@v6.2`
2. Creates GitHub Release using `softprops/action-gh-release@v3`
3. On major releases (x.0.0), creates `release/X.x` branch from previous major tag

## Dependencies

- `mathieudutour/github-tag-action@v6.2` — semantic version tagging
- `softprops/action-gh-release@v3` — GitHub Release creation

## Linting

Pre-commit must be installed in a virtual environment to work. Create and activate one:

```bash
python3 -m venv .venv
source .venv/bin/activate
pip install pre-commit
pre-commit install
pre-commit run --all-files
```

Pre-commit hooks:
- `actionlint` — GitHub Actions linting
- `check-yaml`, `check-merge-conflict`, `trailing-whitespace`, `end-of-file-fixer`
- `no-commit-to-branch` — protects `main`, `master`, `release/*`
- `conventional-pre-commit` — enforces conventional commits on commit-msg
- `markdownlint` — markdown linting

## Branch Protection

Direct commits to `main`, `master`, and `release/*` are protected branches. PRs are required for all changes to these branches. Use feature branches:
- `feat/` — new features (triggers minor version bump)
- `fix/` — bug fixes (triggers patch version bump)
- `chore/` — maintenance (no version bump)
- `docs/` — documentation changes (skip release)
- `refactor/` — code restructuring (skip release)
- `test/` — test additions/changes (skip release)
- `style/` — formatting changes (skip release)
- `perf/` — performance improvements (skip release)
- `ci/` — CI/CD changes (skip release)

## PR Conventions

All repositories using this action are configured with:
- **Linear history** (no merge commits)
- **Squash commits** (PR titles become commit messages)
- **PR title as commit message** (the squash commit uses the PR title)

**Always use conventional commit format in PR titles.** The PR title will become the commit message after squash, and the release action will parse it to determine version bumps.

Since `main`, `master`, and `release/*` branches are protected, all changes must go through PRs. This ensures:
- Code review before merging
- Consistent commit history using conventional commits
- Proper version bumping based on PR titles
- Pre-commit checks pass before merging

### PR Title Format

```
type(scope): description
```

Examples:
- `feat(auth): add OAuth2 login support`
- `fix(api): handle null response from endpoint`
- `docs(readme): update installation instructions`
- `chore(deps): bump dependencies`
- `refactor(utils): simplify validation logic`
- `test(auth): add integration tests for login`
- `style(css): fix indentation in components`
- `perf(query): optimize database queries`
- `ci(github): add codeql workflow`

### Branch Naming

Use conventional prefixes in branch names to clarify intent:
- `feat/` — new features (triggers minor version bump)
- `fix/` — bug fixes (triggers patch version bump)
- `chore/` — maintenance (no version bump)
- `docs/` — documentation changes (skip release)
- `refactor/` — code restructuring (skip release)
- `test/` — test additions/changes (skip release)
- `style/` — formatting changes (skip release)
- `perf/` — performance improvements (skip release)
- `ci/` — CI/CD changes (skip release)

### Why This Matters

When you create a PR:
1. GitHub squash-merges the PR using the PR title as the commit message
2. The release action reads this commit message on the next release
3. The commit type determines version bumps and whether to skip the release

If your PR title doesn't follow conventional commit format, the action will auto-detect the version bump from the merge commit, which may not match your intent.

## Versioning

- **Source of truth**: Git tags (`v1.0.0`, `v1.1.0`, etc.)
- **Floating tags**: `v1`, `v2` — point to latest in major line
- **Bump rules**: Conventional commits determine bump type (feat=minor, fix=patch, BREAKING=major)

## Skipping Releases

You can configure the action to skip releases for specific conventional commit types. This is useful for documentation changes, chores, or other commits that shouldn't trigger a release.

### Input: `skip-release-types`

A comma-separated list of conventional commit types that should skip the release. For example:

```yaml
- uses: calavia-org/release-github@v1
  with:
    skip-release-types: 'docs,chore,style,refactor,perf,test,build,ci'
    github-token: ${{ secrets.GITHUB_TOKEN }}
```

When the latest commit matches one of the specified types (e.g., `docs: update README`), the action will:
- Skip tag creation
- Skip release creation
- Skip floating major tag update
- Skip maintenance branch creation

### Output: `skipped`

The action outputs a `skipped` field that indicates whether the release was skipped:

```yaml
- uses: calavia-org/release-github@v1
  id: release
  with:
    skip-release-types: 'docs,chore'
    github-token: ${{ secrets.GITHUB_TOKEN }}

- name: Check if release was skipped
  if: steps.release.outputs.skipped == 'true'
  run: echo "Release was skipped due to commit type"
```

## Testing

No test suite — this is a composite action. Verify by using it in a workflow.

## Gotchas

- `action.yml` uses `custom_tag: ${{ inputs.version }}` — empty string triggers auto-bump from commits
- `no-commit-to-branch` uses regex `/^release\/.+/` for release branches
- `conventional-pre-commit` runs on `commit-msg` stage, not `pre-commit`
