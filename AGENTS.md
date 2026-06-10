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

```bash
source .venv/bin/activate
pre-commit run --all-files
```

Pre-commit hooks:
- `actionlint` — GitHub Actions linting
- `check-yaml`, `check-merge-conflict`, `trailing-whitespace`, `end-of-file-fixer`
- `no-commit-to-branch` — protects `main`, `master`, `release/*`
- `conventional-pre-commit` — enforces conventional commits on commit-msg
- `markdownlint` — markdown linting

## Branch Protection

Direct commits to `main`, `master`, and `release/*` are blocked by pre-commit. Use feature branches:
- `feat/` — new features (triggers minor version bump)
- `fix/` — bug fixes (triggers patch version bump)
- `chore/` — maintenance (no version bump)

## Versioning

- **Source of truth**: Git tags (`v1.0.0`, `v1.1.0`, etc.)
- **Floating tags**: `v1`, `v2` — point to latest in major line
- **Bump rules**: Conventional commits determine bump type (feat=minor, fix=patch, BREAKING=major)

## Testing

No test suite — this is a composite action. Verify by using it in a workflow.

## Gotchas

- `action.yml` uses `custom_tag: ${{ inputs.version }}` — empty string triggers auto-bump from commits
- `no-commit-to-branch` uses regex `/^release\/.+/` for release branches
- `conventional-pre-commit` runs on `commit-msg` stage, not `pre-commit`
