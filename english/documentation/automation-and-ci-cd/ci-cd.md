---
description: "Integrate Raigal into GitHub Actions, GitLab CI, and automated pipelines."
icon: route
---

# CI/CD pipelines and GitHub Actions

Configure Raigal to check pull requests in CI and report scores to your dashboard.

## Environment variables

Provide these variables in your CI environment:

| Variable | Required | Description | Example |
|---|---|---|---|
| `RAIGAL_TOKEN` | Yes* | Organization API key (`rgl_live_...`) | `rgl_live_8f3a9b...` |
| `RAIGAL_API_URL` | Optional | Custom backend URL (defaults to `https://app.raigal.dev`) | `https://app.raigal.dev` |

*Note: In GitHub Actions jobs configured with `permissions: id-token: write`, static tokens are not required because Raigal validates the runner using GitHub OIDC directly.*

## GitHub Actions

### 1. Hardened pull request check

Create `.github/workflows/raigal.yml` to evaluate pull requests, report to your dashboard, and generate SARIF annotations:

```yaml
name: Raigal Quality Gate

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main, develop]

permissions:
  contents: read
  id-token: write        # Enables cryptographic OIDC authentication
  pull-requests: write   # Single-comment PR summary reporting
  security-events: write # Required for GitHub Code Scanning SARIF upload

jobs:
  gate:
    name: Code Quality & Slop Gate
    runs-on: ubuntu-latest

    steps:
      - name: Checkout Code
        uses: actions/checkout@v4
        with:
          fetch-depth: 0 # Full git history required for diff base resolution

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: 22

      - name: Fetch Base Branch for Diff Analysis
        if: github.event_name == 'pull_request'
        run: git fetch origin ${{ github.base_ref }} --depth=1

      - name: Run Raigal Quality Gate
        env:
          RAIGAL_TOKEN: ${{ secrets.RAIGAL_TOKEN }}
          RAIGAL_API_URL: https://app.raigal.dev
          IS_FORK: ${{ github.event.pull_request.head.repo.full_name != github.repository }}
        run: |
          if [ "$IS_FORK" = "true" ]; then
            echo "Fork PR: running in advisory mode without write permissions"
            npx -y @methiu/raigal@0.2.2 ci --changes --base origin/${{ github.base_ref }} --human
          else
            npx -y @methiu/raigal@0.2.2 ci --changes --base origin/${{ github.base_ref || 'main' }} --report-pr --sarif-output raigal-results.sarif
          fi

      - name: Upload SARIF to GitHub Code Scanning
        if: always() && github.event.pull_request.head.repo.full_name == github.repository
        uses: github/codeql-action/upload-sarif@v3
        continue-on-error: true
        with:
          sarif_file: raigal-results.sarif
          category: raigal
```

{% hint style="info" %}
Uploading SARIF reports to GitHub Code Scanning on private repositories requires GitHub Advanced Security (GHAS) enabled on your GitHub enterprise account. On public repositories, code scanning is included for free.
{% endhint %}

### 2. Pull request close and merge sync

{% hint style="info" %}
If you use the **Raigal GitHub App**, this step is unnecessary. The app receives `pull_request.closed` webhook events directly from GitHub and updates your board automatically.
{% endhint %}

For standalone CI environments without the GitHub App, sync card status when PRs merge or close:

```yaml
name: Raigal PR Lifecycle

on:
  pull_request:
    types: [closed]

jobs:
  sync:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 22

      - name: Synchronize PR with Raigal Cloud
        env:
          RAIGAL_TOKEN: ${{ secrets.RAIGAL_TOKEN }}
          RAIGAL_API_URL: ${{ secrets.RAIGAL_API_URL || 'https://app.raigal.dev' }}
        run: |
          npx -y @methiu/raigal@0.2.2 report-pr-closed --pr ${{ github.event.pull_request.number }} ${{ github.event.pull_request.merged && '--merged' || '' }}
```

## GitLab CI

In `.gitlab-ci.yml`:

```yaml
raigal_gate:
  stage: test
  image: node:22
  variables:
    RAIGAL_TOKEN: $RAIGAL_TOKEN
    RAIGAL_API_URL: $RAIGAL_API_URL
  script:
    - npx -y @methiu/raigal@0.2.2 ci --human
```
