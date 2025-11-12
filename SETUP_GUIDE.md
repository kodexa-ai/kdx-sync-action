# Kodexa Sync Action - Setup Guide

This guide walks you through setting up the kdx-sync-action repository on GitHub.

## Repository Created

A complete GitHub Action has been created at `/Users/pdodds/src/kodexa/kdx-sync-action` with:

- ✅ Full action implementation (`action.yml`)
- ✅ Comprehensive documentation (`README.md`)
- ✅ Example workflows and configurations
- ✅ CI/CD workflows for testing and releases
- ✅ All necessary metadata files

## Next Steps

### 1. Create GitHub Repository

Create a new **public** repository on GitHub under the `kodexa-ai` organization:

```bash
# Option 1: Using GitHub CLI (recommended)
cd /Users/pdodds/src/kodexa/kdx-sync-action
gh repo create kodexa-ai/kdx-sync-action --public --source=. --remote=origin --push

# Option 2: Manual setup
# 1. Go to https://github.com/organizations/kodexa-ai/repositories/new
# 2. Repository name: kdx-sync-action
# 3. Visibility: Public
# 4. Don't initialize with README (we already have one)
# 5. Click "Create repository"
```

### 2. Push Initial Code (if using manual setup)

```bash
cd /Users/pdodds/src/kodexa/kdx-sync-action
git remote add origin git@github.com:kodexa-ai/kdx-sync-action.git
git branch -M main
git push -u origin main
```

### 3. Configure Repository Settings

#### a. Branch Protection (recommended)

1. Go to: Settings → Branches → Add rule
2. Branch name pattern: `main`
3. Enable:
   - ✅ Require a pull request before merging
   - ✅ Require status checks to pass before merging
   - ✅ Require conversation resolution before merging

#### b. GitHub Actions Permissions

1. Go to: Settings → Actions → General
2. Workflow permissions:
   - ✅ Read and write permissions (needed for releases)

#### c. Topics (optional but recommended)

Add these topics to help discoverability:
- `github-actions`
- `kodexa`
- `gitops`
- `sync`
- `metadata`
- `automation`

### 4. Test the Action

Create a test repository to validate the action works:

```bash
# Create a test repo
mkdir test-kdx-sync
cd test-kdx-sync
git init

# Create sync config
mkdir -p kodexa-metadata
cat > kodexa-metadata/sync-config.yaml << 'EOF'
metadata_dir: kodexa-metadata
organizations:
  - slug: test-org
    resources:
      - type: taxonomy
        slug: test-taxonomy
EOF

# Create workflow
mkdir -p .github/workflows
cat > .github/workflows/sync.yml << 'EOF'
name: Test Sync
on:
  push:
    branches: [main]

jobs:
  sync:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: kodexa-ai/kdx-sync-action@v1
        with:
          kodexa-url: ${{ secrets.KODEXA_URL }}
          kodexa-token: ${{ secrets.KODEXA_TOKEN }}
          dry-run: true
EOF

# Commit and push
git add -A
git commit -m "Test kdx-sync-action"
# Push to GitHub and add secrets KODEXA_URL and KODEXA_TOKEN
```

### 5. Publish to GitHub Marketplace (optional)

1. Go to your repository on GitHub
2. Click "Draft a release"
3. Create tag: `v1.0.0`
4. Title: `v1.0.0 - Initial Release`
5. Check: ✅ Publish this Action to the GitHub Marketplace
6. Primary category: **Deployment**
7. Click "Publish release"

The action will be available at: `uses: kodexa-ai/kdx-sync-action@v1`

### 6. Update README Badges (after publishing)

Once published, update the README.md with actual marketplace badge:

```markdown
[![GitHub Marketplace](https://img.shields.io/github/v/release/kodexa-ai/kdx-sync-action?label=marketplace&logo=github)](https://github.com/marketplace/actions/kodexa-sync)
```

## Maintenance

### Creating Releases

To create a new release:

1. **Update version**:
   - Update `CHANGELOG.md` with changes
   - Commit changes

2. **Create and push tag**:
   ```bash
   git tag v1.1.0
   git push origin v1.1.0
   ```

3. **Automated release**:
   - The `.github/workflows/release.yml` will automatically create a GitHub release
   - It will also update the major version tag (e.g., `v1` points to latest `v1.x.x`)

### Testing Changes

Before releasing:

1. Test locally using [act](https://github.com/nektos/act)
2. Test in a separate repository using your fork/branch
3. Run the test workflow: `.github/workflows/test.yml`

## Verification Checklist

Before announcing the action, verify:

- [ ] Repository is public and accessible
- [ ] README.md renders correctly on GitHub
- [ ] Examples work correctly
- [ ] CI/CD workflows pass
- [ ] Action can be used in other repositories
- [ ] Marketplace listing is accurate (if published)
- [ ] All links in documentation work
- [ ] License is correct

## Support and Documentation

### User Documentation
- **Primary**: README.md in repository root
- **Examples**: `examples/` directory with real-world patterns
- **Troubleshooting**: Section in README.md

### Developer Documentation
- **Contributing**: CONTRIBUTING.md
- **Code of Conduct**: CODE_OF_CONDUCT.md
- **Changelog**: CHANGELOG.md

## Announcement Template

Once ready, announce the action:

### Internal (Kodexa)

```
📢 New GitHub Action: Kodexa Sync

We've released a GitHub Action that makes it easy to sync Kodexa metadata using GitOps workflows!

🔗 Repository: https://github.com/kodexa-ai/kdx-sync-action
📖 Documentation: Full examples and guides in README
🎯 Use Cases: Multi-environment deployments, PR validation, scheduled syncs

Check it out and let us know what you think!
```

### External (if applicable)

```
We're excited to announce the Kodexa Sync GitHub Action! 🚀

Automate your Kodexa metadata deployments with GitOps:
✅ Multi-environment support (dev/staging/prod)
✅ Dry-run validation on PRs
✅ Scheduled syncs
✅ Full platform support (Linux/macOS/Windows)

Get started: https://github.com/kodexa-ai/kdx-sync-action

#GitOps #Automation #Kodexa
```

## Files Created

### Core Action
- `action.yml` - Action definition with all inputs/outputs
- `README.md` - Comprehensive documentation

### Examples
- `examples/sync-config-basic.yaml` - Simple configuration
- `examples/sync-config-complete.yaml` - All resource types
- `examples/sync-config-multi-org.yaml` - Multi-organization setup
- `examples/workflows/basic-sync.yml` - Basic sync workflow
- `examples/workflows/multi-environment.yml` - Environment-based deployment
- `examples/workflows/pr-validation.yml` - PR validation with dry-run
- `examples/workflows/manual-deployment.yml` - Manual trigger with approval
- `examples/workflows/scheduled-sync.yml` - Scheduled sync

### CI/CD
- `.github/workflows/test.yml` - Comprehensive testing
- `.github/workflows/release.yml` - Release automation

### Metadata
- `LICENSE` - MIT License with kdx-cli notice
- `CONTRIBUTING.md` - Contribution guidelines
- `CODE_OF_CONDUCT.md` - Code of conduct
- `CHANGELOG.md` - Version history
- `.gitignore` - Git ignore patterns
- `.gitattributes` - Line ending handling

## Questions?

Contact: support@kodexa.com
