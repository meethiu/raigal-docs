---
description: "Setup guide for teams and organization administrators using Raigal Cloud."
icon: building
---

# Organization setup and onboarding

This guide covers setting up Raigal for your team using the web dashboard at [app.raigal.dev](https://app.raigal.dev) and the [Raigal GitHub App](https://github.com/apps/raigal-cloud).

## Setup steps

| Step | Action | Location | Time |
|---|---|---|---|
| 1. Create organization | Sign up with GitHub, GitLab, or work email | [app.raigal.dev](https://app.raigal.dev) | 1 min |
| 2. Install GitHub App | Add the app to target repositories | [github.com/apps/raigal-cloud](https://github.com/apps/raigal-cloud) | 1 min |
| 3. Copy API token | Copy your `rgl_live_...` token | Settings > API Tokens | 30 sec |
| 4. Add CI workflow | Add `.github/workflows/raigal.yml` | Your repositories | 2 min |
| 5. Invite team | Send invites or configure SSO | Settings > Organization Members | 1 min |

## 1. Create your organization

1. Go to [app.raigal.dev](https://app.raigal.dev).
2. Sign in with GitHub, GitLab, or your corporate email.
3. Open the organization menu in the top-right corner and select **Create Organization** (or pick an existing workspace).
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

GitHub then links those repositories to your Raigal organization.

## 3. Get organization credentials

To allow GitHub Actions or local CLI sessions to send scan results to your dashboard, copy your API key:

1. Open [app.raigal.dev/settings](https://app.raigal.dev).
2. In the **Active Organization Token** card, copy your live key (`rgl_live_...`).
3. To generate a replacement key, select **Generate New Token** under **API Keys**.

{% hint style="tip" %}
If you do not want to manage static secrets in GitHub, add `permissions: id-token: write` to your workflow. Raigal Cloud can verify the runner using GitHub OIDC directly.
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

    steps:
      - name: Checkout Code
        uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: 22

      - name: Run Raigal Quality Gate
        env:
          RAIGAL_TOKEN: ${{ secrets.RAIGAL_TOKEN }}
          RAIGAL_API_URL: https://app.raigal.dev
        run: |
          npx @methiu/raigal ci --changes --base origin/${{ github.base_ref || 'main' }} --human
```

In GitHub:
1. Open **Settings** > **Secrets and variables** > **Actions**.
2. Add a repository secret named `RAIGAL_TOKEN` with your `rgl_live_...` value.

## 5. Invite your team

1. In **Settings**, open the **Members** tab.
2. Select **Invite Members** and enter team email addresses.
3. Choose roles: **Admin** for platform leads, **Member** for engineers.
4. Team members receive an email invite to join your organization on [app.raigal.dev](https://app.raigal.dev).

## Developer machine setup

Developers can authenticate local machines to send scan metrics and run agent repairs:

```bash
# Authenticate via browser
npx @methiu/raigal login

# Verify active organization
npx @methiu/raigal whoami
```

To install hooks for Claude Code or Cursor:

```bash
npx @methiu/raigal hook install
```

## Managing connected repositories

In **Settings** > **Connected Repositories**, administrators can:
- Expand a repository to inspect recent scans, common findings, and branch scores.
- Select **Board** to open the repository's 4-stage remediation board.
- Select **Remove** to delete an inactive repository along with its historical runs and pull request records.
