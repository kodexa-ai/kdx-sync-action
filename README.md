# Kodexa Sync GitHub Action

[![GitHub Marketplace](https://img.shields.io/badge/Marketplace-Kodexa%20Sync-blue.svg?colorA=24292e&colorB=0366d6&style=flat&longCache=true&logo=data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAA4AAAAOCAYAAAAfSC3RAAAABHNCSVQICAgIfAhkiAAAAAlwSFlzAAAM6wAADOsB5dZE0gAAABl0RVh0U29mdHdhcmUAd3d3Lmlua3NjYXBlLm9yZ5vuPBoAAAERSURBVCiRhZG/SsMxFEZPfsVJ61jbxaF0cRQRcRJ9hlYn30IHN/+9iquDCOIsblIrOjqKgy5aKoJQj4O3EEtbPwhJbr6Te28CmdSKeqzeqr0YbfVIrTBKakvtOl5dtTkK+v4HfA9PEyBFCY9AGVgCBLaBp1jPAyfAJ/AAdIEG0dNAiyP7+K1qIfMdonZic6+WJoBJvQlvuwDqcXadUuqPA1NKAlexbRTAIMvMOCjTbMwl1LtI/6KWJ5Q6rT6Ht1MA58AX8Apcqqt5r2qhrgAXQC3CZ6i1+KMd9TRu3MvA3aH/fFPnBodb6oe6HM8+lYHrGdRXW8M9bMZtPXUji69lmf5Cmamq7quNLFZXD9Rq7v0Bpc1o/tp0fisAAAAASUVORK5CYII=)](https://github.com/marketplace/actions/kodexa-sync)

**Zero-configuration GitOps deployment for Kodexa** - Branch determines what gets deployed where.

## Philosophy

**One way to do it.** Branch mappings in your config, environment variables in GitHub secrets. That's it.

- 🌿 **Branch → Deployment** - Push to main → deploys to production
- 🎯 **Zero Knobs** - No manual overrides, no complex logic, just config
- 🔐 **Environment Variables** - API keys from GitHub secrets
- 🧪 **Dry Run** - Preview changes on PRs

## Quick Start

### 1. Define Branch Mappings

Create `sync-config.yaml`:

```yaml
metadata_dir: kodexa-metadata

# Branch determines deployment
branch_mappings:
  - pattern: "main"
    target: production
    environment: prod

  - pattern: "staging"
    target: staging
    environment: staging

  - pattern: "feature/*"
    target: development
    environment: dev

# Environments define API endpoints
environments:
  prod:
    url: https://platform.kodexa.com
    api_key_from_env: KODEXA_PROD_API_KEY

  staging:
    url: https://staging.kodexa.com
    api_key_from_env: KODEXA_STAGING_API_KEY

  dev:
    url: https://dev.kodexa.com
    api_key_from_env: KODEXA_DEV_API_KEY

# Targets define what gets deployed
targets:
  production:
    manifests:
      - organizations/acme-corp/manifest.yaml
      - organizations/acme-corp/modules/*/manifest.yaml

  staging:
    manifests:
      - organizations/acme-corp/manifest.yaml

  development:
    manifests:
      - organizations/acme-corp/manifest.yaml
```

### 2. Add Secrets

Settings → Secrets → Actions:
- `KODEXA_PROD_API_KEY`
- `KODEXA_STAGING_API_KEY`
- `KODEXA_DEV_API_KEY`

### 3. Create Workflow

`.github/workflows/deploy.yml`:

```yaml
name: Deploy

on:
  push:
    branches: [main, staging, 'feature/**']

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: kodexa-ai/kdx-sync-action@v2
        env:
          KODEXA_PROD_API_KEY: ${{ secrets.KODEXA_PROD_API_KEY }}
          KODEXA_STAGING_API_KEY: ${{ secrets.KODEXA_STAGING_API_KEY }}
          KODEXA_DEV_API_KEY: ${{ secrets.KODEXA_DEV_API_KEY }}
```

**Done.** Push to `main` → production. Push to `feature/new-thing` → dev.

## Inputs

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `sync-config` | Path to sync-config.yaml | No | Auto-discovered |
| `metadata-dir` | Metadata directory | No | Auto-discovered |
| `dry-run` | Preview only (`true`/`false`) | No | `false` |
| `kdx-version` | kdx-cli version | No | `latest` |
| `workers` | Number of parallel workers | No | `8` |
| `filter` | Resource filter pattern | No | - |
| `branch` | Override branch detection | No | - |
| `tag` | Override with tag mapping | No | - |

## Outputs

| Output | Description |
|--------|-------------|
| `targets-deployed` | Number of targets deployed |
| `resources-created` | Resources created |
| `resources-updated` | Resources updated |
| `resources-skipped` | Resources unchanged |
| `json-report` | Full JSON deployment report |
| `json-report-path` | Path to JSON report file |

## Examples

### Basic Deployment

The standard pattern - branch determines everything:

```yaml
name: Deploy

on:
  push:
    branches: [main, staging, develop]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: kodexa-ai/kdx-sync-action@v2
        env:
          KODEXA_PROD_API_KEY: ${{ secrets.KODEXA_PROD_API_KEY }}
          KODEXA_STAGING_API_KEY: ${{ secrets.KODEXA_STAGING_API_KEY }}
          KODEXA_DEV_API_KEY: ${{ secrets.KODEXA_DEV_API_KEY }}
```

### Dry Run on PRs

Validate without deploying:

```yaml
name: Validate

on:
  pull_request:
    paths:
      - 'kodexa-metadata/**'
      - 'sync-config.yaml'

jobs:
  validate:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - id: preview
        uses: kodexa-ai/kdx-sync-action@v2
        with:
          dry-run: true
        env:
          KODEXA_DEV_API_KEY: ${{ secrets.KODEXA_DEV_API_KEY }}

      - uses: actions/github-script@v7
        with:
          script: |
            github.rest.issues.createComment({
              issue_number: context.issue.number,
              owner: context.repo.owner,
              repo: context.repo.repo,
              body: `## Deployment Preview

              Targets: ${{ steps.preview.outputs.targets-deployed }}
              Create: ${{ steps.preview.outputs.resources-created }}
              Update: ${{ steps.preview.outputs.resources-updated }}
              Skip: ${{ steps.preview.outputs.resources-skipped }}`
            });
```

### Production with Approval

Require manual approval:

```yaml
name: Production

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    environment: production  # Requires approval in Settings
    steps:
      - uses: actions/checkout@v4
      - uses: kodexa-ai/kdx-sync-action@v2
        env:
          KODEXA_PROD_API_KEY: ${{ secrets.KODEXA_PROD_API_KEY }}
```

### Multi-Target Deployment

Deploy to multiple organizations:

```yaml
# sync-config.yaml
branch_mappings:
  - pattern: "main"
    targets:
      - name: org-a-prod
        environment: prod
      - name: org-b-prod
        environment: prod

targets:
  org-a-prod:
    manifests: [organizations/org-a/manifest.yaml]
  org-b-prod:
    manifests: [organizations/org-b/manifest.yaml]
```

### Parallel Execution & Filtering

Speed up deployments with parallel workers and filter resources:

```yaml
name: Deploy with Parallel Execution

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - id: deploy
        uses: kodexa-ai/kdx-sync-action@v2
        with:
          workers: 8              # 8 parallel workers (default)
          filter: "invoice-*"    # Only deploy resources matching pattern
        env:
          KODEXA_PROD_API_KEY: ${{ secrets.KODEXA_PROD_API_KEY }}

      # Use JSON report in subsequent steps
      - name: Process Results
        run: |
          echo "JSON Report: ${{ steps.deploy.outputs.json-report }}"
```

### Tag-Based Deployment

Deploy based on git tags instead of branches:

```yaml
name: Deploy on Tag

on:
  push:
    tags: ['v*']

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: kodexa-ai/kdx-sync-action@v2
        with:
          tag: ${{ github.ref_name }}  # Use the pushed tag
        env:
          KODEXA_PROD_API_KEY: ${{ secrets.KODEXA_PROD_API_KEY }}
```

With tag_mappings in sync-config.yaml:

```yaml
tag_mappings:
  - pattern: "v*"
    target: production
    environment: prod

  - pattern: "rc-*"
    target: staging
    environment: staging
```

### Using JSON Output

Access the structured deployment report in subsequent steps:

```yaml
name: Deploy with Notifications

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - id: deploy
        uses: kodexa-ai/kdx-sync-action@v2
        env:
          KODEXA_PROD_API_KEY: ${{ secrets.KODEXA_PROD_API_KEY }}

      - name: Send Slack Notification
        uses: slackapi/slack-github-action@v1
        with:
          payload: |
            {
              "text": "Deployment completed: ${{ steps.deploy.outputs.resources-updated }} updated, ${{ steps.deploy.outputs.resources-created }} created"
            }
        env:
          SLACK_WEBHOOK_URL: ${{ secrets.SLACK_WEBHOOK }}

      - name: Upload JSON Report
        uses: actions/upload-artifact@v4
        with:
          name: deployment-report
          path: ${{ steps.deploy.outputs.json-report-path }}
```

## Configuration

### Complete Example

```yaml
metadata_dir: kodexa-metadata

branch_mappings:
  # Production
  - pattern: "main"
    target: production
    environment: prod

  # Staging
  - pattern: "staging"
    target: staging
    environment: staging

  # Development
  - pattern: "develop"
    target: development
    environment: dev

  - pattern: "feature/*"
    target: development
    environment: dev

  # Hotfix to both
  - pattern: "hotfix/*"
    targets:
      - name: staging
        environment: staging
      - name: production
        environment: prod

environments:
  prod:
    url: https://platform.kodexa.com
    api_key_from_env: KODEXA_PROD_API_KEY

  staging:
    url: https://staging.kodexa.com
    api_key_from_env: KODEXA_STAGING_API_KEY

  dev:
    url: https://dev.kodexa.com
    api_key_from_env: KODEXA_DEV_API_KEY

targets:
  production:
    manifests:
      - organizations/acme-corp/manifest.yaml
      - organizations/acme-corp/modules/*/manifest.yaml

  staging:
    manifests:
      - organizations/acme-corp/manifest.yaml

  development:
    manifests:
      - organizations/acme-corp/manifest.yaml
```

### Branch & Tag Patterns

Glob patterns supported for both `branch_mappings` and `tag_mappings`:
- `main` - Exact
- `feature/*` - Any feature branch
- `release/*` - Any release
- `hotfix/*` - Any hotfix
- `v*` - Any version tag
- `*` - Fallback (matches anything)

First match wins.

### Tag Mappings

Deploy based on git tags (useful for release workflows):

```yaml
tag_mappings:
  - pattern: "v*"
    target: production
    environment: prod

  - pattern: "rc-*"
    target: staging
    environment: staging

  - pattern: "beta-*"
    target: development
    environment: dev
```

## Security

### Environment-Specific Secrets

```yaml
# GitHub Secrets
KODEXA_PROD_API_KEY
KODEXA_STAGING_API_KEY
KODEXA_DEV_API_KEY

# Referenced in sync-config.yaml
environments:
  prod:
    api_key_from_env: KODEXA_PROD_API_KEY
```

### Branch Protection

- Require PR reviews for `main`
- Require status checks
- Restrict push access

### Environment Protection

Settings → Environments → `production`:
- Add required reviewers
- Add deployment protection rules

```yaml
jobs:
  deploy:
    environment: production  # Requires approval
```

### Validation First

```yaml
on:
  pull_request:  # Dry run
  push:
    branches: [main]  # Real deploy
```

## Troubleshooting

### Binary Download Failed

Check [releases](https://github.com/kodexa-ai/kdx-cli-releases/releases), specify version:

```yaml
with:
  kdx-version: "v0.1.20"
```

### Branch Mapping Not Found

Verify `sync-config.yaml` has pattern for your branch:

```yaml
branch_mappings:
  - pattern: "*"  # Fallback
    target: development
    environment: dev
```

### Environment Variable Not Found

1. Secret exists in GitHub?
2. `env:` block references it?
3. Name matches `api_key_from_env`?

### Deployment Failed

Run with dry-run first:

```yaml
with:
  dry-run: true
```

Check:
- Manifest files are valid YAML
- Referenced resources exist
- API credentials have permissions

## Migration from v1

**v1** (old): Manual environment logic in workflow

```yaml
# Old - complex
- name: Determine environment
  run: |
    if [[ "${{ github.ref }}" == "refs/heads/main" ]]; then
      echo "url=${{ secrets.PROD_URL }}" >> $GITHUB_ENV
      echo "token=${{ secrets.PROD_TOKEN }}" >> $GITHUB_ENV
    fi

- uses: kodexa-ai/kdx-sync-action@v1
  with:
    kodexa-url: ${{ env.url }}
    kodexa-token: ${{ env.token }}
```

**v2** (new): Branch mappings in config

```yaml
# New - simple
- uses: kodexa-ai/kdx-sync-action@v2
  env:
    KODEXA_PROD_API_KEY: ${{ secrets.KODEXA_PROD_API_KEY }}
```

**Steps:**

1. **Move mappings** to `sync-config.yaml`:
   ```yaml
   branch_mappings:
     - pattern: "main"
       target: production
       environment: prod
   ```

2. **Add environments** to config:
   ```yaml
   environments:
     prod:
       url: https://platform.kodexa.com
       api_key_from_env: KODEXA_PROD_API_KEY
   ```

3. **Simplify workflow**:
   ```yaml
   - uses: kodexa-ai/kdx-sync-action@v2
     env:
       KODEXA_PROD_API_KEY: ${{ secrets.KODEXA_PROD_API_KEY }}
   ```

4. **Delete environment detection logic** - config handles it

## Support

- **Action issues**: [GitHub Issues](https://github.com/kodexa-ai/kdx-sync-action/issues)
- **CLI issues**: support@kodexa.com
- **Platform**: https://support.kodexa.com

---

**Made with ❤️ by [Kodexa](https://www.kodexa.com)**
