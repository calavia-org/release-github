# Release to GitHub

A GitHub Action for creating releases with semantic versioning and automatic maintenance branches.

## Features

- Automatic version bumping from conventional commits
- Annotated tag creation
- GitHub Release with auto-generated changelog
- Automatic `release/X.x` branch creation on major releases
- Support for custom version input
- Artifact upload support

## Usage

### Basic Release

```yaml
steps:
  - uses: actions/checkout@v4
    with:
      fetch-depth: 0

  - name: Release
    uses: calavia-org/release-github@v1
    with:
      github-token: ${{ secrets.GITHUB_TOKEN }}
```

### Release with Specific Version

```yaml
steps:
  - uses: actions/checkout@v4
    with:
      fetch-depth: 0

  - name: Release
    uses: calavia-org/release-github@v1
    with:
      version: '2.0.0'
      github-token: ${{ secrets.GITHUB_TOKEN }}
```

### Release with Artifacts

```yaml
steps:
  - uses: actions/checkout@v4
    with:
      fetch-depth: 0

  - name: Build
    run: npm run build

  - name: Release
    uses: calavia-org/release-github@v1
    with:
      files: |
        dist/*.tar.gz
        dist/*.zip
      github-token: ${{ secrets.GITHUB_TOKEN }}
```

### Draft Release

```yaml
steps:
  - uses: actions/checkout@v4
    with:
      fetch-depth: 0

  - name: Release
    uses: calavia-org/release-github@v1
    with:
      draft: 'true'
      github-token: ${{ secrets.GITHUB_TOKEN }}
```

## Inputs

| Input | Description | Default |
|-------|-------------|---------|
| `version` | Specific version to release (e.g., `1.2.3`) | Auto-detect from commits |
| `release-name` | Override release name | `Release vX.Y.Z` |
| `body` | Release notes body | Auto-generated changelog |
| `files` | Glob patterns for release artifacts | None |
| `draft` | Create as draft release | `false` |
| `prerelease` | Mark as prerelease | `false` |
| `create-maintenance-branch` | Create `release/X.x` on major releases | `true` |
| `github-token` | GitHub token with write permissions | Required |

## Outputs

| Output | Description |
|--------|-------------|
| `version` | Released version (e.g., `1.2.3`) |
| `tag` | Created git tag (e.g., `v1.2.3`) |
| `release-url` | URL to the created GitHub release |
| `release-id` | ID of the created GitHub release |
| `release-type` | Release type: `major`, `minor`, `patch`, or `custom` |
| `changelog` | Auto-generated changelog from conventional commits |
| `maintenance-branch` | Created maintenance branch (e.g., `release/1.x`) |

## Maintenance Branches

When a **major release** is created (e.g., `v2.0.0`), this action automatically:

1. Creates a `release/1.x` branch from the previous major tag (`v1.x.x`)
2. Pushes the branch to the repository
3. Allows future maintenance releases on that branch

```
main:  v1.0.0 → v1.1.0 → v1.2.0 → v2.0.0 (release)
                                      ↓
                              Creates release/1.x from v1.2.0
```

To release a patch on the maintenance branch:

```yaml
jobs:
  release:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          ref: release/1.x
          fetch-depth: 0

      - name: Release
        uses: calavia-org/release-github@v1
        with:
          github-token: ${{ secrets.GITHUB_TOKEN }}
```

## Conventional Commits

This action uses [conventional commits](https://www.conventionalcommits.org/) for version bumping:

| Commit Type | Version Bump | Example |
|-------------|--------------|---------|
| `fix:` | Patch | `v1.0.0` → `v1.0.1` |
| `feat:` | Minor | `v1.0.0` → `v1.1.0` |
| `feat!:` or `BREAKING CHANGE:` | Major | `v1.0.0` → `v2.0.0` |

## Dependencies

This action uses:

- [mathieudutour/github-tag-action](https://github.com/mathieudutour/github-tag-action) - Semantic version tagging
- [softprops/action-gh-release](https://github.com/softprops/action-gh-release) - GitHub Release creation

## License

MIT
