# CVM Action - Usage Examples

## Table of Contents

- [Basic Usage](#basic-usage)
- [Version Management](#version-management)
- [Publishing](#publishing)
- [Complete CI/CD Pipeline](#complete-cicd-pipeline)
- [Advanced Examples](#advanced-examples)

## Basic Usage

The simplest way to use CVM Action in your CI pipeline:

```yaml
name: Apply Version Bumps

on:
  workflow_dispatch:

jobs:
  version-bump:
    runs-on: ubuntu-latest
    permissions:
      contents: write
      pull-requests: write
    steps:
      - uses: actions/checkout@v4
      - uses: blitzforge/cvm-action@v1
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

## Weekly Automated Version Bumps

Apply accumulated version changes every Monday:

```yaml
name: Weekly Version Bump

on:
  schedule:
    - cron: '0 0 * * 1'  # Monday at midnight UTC
  workflow_dispatch:

jobs:
  apply-versions:
    runs-on: ubuntu-latest
    permissions:
      contents: write
      pull-requests: write
    steps:
      - uses: actions/checkout@v4
      - uses: blitzforge/cvm-action@v1
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        with:
          pr-title: 'chore(release): weekly version bumps'
          pr-labels: 'release,automated'
```

## Check for Pending Changes on PR

Warn when a PR has pending version changes:

```yaml
name: PR Checks

on:
  pull_request:

jobs:
  check-versions:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Check for pending version changes
        id: cvm-check
        uses: blitzforge/cvm-action@v1
        with:
          command: 'status'
          create-pr: 'false'
        continue-on-error: true
      
      - name: Comment if changes pending
        if: steps.cvm-check.outputs.has-changes == 'true'
        uses: actions/github-script@v7
        with:
          script: |
            github.rest.issues.createComment({
              issue_number: context.issue.number,
              owner: context.repo.owner,
              repo: context.repo.repo,
              body: '⚠️ This PR has pending CVM version changes in `.cvm/changes/`'
            })
```

## Dry Run Preview

Preview what changes would be applied without actually applying them:

```yaml
name: Preview Version Changes

on:
  pull_request:
    paths:
      - '.cvm/changes/**'

jobs:
  preview:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Preview changes
        uses: blitzforge/cvm-action@v1
        with:
          dry-run: 'true'
          create-pr: 'false'
```

## Complete CI Workflow

A full example with checking, previewing, and applying:

```yaml
name: Version Management

on:
  pull_request:
    paths:
      - '.cvm/changes/**'
  push:
    branches:
      - main
  workflow_dispatch:

jobs:
  check:
    runs-on: ubuntu-latest
    outputs:
      has-changes: ${{ steps.check.outputs.has-changes }}
    steps:
      - uses: actions/checkout@v4
      - id: check
        uses: blitzforge/cvm-action@v1
        with:
          command: 'status'
          create-pr: 'false'
        continue-on-error: true

  preview:
    needs: check
    if: needs.check.outputs.has-changes == 'true' && github.event_name == 'pull_request'
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: blitzforge/cvm-action@v1
        with:
          dry-run: 'true'
          create-pr: 'false'

  apply:
    needs: check
    if: needs.check.outputs.has-changes == 'true' && github.ref == 'refs/heads/main'
    runs-on: ubuntu-latest
    permissions:
      contents: write
      pull-requests: write
    steps:
      - uses: actions/checkout@v4
      - uses: blitzforge/cvm-action@v1
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        with:
          pr-title: 'chore: apply version bumps'
          pr-labels: 'version-bump,automated'
```

## Custom PR Configuration

Customize the pull request title and labels:

```yaml
- uses: blitzforge/cvm-action@v1
  env:
    GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
  with:
    pr-title: 'chore(release): bump crate versions 🚀'
    pr-labels: 'release,dependencies,automated,semver'
```

## Apply Without Creating PR

Apply changes and commit directly (use with caution):

```yaml
- uses: blitzforge/cvm-action@v1
  with:
    create-pr: 'false'

- name: Commit changes
  run: |
    git config user.name "github-actions[bot]"
    git config user.email "github-actions[bot]@users.noreply.github.com"
    git add .
    git commit -m "chore: apply version bumps"
    git push
```

## Fail CI if Pending Changes

Block merges if there are unapplied version changes:

```yaml
- name: Check for pending changes
  uses: blitzforge/cvm-action@v1
  with:
    command: 'status'
    create-pr: 'false'

- name: Fail if changes pending
  if: steps.check.outputs.has-changes == 'true'
  run: |
    echo "::error::Pending version changes must be applied before merging"
    exit 1
```

## Publishing

### Basic Publishing

Publish crates to crates.io with Git tags and GitHub releases:

```yaml
name: Publish Crates

on:
  workflow_dispatch:

jobs:
  publish:
    runs-on: ubuntu-latest
    permissions:
      contents: write
    steps:
      - uses: actions/checkout@v4
      
      - name: Publish
        uses: blitzforge/cvm-action@v1
        with:
          command: 'publish'
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
          CARGO_REGISTRY_TOKEN: ${{ secrets.CARGO_REGISTRY_TOKEN }}
```

### Publish Preview (Dry-run)

Preview what would be published without actually publishing:

```yaml
- name: Preview publish
  uses: blitzforge/cvm-action@v1
  with:
    command: 'publish'
    dry-run: 'true'
  env:
    GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
    CARGO_REGISTRY_TOKEN: ${{ secrets.CARGO_REGISTRY_TOKEN }}
```

### Publish without GitHub Releases

Create tags and publish to crates.io, but skip GitHub releases:

```yaml
- name: Publish (no releases)
  uses: blitzforge/cvm-action@v1
  with:
    command: 'publish'
    publish-no-release: 'true'
  env:
    GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
    CARGO_REGISTRY_TOKEN: ${{ secrets.CARGO_REGISTRY_TOKEN }}
```

### Publish to crates.io Only

Publish to crates.io without creating Git tags or GitHub releases:

```yaml
- name: Publish to crates.io only
  uses: blitzforge/cvm-action@v1
  with:
    command: 'publish'
    publish-no-tag: 'true'
    publish-no-release: 'true'
  env:
    CARGO_REGISTRY_TOKEN: ${{ secrets.CARGO_REGISTRY_TOKEN }}
```

### Publish with Uncommitted Changes

Allow publishing even with uncommitted changes (use with caution):

```yaml
- name: Publish (allow dirty)
  uses: blitzforge/cvm-action@v1
  with:
    command: 'publish'
    publish-allow-dirty: 'true'
  env:
    GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
    CARGO_REGISTRY_TOKEN: ${{ secrets.CARGO_REGISTRY_TOKEN }}
```

## Complete CI/CD Pipeline

### Automatic Version Management + Publishing

This is the recommended setup for fully automated version management and publishing:

```yaml
name: Version Management & Publishing

on:
  push:
    branches:
      - main
  workflow_dispatch:

jobs:
  check-and-apply:
    runs-on: ubuntu-latest
    permissions:
      contents: write
      pull-requests: write
    outputs:
      has-changes: ${{ steps.cvm.outputs.has-changes }}
    
    steps:
      - name: Checkout repository
        uses: actions/checkout@v4
      
      - name: Check and apply version changes
        id: cvm
        uses: blitzforge/cvm-action@v1
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        with:
          pr-title: 'chore: apply version bumps'
          pr-labels: 'version-bump,automated'

  publish:
    runs-on: ubuntu-latest
    needs: check-and-apply
    # Only publish when there are NO pending changes
    # (meaning versions are already up-to-date in Cargo.toml)
    if: needs.check-and-apply.outputs.has-changes == 'false'
    permissions:
      contents: write
    
    steps:
      - name: Checkout repository
        uses: actions/checkout@v4
      
      - name: Publish crates
        id: publish
        uses: blitzforge/cvm-action@v1
        with:
          command: 'publish'
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
          CARGO_REGISTRY_TOKEN: ${{ secrets.CARGO_REGISTRY_TOKEN }}
      
      - name: Summary
        if: steps.publish.outputs.published == 'true'
        run: |
          echo "✅ Successfully published!"
          echo "Version: ${{ steps.publish.outputs.version }}"
```

**How this works:**

1. **On every push to main:**
   - Checks for pending version changes in `.cvm/changes/`
   
2. **If changes exist:**
   - Applies them to `Cargo.toml`
   - Creates a PR with the version bumps
   - Removes the change files
   - Skips publishing (because changes were just applied)
   
3. **After merging the PR (no pending changes):**
   - Creates Git tags (e.g., `v1.2.3`)
   - Creates GitHub releases
   - Publishes to crates.io
   
4. **Result:**
   - Fully automated version management
   - No manual intervention needed
   - Clean separation between versioning and publishing

### Multi-Environment Publishing

Publish to different environments with different configurations:

```yaml
name: Multi-Environment Publish

on:
  workflow_dispatch:
    inputs:
      environment:
        description: 'Environment to publish to'
        required: true
        type: choice
        options:
          - staging
          - production

jobs:
  publish-staging:
    if: github.event.inputs.environment == 'staging'
    runs-on: ubuntu-latest
    permissions:
      contents: write
    steps:
      - uses: actions/checkout@v4
      
      - name: Publish to staging (no releases)
        uses: blitzforge/cvm-action@v1
        with:
          command: 'publish'
          publish-no-release: 'true'
        env:
          CARGO_REGISTRY_TOKEN: ${{ secrets.CARGO_REGISTRY_TOKEN }}

  publish-production:
    if: github.event.inputs.environment == 'production'
    runs-on: ubuntu-latest
    permissions:
      contents: write
    steps:
      - uses: actions/checkout@v4
      
      - name: Publish to production (full release)
        uses: blitzforge/cvm-action@v1
        with:
          command: 'publish'
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
          CARGO_REGISTRY_TOKEN: ${{ secrets.CARGO_REGISTRY_TOKEN }}
```

## Using Outputs

Use the action outputs in subsequent steps:

```yaml
- name: Check versions
  id: cvm
  uses: blitzforge/cvm-action@v1
  env:
    GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}

- name: Notify Slack if applied
  if: steps.cvm.outputs.applied == 'true'
  run: |
    curl -X POST ${{ secrets.SLACK_WEBHOOK }} \
      -H 'Content-Type: application/json' \
      -d '{"text":"Version bumps applied! 🎉"}'

- name: Notify on publish
  if: steps.cvm.outputs.published == 'true'
  run: |
    echo "Published version: ${{ steps.cvm.outputs.version }}"
    curl -X POST ${{ secrets.SLACK_WEBHOOK }} \
      -H 'Content-Type: application/json' \
      -d "{\"text\":\"🚀 Released version ${{ steps.cvm.outputs.version }}!\"}"
```

## Advanced Examples

### Conditional Publishing Based on Branch

```yaml
name: Smart Publishing

on:
  push:
    branches:
      - main
      - beta

jobs:
  publish:
    runs-on: ubuntu-latest
    permissions:
      contents: write
    steps:
      - uses: actions/checkout@v4
      
      # Publish with releases on main
      - name: Publish (production)
        if: github.ref == 'refs/heads/main'
        uses: blitzforge/cvm-action@v1
        with:
          command: 'publish'
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
          CARGO_REGISTRY_TOKEN: ${{ secrets.CARGO_REGISTRY_TOKEN }}
      
      # Publish without releases on beta
      - name: Publish (beta)
        if: github.ref == 'refs/heads/beta'
        uses: blitzforge/cvm-action@v1
        with:
          command: 'publish'
          publish-no-release: 'true'
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
          CARGO_REGISTRY_TOKEN: ${{ secrets.CARGO_REGISTRY_TOKEN }}
```

### Publishing with Custom Validation

```yaml
- name: Check versions
  id: check
  uses: blitzforge/cvm-action@v1
  with:
    command: 'status'
    create-pr: 'false'

- name: Run tests before publishing
  if: steps.check.outputs.has-changes == 'false'
  run: cargo test --all-features

- name: Publish after tests pass
  if: steps.check.outputs.has-changes == 'false'
  uses: blitzforge/cvm-action@v1
  with:
    command: 'publish'
  env:
    GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
    CARGO_REGISTRY_TOKEN: ${{ secrets.CARGO_REGISTRY_TOKEN }}
```
