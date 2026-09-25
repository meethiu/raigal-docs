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

*Note: In GitHub Actions jobs configured with `permissions: id-token: write`, you do not need `RAIGAL_TOKEN` because Raigal can authenticate using GitHub OIDC tokens directly.*

## GitHub Actions

### 1. Pull request check

Create `.github/workflows/raigal.yml` to check pull requests and send scores to the dashboard:

```yaml
name: Raigal Quality Gate

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main, develop]

jobs:
  gate:
    name: Code Quality & Slop Gate
    runs-on: ubuntu-latest
    permissions:
      contents: read
      id-token: write

    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - uses: actions/setup-node@v4
        with:
          node-version: 22

      - name: Run Raigal PR Quality Gate
        env:
          RAIGAL_TOKEN: ${{ secrets.RAIGAL_TOKEN }}
          RAIGAL_API_URL: ${{ secrets.RAIGAL_API_URL }}
        run: |
          npx @methiu/raigal ci --changes --base origin/${{ github.base_ref || 'main' }} --human
```

{% hint style="tip" %}
Setting `permissions: id-token: write` allows the runner to generate a short-lived OIDC token. Raigal passes this token in `X-GitHub-OIDC`, so you do not need to store static tokens in repository secrets.
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
          npx @methiu/raigal report-pr-closed --pr ${{ github.event.pull_request.number }} ${{ github.event.pull_request.merged && '--merged' || '' }}
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
    - npx @methiu/raigal ci --human
```

## Pull request annotations and SARIF

To generate annotations in GitHub's code scanning view:

```bash
npx @methiu/raigal scan --sarif > results.sarif
```

Upload the output with the standard GitHub SARIF action (`github/codeql-action/upload-sarif@v3`).
