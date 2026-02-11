# CVM Action - Workflow Examples

This directory contains example GitHub Actions workflows demonstrating different use cases for the CVM Action.

## 📁 Available Examples

### 1. [`example-minimal.yml`](./example-minimal.yml)
**Best for:** Simple projects that want automated version management with minimal configuration.

**What it does:**
- ✅ Checks for pending version changes on every push to `main`
- ✅ Creates PR if changes are found
- ✅ Publishes to crates.io automatically when no changes are pending

**When to use:** You want a "set it and forget it" solution.

---

### 2. [`example-complete-pipeline.yml`](./example-complete-pipeline.yml)
**Best for:** Production projects that need testing before publishing.

**What it does:**
- ✅ Checks for pending version changes
- ✅ Creates PR if changes are found
- ✅ Runs full test suite (tests, clippy, formatting)
- ✅ Publishes to crates.io only if tests pass
- ✅ Creates Git tags and GitHub releases

**When to use:** You want quality gates before publishing.

---

### 3. [`example-manual-publish.yml`](./example-manual-publish.yml)
**Best for:** Projects that need manual control over publishing.

**What it does:**
- 🔍 Preview what would be published (dry-run mode)
- ✅ Manual trigger with customizable options
- ✅ Optional: Skip tags or releases
- ✅ Validation checks before publishing

**When to use:** You want to review and manually approve each publish.

---

## 🚀 Quick Start

### Step 1: Choose a Workflow

Copy one of the example workflows to your repository:

```bash
# For minimal setup
cp .github/workflows/example-minimal.yml .github/workflows/cvm.yml

# For complete pipeline
cp .github/workflows/example-complete-pipeline.yml .github/workflows/cvm.yml

# For manual control
cp .github/workflows/example-manual-publish.yml .github/workflows/cvm.yml
```

### Step 2: Add Required Secrets

Add your crates.io API token to repository secrets:

1. Go to https://crates.io/me
2. Generate a new API token
3. Add it to your repository secrets as `CARGO_REGISTRY_TOKEN`
   - Repository Settings → Secrets and variables → Actions → New repository secret

### Step 3: Configure Permissions

Make sure your workflow has the required permissions:

```yaml
permissions:
  contents: write        # For creating tags and releases
  pull-requests: write   # For creating version bump PRs
```

### Step 4: Test It!

1. **Create a version change:**
   ```bash
   # Install CVM CLI
   cargo install cvm_cli
   
   # Create a version change
   cvm
   ```

2. **Commit and push:**
   ```bash
   git add .cvm/changes/
   git commit -m "chore: bump version to 1.2.0"
   git push
   ```

3. **Watch the magic happen:**
   - GitHub Action runs automatically
   - Creates a PR with version updates
   - After merging, publishes to crates.io

---

## 🔄 How the Pipeline Works

### First Run (with pending changes):
```
Push to main
    ↓
Check for changes (.cvm/changes/)
    ↓
Changes found! ✓
    ↓
Apply changes to Cargo.toml
    ↓
Create PR with updates
    ↓
Skip publish step ⏭️
```

### After PR is Merged (no pending changes):
```
Push to main (PR merged)
    ↓
Check for changes (.cvm/changes/)
    ↓
No changes found ✓
    ↓
Run tests (optional)
    ↓
Create Git tags (e.g., v1.2.0)
    ↓
Create GitHub releases
    ↓
Publish to crates.io 🚀
```

---

## 📋 Customization Options

### Publish Options

```yaml
- uses: lucasaarch/cvm-action@v1
  with:
    command: 'publish'
    dry-run: 'true'                    # Preview only
    publish-no-tag: 'true'             # Skip Git tags
    publish-no-release: 'true'         # Skip GitHub releases
    publish-allow-dirty: 'true'        # Allow uncommitted changes
```

### PR Customization

```yaml
- uses: lucasaarch/cvm-action@v1
  with:
    pr-title: 'chore(release): bump versions'
    pr-labels: 'release,automated,semver'
```

### Different Triggers

```yaml
on:
  # Manual trigger
  workflow_dispatch:
  
  # Weekly schedule
  schedule:
    - cron: '0 0 * * 1'  # Monday at midnight
  
  # On every push
  push:
    branches:
      - main
  
  # On version file changes
  pull_request:
    paths:
      - '.cvm/changes/**'
```

---

## 🔍 Outputs

All workflows expose these outputs that you can use in subsequent steps:

```yaml
steps:
  - name: Publish
    id: publish
    uses: lucasaarch/cvm-action@v1
    with:
      command: 'publish'

  - name: Use outputs
    run: |
      echo "Published: ${{ steps.publish.outputs.published }}"
      echo "Version: ${{ steps.publish.outputs.version }}"
      echo "Crates: ${{ steps.publish.outputs.crates }}"
```

---

## 🛠️ Troubleshooting

### "No CARGO_REGISTRY_TOKEN found"

Make sure you've added your crates.io token to repository secrets:
1. Go to Repository Settings → Secrets and variables → Actions
2. Add `CARGO_REGISTRY_TOKEN` with your token from https://crates.io/me

### "Permission denied" errors

Your workflow needs these permissions:
```yaml
permissions:
  contents: write
  pull-requests: write
```

### "Tag already exists"

The publish command automatically skips tags that already exist. This is normal behavior and prevents duplicate releases.

### Publish runs on every push

Make sure you have the condition:
```yaml
if: needs.check-and-apply.outputs.has-changes == 'false'
```

This ensures publishing only happens when there are NO pending changes.

---

## 📚 Related Documentation

- [Main README](../../README.md) - Full documentation
- [Usage Examples](../../EXAMPLE.md) - More examples and patterns
- [CVM CLI](https://github.com/lucasaarch/cvm) - The underlying tool

---

## 💡 Tips

1. **Start with `example-minimal.yml`** if you're new to CVM
2. **Use dry-run mode** to preview changes before committing
3. **Add tests** before publishing (see `example-complete-pipeline.yml`)
4. **Customize PR labels** to integrate with your project management tools
5. **Use `workflow_dispatch`** to manually trigger publishes when needed

---

**Questions?** Open an issue at [github.com/lucasaarch/cvm-action](https://github.com/lucasaarch/cvm-action/issues)
