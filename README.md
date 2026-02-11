# CVM Action - Crate Version Manager

A GitHub Action to automatically manage Rust crate versions with semantic versioning. This action integrates with [CVM (Crate Version Manager)](https://crates.io/crates/cvm_cli) to check for pending version changes and apply them via pull requests.

## 🚀 Quick Start

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

## ✨ Features

- 🎯 Automatic semantic versioning for Rust crates
- 🔄 Support for workspaces and multi-crate projects
- 📦 Staged version changes via `.cvm/changes/` files
- ⚡ CI-friendly with status checks and dry-run mode
- 🤖 Automatic PR creation with version bumps
- ✅ Preserves Cargo.toml formatting

## 📋 CI Pipeline Example

Here's a complete CI pipeline that checks for pending version changes and applies them automatically:

```yaml
name: Version Management

on:
  # Run on changes to version files
  pull_request:
    paths:
      - '.cvm/changes/**'
  # Run weekly to apply accumulated changes
  schedule:
    - cron: '0 0 * * 1'  # Monday at midnight
  # Allow manual trigger
  workflow_dispatch:

jobs:
  check-and-apply:
    runs-on: ubuntu-latest
    permissions:
      contents: write
      pull-requests: write
    
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
      
      - name: Summary
        run: |
          echo "Has pending changes: ${{ steps.cvm.outputs.has-changes }}"
          echo "Changes applied: ${{ steps.cvm.outputs.applied }}"
```

This pipeline will:
1. ✅ Check for pending version changes in `.cvm/changes/`
2. 📝 Apply changes to `Cargo.toml` files
3. 🔀 Create a pull request with the version bumps
4. 🗑️ Remove applied change files

## 📖 How It Works

### 1. Create Version Changes Locally

Use the CVM CLI to create version change files:

```bash
# Install CVM
cargo install cvm_cli

# Interactive version bump
cvm

# This creates a file in .cvm/changes/
```

### 2. Commit Change Files

```bash
git add .cvm/changes/
git commit -m "chore: bump crate-a to v1.2.0"
git push
```

### 3. GitHub Action Applies Changes

The action will:
- Detect the change files
- Update `Cargo.toml` versions
- Create a PR with the changes
- Remove the change files

## 🎮 Usage Examples

### Dry Run (Preview Only)

Preview changes without applying them:

```yaml
- uses: blitzforge/cvm-action@v1
  with:
    dry-run: 'true'
    create-pr: 'false'
```

### Status Check Only

Check for pending changes without applying:

```yaml
- name: Check for pending changes
  id: check
  uses: blitzforge/cvm-action@v1
  with:
    command: 'status'
    create-pr: 'false'

- name: Fail if changes pending
  if: steps.check.outputs.has-changes == 'true'
  run: exit 1
```

### Custom PR Configuration

```yaml
- uses: blitzforge/cvm-action@v1
  env:
    GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
  with:
    pr-title: 'chore(release): version bump'
    pr-labels: 'release,automated,dependencies'
```

### Manual Apply (No PR)

Apply changes without creating a PR:

```yaml
- uses: blitzforge/cvm-action@v1
  with:
    create-pr: 'false'
```

Then commit manually in a subsequent step.

## ⚙️ Inputs

| Input | Description | Default |
|-------|-------------|---------|
| `command` | Command to run: `status`, `apply`, or `check-and-apply` | `check-and-apply` |
| `dry-run` | Run in dry-run mode (preview only) | `false` |
| `create-pr` | Create a pull request with version changes | `true` |
| `pr-title` | Title for the pull request | `chore: apply version bumps` |
| `pr-labels` | Comma-separated labels for the PR | `version-bump,automated` |

## 📤 Outputs

| Output | Description |
|--------|-------------|
| `has-changes` | Whether there are pending version changes (`true`/`false`) |
| `applied` | Whether changes were applied (`true`/`false`) |

## 📁 Change File Format

Change files are stored in `.cvm/changes/` as TOML files:

```toml
[update]
summary = "Add new authentication feature"
minor = ["my-auth-crate"]
patch = ["my-utils-crate"]
pre = false
```

## 🔧 Local CVM Commands

```bash
# Interactive version bump
cvm

# Check pending changes
cvm status

# Preview changes
cvm apply --dry-run

# Apply changes
cvm apply

# Start prerelease mode
cvm pre start canary

# Exit prerelease mode
cvm pre exit
```

## 🧪 Testing

Test the action locally:

```bash
./test-action-local.sh
```

See [TESTING.md](TESTING.md) for detailed testing instructions.

## 📝 Example Workflow Files

Check the [examples/](examples/) directory for:
- `basic.yml` - Simple weekly version bump
- `advanced.yml` - Full CI integration with checks

## 🔒 Permissions

The action requires these permissions to create PRs:

```yaml
permissions:
  contents: write
  pull-requests: write
```

Set the `GITHUB_TOKEN` environment variable:

```yaml
- uses: blitzforge/cvm-action@v1
  env:
    GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

## 🤝 Related Projects

- [CVM CLI](https://github.com/blitzforge/cvm) - The core CVM tool
- [cvm_cli on crates.io](https://crates.io/crates/cvm_cli) - Install with `cargo install cvm_cli`

## 📄 License

MIT License - see the [LICENSE](LICENSE) file for details.

## 🐛 Issues & Contributing

- Report issues: [GitHub Issues](https://github.com/blitzforge/cvm-action/issues)
- Contributions welcome! Please submit a Pull Request.

## 💡 Why Use CVM?

- **Staged Changes**: Version bumps are staged and reviewed before applying
- **Workspace Support**: Handles complex Rust workspaces with multiple crates
- **Prerelease Support**: Built-in support for prerelease versioning (canary, alpha, beta)
- **CI-Friendly**: Designed for automation with clear exit codes and dry-run mode
- **Format Preserving**: Maintains your Cargo.toml formatting and comments

---

**Made with ❤️ by BlitzForge**