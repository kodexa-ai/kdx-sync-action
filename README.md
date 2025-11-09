# Kodexa Sync GitHub Action

[![GitHub Marketplace](https://img.shields.io/badge/Marketplace-Kodexa%20Sync-blue.svg?colorA=24292e&colorB=0366d6&style=flat&longCache=true&logo=data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAA4AAAAOCAYAAAAfSC3RAAAABHNCSVQICAgIfAhkiAAAAAlwSFlzAAAM6wAADOsB5dZE0gAAABl0RVh0U29mdHdhcmUAd3d3Lmlua3NjYXBlLm9yZ5vuPBoAAAERSURBVCiRhZG/SsMxFEZPfsVJ61jbxaF0cRQRcRJ9hlYn30IHN/+9iquDCOIsblIrOjqKgy5aKoJQj4O3EEtbPwhJbr6Te28CmdSKeqzeqr0YbfVIrTBKakvtOl5dtTkK+v4HfA9PEyBFCY9AGVgCBLaBp1jPAyfAJ/AAdIEG0dNAiyP7+K1qIfMdonZic6+WJoBJvQlvuwDqcXadUuqPA1NKAlexbRTAIMvMOCjTbMwl1LtI/6KWJ5Q6rT6Ht1MA58AX8Apcqqt5r2qhrgAXQC3CZ6i1+KMd9TRu3MvA3aH/fFPnBodb6oe6HM8+lYHrGdRXW8M9bMZtPXUji69lmf5Cmamq7quNLFZXD9Rq7v0Bpc1o/tp0fisAAAAASUVORK5CYII=)](https://github.com/marketplace/actions/kodexa-sync)

Synchronize your Kodexa metadata repositories using GitOps workflows. This action enables you to:

- 🔄 Push metadata changes from GitHub to Kodexa platform
- 🌍 Deploy configurations to different environments (dev, staging, production)
- 🔐 Securely manage credentials using GitHub secrets
- 🧪 Validate changes with dry-run mode before deployment
- 📊 Track sync statistics in workflow outputs

## Table of Contents

- [Quick Start](#quick-start)
- [Inputs](#inputs)
- [Outputs](#outputs)
- [Usage Examples](#usage-examples)
  - [Basic Sync](#basic-sync)
  - [Multi-Environment Deployment](#multi-environment-deployment)
  - [Dry Run Validation](#dry-run-validation)
  - [Filtered Sync](#filtered-sync)
  - [Pull Request Preview](#pull-request-preview)
- [Sync Configuration](#sync-configuration)
- [Security Best Practices](#security-best-practices)
- [Troubleshooting](#troubleshooting)

## Quick Start

1. **Create a sync configuration file** in your repository (e.g., `kodexa-metadata/sync-config.yaml`):

```yaml
metadata_dir: kodexa-metadata

organizations:
  - slug: my-org
    resources:
      - type: taxonomy
        slug: document-classification
      - type: store
        slug: document-store
      - type: featureType
        slug: text-extraction
```

2. **Add secrets to your GitHub repository**:
   - `KODEXA_URL` - Your Kodexa platform URL
   - `KODEXA_TOKEN` - Your Kodexa API token

3. **Create a workflow file** (`.github/workflows/sync.yml`):

```yaml
name: Sync to Kodexa

on:
  push:
    branches: [main]

jobs:
  sync:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Sync to Kodexa
        uses: kodexa-ai/kdx-sync-action@v1
        with:
          kodexa-url: ${{ secrets.KODEXA_URL }}
          kodexa-token: ${{ secrets.KODEXA_TOKEN }}
```

## Inputs

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `kodexa-url` | Kodexa platform URL (e.g., `https://platform.kodexa.com`) | ✅ Yes | - |
| `kodexa-token` | Kodexa API authentication token | ✅ Yes | - |
| `sync-config` | Path to sync-config.yaml | ❌ No | Auto-discovered |
| `metadata-dir` | Metadata directory override | ❌ No | Auto-discovered |
| `dry-run` | Preview changes without applying (`true`/`false`) | ❌ No | `false` |
| `kdx-version` | kdx-cli version to use | ❌ No | `latest` |
| `organization` | Organization slug filter (comma-separated) | ❌ No | - |
| `project` | Project reference filter (comma-separated, format: `org-slug/project-slug`) | ❌ No | - |

## Outputs

| Output | Description |
|--------|-------------|
| `organizations` | Number of organizations synced |
| `resources-created` | Number of resources created |
| `resources-updated` | Number of resources updated |
| `resources-skipped` | Number of resources skipped (unchanged) |

## Usage Examples

### Basic Sync

Sync all resources on every push to main:

```yaml
name: Sync to Production

on:
  push:
    branches: [main]

jobs:
  sync:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Sync to Kodexa
        uses: kodexa-ai/kdx-sync-action@v1
        with:
          kodexa-url: ${{ secrets.KODEXA_PROD_URL }}
          kodexa-token: ${{ secrets.KODEXA_PROD_TOKEN }}
```

### Multi-Environment Deployment

Deploy to different environments based on branch:

```yaml
name: Multi-Environment Sync

on:
  push:
    branches:
      - main
      - staging
      - develop

jobs:
  sync:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Determine environment
        id: env
        run: |
          if [[ "${{ github.ref }}" == "refs/heads/main" ]]; then
            echo "name=production" >> $GITHUB_OUTPUT
          elif [[ "${{ github.ref }}" == "refs/heads/staging" ]]; then
            echo "name=staging" >> $GITHUB_OUTPUT
          else
            echo "name=development" >> $GITHUB_OUTPUT
          fi

      - name: Sync to Kodexa
        uses: kodexa-ai/kdx-sync-action@v1
        with:
          kodexa-url: ${{ secrets[format('KODEXA_{0}_URL', steps.env.outputs.name)] }}
          kodexa-token: ${{ secrets[format('KODEXA_{0}_TOKEN', steps.env.outputs.name)] }}

      - name: Report sync results
        if: success()
        run: |
          echo "✅ Synced to ${{ steps.env.outputs.name }}"
          echo "Organizations: ${{ steps.sync.outputs.organizations }}"
          echo "Created: ${{ steps.sync.outputs.resources-created }}"
          echo "Updated: ${{ steps.sync.outputs.resources-updated }}"
```

### Dry Run Validation

Validate changes on pull requests without applying them:

```yaml
name: Validate Sync

on:
  pull_request:
    paths:
      - 'kodexa-metadata/**'
      - '.github/workflows/**'

jobs:
  validate:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Dry run sync
        uses: kodexa-ai/kdx-sync-action@v1
        with:
          kodexa-url: ${{ secrets.KODEXA_URL }}
          kodexa-token: ${{ secrets.KODEXA_TOKEN }}
          dry-run: true

      - name: Comment PR
        uses: actions/github-script@v7
        if: always()
        with:
          script: |
            const output = `### 🧪 Sync Validation Results

            - Organizations: ${{ steps.sync.outputs.organizations }}
            - Resources to create: ${{ steps.sync.outputs.resources-created }}
            - Resources to update: ${{ steps.sync.outputs.resources-updated }}
            - Resources unchanged: ${{ steps.sync.outputs.resources-skipped }}

            ${process.env.GITHUB_WORKFLOW_CONCLUSION === 'success' ? '✅ Validation passed!' : '❌ Validation failed!'}
            `;

            github.rest.issues.createComment({
              issue_number: context.issue.number,
              owner: context.repo.owner,
              repo: context.repo.repo,
              body: output
            });
```

### Filtered Sync

Sync only specific organizations or projects:

```yaml
name: Sync Specific Resources

on:
  workflow_dispatch:
    inputs:
      organizations:
        description: 'Organizations to sync (comma-separated)'
        required: false
      projects:
        description: 'Projects to sync (format: org/project, comma-separated)'
        required: false

jobs:
  sync:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Sync filtered resources
        uses: kodexa-ai/kdx-sync-action@v1
        with:
          kodexa-url: ${{ secrets.KODEXA_URL }}
          kodexa-token: ${{ secrets.KODEXA_TOKEN }}
          organization: ${{ github.event.inputs.organizations }}
          project: ${{ github.event.inputs.projects }}
```

### Pull Request Preview

Automatically preview sync changes on pull requests:

```yaml
name: PR Sync Preview

on:
  pull_request:
    types: [opened, synchronize]
    paths:
      - 'kodexa-metadata/**'

jobs:
  preview:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Dry run sync
        id: preview
        uses: kodexa-ai/kdx-sync-action@v1
        with:
          kodexa-url: ${{ secrets.KODEXA_URL }}
          kodexa-token: ${{ secrets.KODEXA_TOKEN }}
          dry-run: true

      - name: Generate summary
        run: |
          cat >> $GITHUB_STEP_SUMMARY << EOF
          ## 🔍 Sync Preview

          This pull request will sync the following changes:

          | Metric | Count |
          |--------|-------|
          | Organizations | ${{ steps.preview.outputs.organizations }} |
          | Resources to create | ${{ steps.preview.outputs.resources-created }} |
          | Resources to update | ${{ steps.preview.outputs.resources-updated }} |
          | Resources unchanged | ${{ steps.preview.outputs.resources-skipped }} |

          **Note:** This is a preview. No changes have been applied to Kodexa.
          EOF
```

### Manual Deployment with Approvals

Require manual approval before syncing to production:

```yaml
name: Production Deployment

on:
  workflow_dispatch:

jobs:
  sync:
    runs-on: ubuntu-latest
    environment: production  # Requires environment protection rules
    steps:
      - uses: actions/checkout@v4

      - name: Sync to Production
        uses: kodexa-ai/kdx-sync-action@v1
        with:
          kodexa-url: ${{ secrets.KODEXA_PROD_URL }}
          kodexa-token: ${{ secrets.KODEXA_PROD_TOKEN }}

      - name: Notify on success
        if: success()
        run: |
          echo "✅ Production sync completed successfully"
          # Add notification logic (Slack, email, etc.)
```

### Schedule-Based Sync

Automatically sync on a schedule:

```yaml
name: Scheduled Sync

on:
  schedule:
    - cron: '0 2 * * *'  # Run at 2 AM UTC daily
  workflow_dispatch:  # Allow manual triggers

jobs:
  sync:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Sync to Kodexa
        uses: kodexa-ai/kdx-sync-action@v1
        with:
          kodexa-url: ${{ secrets.KODEXA_URL }}
          kodexa-token: ${{ secrets.KODEXA_TOKEN }}

      - name: Report results
        if: always()
        run: |
          echo "Sync completed at $(date)"
          echo "Organizations: ${{ steps.sync.outputs.organizations }}"
```

## Sync Configuration

The action uses a `sync-config.yaml` file to define which resources to synchronize. Here's a complete example:

### Resource-Based Configuration (Recommended)

```yaml
# sync-config.yaml
metadata_dir: kodexa-metadata

organizations:
  - slug: my-organization
    resources:
      # Taxonomies (classification hierarchies)
      - type: taxonomy
        slug: document-classification

      - type: taxonomy
        slug: entity-types

      # Data stores
      - type: store
        slug: document-store

      - type: store
        slug: vector-store

      # Knowledge types
      - type: knowledgeType
        slug: conditional-rules

      # Feature types
      - type: featureType
        slug: text-extraction

      # Feature instances
      - type: featureInstance
        slug: ocr-extractor-prod

      # Models
      - type: model
        slug: invoice-classifier

      # Project templates
      - type: projectTemplate
        slug: invoice-processing

      # Data forms
      - type: dataForm
        slug: invoice-form
```

### Project-Based Configuration (Legacy)

```yaml
# sync-config.yaml
metadata_dir: kodexa-metadata

organizations:
  - slug: my-organization
    projects:
      - finance-automation
      - operations-bot

  - slug: research-org
    # Omit projects to sync all
```

### Supported Resource Types

**Organization-scoped resources:**
- `taxonomy` (aliases: `label`)
- `store` (aliases: `datastore`, `documentstore`)
- `knowledgeType`
- `featureType`
- `featureInstance`
- `model` (alias: `cloudmodel`)
- `projectTemplate`
- `dataForm`
- `exceptionType`
- `project`

**Project-scoped resources** (requires project context):
- `knowledgeItem`
- `assistant`
- `workflow`
- `action`
- `exception`

## Security Best Practices

### 1. Use GitHub Secrets

Never hardcode credentials in your workflows. Always use GitHub secrets:

```yaml
with:
  kodexa-url: ${{ secrets.KODEXA_URL }}      # ✅ Correct
  kodexa-token: ${{ secrets.KODEXA_TOKEN }}  # ✅ Correct

# ❌ Never do this:
with:
  kodexa-url: "https://my-instance.kodexa.com"
  kodexa-token: "my-api-key-12345"
```

### 2. Use Environment Secrets for Multi-Environment

For multiple environments, use environment-specific secrets:

```yaml
jobs:
  sync:
    environment: production  # Links to 'production' environment in GitHub
    steps:
      - uses: kodexa-ai/kdx-sync-action@v1
        with:
          kodexa-url: ${{ secrets.KODEXA_URL }}      # Environment-specific
          kodexa-token: ${{ secrets.KODEXA_TOKEN }}  # Environment-specific
```

### 3. Restrict Branch Access

Use branch protection rules to control when syncs occur:

```yaml
on:
  push:
    branches:
      - main  # Only sync from main branch
```

### 4. Require Approval for Production

Use environment protection rules in GitHub to require manual approval before production deployments.

### 5. Use Dry Run for Validation

Always validate changes with dry run before applying:

```yaml
# On PR: dry run validation
on: pull_request

# On merge: actual sync
on:
  push:
    branches: [main]
```

## Troubleshooting

### Binary Download Failed

**Problem:** Action fails with "Failed to download kdx"

**Solutions:**
1. Check that the specified kdx version exists in [releases](https://github.com/kodexa-ai/kdx-cli-releases/releases)
2. Verify network connectivity from GitHub Actions runner
3. Try specifying an explicit version instead of "latest":
   ```yaml
   with:
     kdx-version: "v1.0.0"
   ```

### Authentication Errors

**Problem:** "API authentication errors" or "401 Unauthorized"

**Solutions:**
1. Verify your `KODEXA_TOKEN` secret is correct and not expired
2. Ensure the token has appropriate permissions for the resources being synced
3. Check that `KODEXA_URL` is correct and accessible

### Config Not Found

**Problem:** "sync-config.yaml not found"

**Solutions:**
1. Ensure `sync-config.yaml` exists in your repository
2. Specify the path explicitly:
   ```yaml
   with:
     sync-config: path/to/sync-config.yaml
   ```
3. Check file permissions and repository structure

### Validation Errors

**Problem:** Sync fails with validation errors

**Solutions:**
1. Run with `dry-run: true` to see detailed error messages
2. Verify resource slugs exist in your Kodexa instance
3. Check that all required fields are present in metadata files
4. Ensure feature instances reference valid feature types

### Unsupported OS/Architecture

**Problem:** "Unsupported OS" or "Unsupported architecture"

**Supported platforms:**
- Linux: amd64, arm64
- macOS: amd64, arm64
- Windows: amd64

If you encounter this error:
1. Check the runner OS: `runs-on: ubuntu-latest` (recommended)
2. Verify the architecture matches supported platforms

## Contributing

Contributions are welcome! Please feel free to submit issues or pull requests.

## License

Copyright © 2025 Kodexa, Inc. All rights reserved.

This action interfaces with the proprietary kdx-cli tool. See [LICENSE](LICENSE) for details.

## Support

For issues related to:
- **This GitHub Action**: Open an issue in this repository
- **kdx-cli tool**: Contact support@kodexa.com
- **Kodexa platform**: Visit https://support.kodexa.com

---

**Made with ❤️ by [Kodexa](https://www.kodexa.com)**
