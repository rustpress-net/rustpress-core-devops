# RustPress DevOps

Centralized GitHub Actions and DevOps workflows for the RustPress organization.

## Reusable Workflows

### Release Workflows

| Workflow | Description | Usage |
|----------|-------------|-------|
| `release-rust.yml` | Release workflow for Rust projects | Core, plugins with Rust |
| `release-node.yml` | Release workflow for Node.js projects | Admin UI |
| `release-theme.yml` | Release workflow for theme packages | All themes |
| `release-docs.yml` | Release workflow for documentation | Web dev docs |

## Usage

Reference these workflows from your repository:

```yaml
name: Release

on:
  push:
    branches: [main, master]
  workflow_dispatch:
    inputs:
      version_bump:
        description: "Version bump type"
        required: true
        default: "auto"
        type: choice
        options:
          - auto
          - patch
          - minor
          - major
      release_type:
        description: "Release type"
        required: true
        default: "auto"
        type: choice
        options:
          - auto
          - release
          - pre-release
          - draft

jobs:
  release:
    uses: rustpress-net/rustpress-core-devops/.github/workflows/release-rust.yml@main
    with:
      version_bump: ${{ inputs.version_bump || 'auto' }}
      release_type: ${{ inputs.release_type || 'auto' }}
    secrets: inherit
```

## Release Type Control

Commits can include release type tags:
- `[release]` - Creates an official release, cleans up prereleases/drafts
- `[pre-release]` - Creates a prerelease
- `[draft]` - Creates a draft release

Only official `[release]` builds trigger Docker image creation and cleanup of previous prereleases/drafts.

## License

MIT License - RustPress Team

