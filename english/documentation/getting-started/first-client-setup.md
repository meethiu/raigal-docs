---
description: "Setup guide for teams, organization administrators, and pilot clients using Raigal Cloud."
icon: building
---

# Organization setup and onboarding

This guide covers setting up Raigal for your team using the web dashboard at [app.raigal.dev](https://app.raigal.dev) and the [Raigal GitHub App](https://github.com/apps/raigal-cloud).

Raigal is designed for engineering teams adopting AI coding assistants (Claude Code, Cursor, Copilot, OpenCode). New organizations start with a fully-featured 14-day evaluation trial with complete access to all six analysis engines, pull request checks, and the remediation dashboard.

## Setup overview

| Step | Action | Location | Time |
|---|---|---|---|
| 1. Create organization | Sign up with GitHub, Google, or corporate email | [app.raigal.dev](https://app.raigal.dev) | 1 min |
| 2. Install GitHub App | Connect the app to target repositories | [github.com/apps/raigal-cloud](https://github.com/apps/raigal-cloud) | 1 min |
| 3. Copy API token | Copy your `rgl_live_...` token | Settings > API Tokens | 30 sec |
| 4. Add CI workflow | Add `.github/workflows/raigal.yml` | Target repository | 2 min |
| 5. Verify first scan | Open a pull request or push a commit to main | GitHub | 1 min |
| 6. Invite team | Send invites to developers and platform leads | Settings > Organization Members | 1 min |

## 1. Create your organization workspace

1. Open [app.raigal.dev](https://app.raigal.dev).
2. Sign in with GitHub, Google, or your corporate email.
3. Open the organization menu in the top-right corner and select **Create Organization** (or select your existing workspace).
4. Enter your organization or company name (e.g., `Acme Corp`).
5. Your workspace is immediately created with an active 14-day evaluation trial.

{% hint style="info" %}
If you manage multiple teams, client workspaces, or sandbox environments, you can switch between organizations at any time using the organization selector in the top-right corner.
{% endhint %}

## 2. Install the Raigal GitHub App

The [Raigal GitHub App](https://github.com/apps/raigal-cloud) connects your repositories to the cloud control plane. It automatically creates pending checks on pull requests, resolves check runs when CI runs complete, and keeps remediation boards updated when PRs are merged or closed.

1. In the Raigal Cloud dashboard, go to **Settings** and select **Sync GitHub Accounts** (or open [github.com/apps/raigal-cloud](https://github.com/apps/raigal-cloud) directly).
2. Click **Install App**.
3. Select your GitHub organization or personal account.
4. Choose **All repositories** or select the specific repositories you want Raigal to monitor.
5. Click **Install & Authorize**.

Once installed, GitHub automatically registers your account under **Settings > Allowed Owners** in Raigal Cloud, authorizing CI runs from those repositories.

## 3. Obtain your organization API token

To allow GitHub Actions runners or local terminal sessions to authenticate and transmit scan metrics to your dashboard:

1. Open [app.raigal.dev/settings](https://app.raigal.dev/settings).
2. Under **API Tokens**, find the **Active Organization Token** card and copy your live key (`rgl_live_...`).
3. To generate a dedicated key for CI or deployment runners, click **Create New Token**, give it a descriptive name (e.g., `github-actions-ci`), and copy the displayed token.

{% hint style="danger" %}
API tokens are displayed in full only once. Store your token securely. Raigal Cloud hashes all tokens with SHA-256 and cannot recover lost keys.
{% endhint %}

{% hint style="tip" %}
In GitHub Actions, you can also use tokenless authentication by including `permissions: id-token: write` in your workflow. Raigal Cloud validates the runner using cryptographic GitHub OIDC. However, configuring `RAIGAL_TOKEN` in GitHub Secrets is recommended for full multi-platform consistency.
{% endhint %}

## 4. Add the GitHub Actions workflow

Add `.github/workflows/raigal.yml` to your repository:

```yaml
name: Raigal Quality Gate

on:
  push:
    branches: [main, master, develop]
  pull_request:
    branches: [main, master, develop]

permissions:
  contents: read
  id-token: write        # Enables zero-token GitHub Actions OIDC authentication
  security-events: write # Required if uploading SARIF reports to GitHub Code Scanning

jobs:
  quality-gate:
    name: Code Quality & Slop Gate
    runs-on: ubuntu-latest

    steps:
      - name: Checkout Code
        uses: actions/checkout@v4
        with:
          fetch-depth: 0 # Full git history required for base diff resolution

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: 22

      - name: Run Raigal Quality Gate
        env:
          RAIGAL_TOKEN: ${{ secrets.RAIGAL_TOKEN }}
          # RAIGAL_ORG_ID: ${{ vars.RAIGAL_ORG_ID }} # Required only if repository owner belongs to multiple orgs
        run: |
          if [ "${{ github.event_name }}" = "pull_request" ]; then
            git fetch origin ${{ github.base_ref }} --depth=1
            npx --yes @methiu/raigal@latest ci --changes --base origin/${{ github.base_ref }}
          else
            npx --yes @methiu/raigal@latest ci .
          fi

      - name: Generate SARIF Security Report (Optional)
        if: always()
        run: npx --yes @methiu/raigal@latest scan . --sarif > raigal-results.sarif

      - name: Upload SARIF to GitHub Code Scanning (Optional)
        if: always() && github.event.pull_request.head.repo.full_name == github.repository
        uses: github/codeql-action/upload-sarif@v3
        continue-on-error: true
        with:
          sarif_file: raigal-results.sarif
          category: raigal
```

### Save the secret in GitHub:
1. In your GitHub repository, open **Settings** > **Secrets and variables** > **Actions**.
2. Click **New repository secret**.
3. Name: `RAIGAL_TOKEN`.
4. Secret: paste your `rgl_live_...` token.
5. Click **Add secret**.

## 5. Verify your first scan

To verify that the integration is working end-to-end:

1. **Pull Request scan**: Open a pull request in your repository.
   - The Raigal GitHub App immediately creates an `in_progress` check run named **Raigal Quality Gate**.
   - Your GitHub Actions runner executes `raigal ci --changes --base origin/<base>`.
   - The check run completes with a rich status report, showing the numeric score (0-100), issue counts, and findings breakdown.
   - A corresponding card appears on your Raigal Cloud dashboard under the **Pull Requests** tab.
2. **Main branch scan**: Push a commit directly to `main` (or merge your PR).
   - The workflow executes a full repository scan (`raigal ci .`).
   - The repository baseline health score and active issues update across the **Errors** and **Warnings** tabs on [app.raigal.dev](https://app.raigal.dev).

## 6. Invite your team

1. In Raigal Cloud, navigate to **Settings** > **Members**.
2. Click **Invite Member** and enter your colleague's email address.
3. Select their role:
   - **Admin**: Full access to manage repository connections, API tokens, billing, and team members.
   - **Member**: Access to view repositories, inspect findings, review PR check runs, and use developer tools.
4. The invitee receives an email with an activation link to join your organization workspace.

## Developer local workstation setup

Developers can run scans and automated fixes locally before pushing code:

```bash
# Authenticate local terminal via browser
npx @methiu/raigal login

# Verify active organization and authorized repositories
npx @methiu/raigal whoami

# Check local toolchains and engines
npx @methiu/raigal doctor

# Run full local scan
npx @methiu/raigal scan .

# Apply safe deterministic fixes
npx @methiu/raigal fix .

# Install pre-commit or agent interception hooks
npx @methiu/raigal hook install --claude
```

## Troubleshooting common onboarding issues

| Symptom | Probable Cause | Actionable Fix |
|---|---|---|
| `ambiguous_organization` (HTTP 400) | Repository owner account is associated with multiple Raigal organizations | In your workflow, set `RAIGAL_ORG_ID: ${{ vars.RAIGAL_ORG_ID }}` with your organization ID (`org_...`) from [Settings](https://app.raigal.dev/settings). |
| `owner_not_allowed` (HTTP 403) | Repository owner is not linked to your Raigal organization | In Raigal Cloud, go to **Settings > Allowed Owners** and click **Sync GitHub**, or ensure the GitHub App is installed on the organization owning the repository. |
| Check run stuck on `in_progress` | CI workflow failed before `raigal ci` ran, or workflow is missing | Verify that `.github/workflows/raigal.yml` ran for the PR's `head_sha`. Raigal Cloud automatically cleans up orphaned check runs after 20 minutes. |
| HTTP 401 Unauthorized in CI | Missing or invalid `RAIGAL_TOKEN` | Confirm that `RAIGAL_TOKEN` is added as a repository secret and matches an active token in [Settings > API Tokens](https://app.raigal.dev/settings). |
| Diff preview shows 0 files changed | Shallow git clone in Actions runner | Ensure `actions/checkout@v4` has `fetch-depth: 0` so the runner has full git commit history to resolve diffs against the base branch. |
