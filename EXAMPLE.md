# CVM Action - Usage Examples

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
```
