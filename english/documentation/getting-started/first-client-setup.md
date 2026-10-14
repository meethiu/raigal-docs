---
description: "Setup guide for teams and organization administrators using Raigal Cloud."
icon: building
---

# Organization setup and onboarding

This guide covers setting up Raigal for your team using the web dashboard at [app.raigal.dev](https://app.raigal.dev) and the [Raigal GitHub App](https://github.com/apps/raigal-cloud).

## Setup steps

| Step | Action | Location | Time |
|---|---|---|---|
| 1. Create organization | Sign up with GitHub or work email | [app.raigal.dev](https://app.raigal.dev) | 1 min |
| 2. Install GitHub App | Add the app to target repositories | [github.com/apps/raigal-cloud](https://github.com/apps/raigal-cloud) | 1 min |
| 3. Copy API token | Copy your `rgl_live_...` token | Settings > API Tokens | 30 sec |
| 4. Add CI workflow | Add `.github/workflows/raigal.yml` | Your repositories | 2 min |
| 5. Invite team | Send invites or configure SSO | Settings > Organization Members | 1 min |

## 1. Create your organization

1. Open [app.raigal.dev](https://app.raigal.dev).
2. Sign in with GitHub or your corporate email.
3. Open the organization menu in the top-right corner and select **Create Organization** (or choose an existing workspace).
4. Enter your organization name.

{% hint style="info" %}
If you manage multiple teams or client accounts, you can switch between organizations from the top-right menu.
{% endhint %}

## 2. Install the Raigal GitHub App

The GitHub App creates pending checks when pull requests open, updates check runs when CI completes, and records merged or closed states.

1. In the Raigal Cloud dashboard, open **Settings** and select **Sync GitHub Accounts** (or open [github.com/apps/raigal-cloud](https://github.com/apps/raigal-cloud)).
2. Select **Install App**.
3. Choose your GitHub organization or personal account.
4. Choose **All repositories** or select specific repositories.
5. Select **Install & Authorize**.

GitHub links those repositories to your Raigal organization.

## 3. Get organization credentials

To allow CI runners or local CLI sessions to send scan results to your dashboard, copy your API key:

1. Open [app.raigal.dev/settings](https://app.raigal.dev).
2. In the **Active Organization Token** card, copy your live key (`rgl_live_...`).
3. To generate a replacement key, select **Generate New Token** under **API Keys**.

{% hint style="tip" %}
To avoid managing static secrets in GitHub, add `permissions: id-token: write` to your workflow. Raigal Cloud validates the runner using GitHub OIDC directly.
{% endhint %}

## 4. Add the GitHub Actions workflow

Add `.github/workflows/raigal.yml` to each repository:

```yaml
name: Raigal Quality Gate

on:
  pull_request:
    branches: [main, develop]
  push:
    branches: [main]

jobs:
  quality-gate:
    name: Code Quality & Slop Gate
    runs-on: ubuntu-latest
    permissions:
      contents: read
      id-token: write
      pull-requests: write
      security-events: write

    steps:
      - name: Checkout Code
        uses: actions/checkout@v4
        with:
          fetch-depth: 0

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
Uploading SARIF reports to GitHub Code Scanning on private repositories requires GitHub Advanced Security (GHAS) enabled on your GitHub account.
{% endhint %}

In GitHub:
1. Open **Settings** > **Secrets and variables** > **Actions**.
2. Add a repository secret named `RAIGAL_TOKEN` with your `rgl_live_...` value (or use GitHub OIDC tokenless auth).

## 5. Invite your team

1. In **Settings**, open the **Members** tab.
2. Select **Invite Members** and enter team email addresses.
3. Choose roles: **Admin** for platform leads, **Member** for engineers.
4. Team members receive an email invite to join your organization on [app.raigal.dev](https://app.raigal.dev).

## Developer machine setup

Developers authenticate local machines to send scan metrics and run agent repairs:

```bash
# Authenticate via browser
raigal login

# Verify active organization and allowed repositories
raigal whoami

# Verify local toolchains and runtimes
raigal doctor

# Capture baseline before running changes scan on legacy repos
raigal hook baseline

# Install interception hooks for active coding agents
raigal hook install --claude
```

## Managing connected repositories

In **Settings** > **Connected Repositories**, administrators can:
- Expand a repository to inspect recent scans, common findings, and branch scores.
- Select **Board** to open the repository's 4-stage remediation board.
- Select **Remove** to delete an inactive repository along with its historical runs and pull request records.
