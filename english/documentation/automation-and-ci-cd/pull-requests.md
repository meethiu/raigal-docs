---
description: "How to install the GitHub App, enforce pull request quality gates, configure branch protection, and track PRs."
icon: git-pull-request
---

# Pull request quality gate and board

Set up Raigal to check pull requests, enforce branch protection, and track review status on your remediation board at `https://app.raigal.dev`.

## Architecture: GitHub App and Actions

Raigal combines the official GitHub App (`raigal-cloud`) with your GitHub Actions workflows:

```
┌────────────────────────────────────────────────────────────────────────┐
│                          GitHub Ecosystem                              │
│                                                                        │
│   1. Developer opens PR / pushes commit                                │
│          │                                                             │
│          ▼                                                             │
│   GitHub Webhook ──────────────────────────┐                           │
│          │                                 │                           │
│          ▼ (triggers CI)                   │ (pull_request event)      │
│   GitHub Actions Runner                    │                           │
│     • Runs AST Lint & Slop Scans           ▼                           │
│     • Computes diff & score        Raigal Cloud Control Plane          │
│     • Zero code leaves runner       (https://app.raigal.dev)           │
│          │                                 │                           │
│          ▼ (POST /v1/runs)                 │ 2. Creates Pending Check  │
│   Ingest Metrics & Head SHA                │    "Raigal Quality Gate"  │
│          │                                 ▼                           │
│          └─────────────────────────► Resolves Check Run                │
│                                      (Success / Failure + Summary)     │
│                                            │                           │
│                                            ▼                           │
│   Branch Protection Enforced ◄─────────────┘                           │
│   Instant Kanban Board Card Updated                                    │
└────────────────────────────────────────────────────────────────────────┘
```

How this setup works:
- Code stays in your runner: source code never leaves your local machine or GitHub Actions runner. Raigal Cloud does not clone or store your code.
- Immediate pending check: when a pull request opens or receives new commits, the GitHub App creates a `"Raigal Quality Gate"` check in `in_progress` status on the head commit and adds a card to your board.
- Branch protection: the `"Raigal Quality Gate"` check can be selected as a required status check in GitHub branch rules.
- Automatic updates on merge or close: GitHub sends merge and close webhook events directly to the app, which updates board cards without extra CI steps.

## Setup: GitHub App and Actions

{% stepper %}

{% step %}
### 1. Install the GitHub App
1. Open the [Raigal Cloud GitHub App](https://github.com/apps/raigal-cloud).
2. Select **Install**.
3. Choose your organization or personal account.
4. Select all repositories or choose specific ones.
5. Select **Install & Authorize**.
{% endstep %}

{% step %}
### 2. Copy your organization token
1. Sign in to your dashboard at [app.raigal.dev](https://app.raigal.dev).
2. Open **Settings**.
3. Under **API Tokens**, select **Create New Token**.
4. Name the token and copy the generated key (`rgl_live_...`).
{% endstep %}

{% step %}
### 3. Add token to GitHub repository
1. In your GitHub repository, open **Settings** > **Secrets and variables** > **Actions**.
2. Select **New repository secret**.
3. Set the name to `RAIGAL_TOKEN` and paste your key.
4. Select **Add secret**.

{% hint style="tip" %}
If you prefer not to manage static tokens, add `permissions: id-token: write` to your workflow. Raigal Cloud authenticates the runner using GitHub OIDC directly.
{% endhint %}
{% endstep %}

{% step %}
### 4. Add the GitHub Actions workflow
Create `.github/workflows/raigal.yml` in your repository:

```yaml
name: Raigal Quality Gate

on:
  pull_request:
    types: [opened, synchronize, reopened]
    branches: [main, develop]
  push:
    branches: [main]

jobs:
  gate:
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
          RAIGAL_API_URL: ${{ secrets.RAIGAL_API_URL || 'https://app.raigal.dev' }}
        run: |
          npx @methiu/raigal ci --changes --base origin/${{ github.base_ref || 'main' }} --human
```

{% hint style="success" %}
Because the GitHub App receives `pull_request.closed` events directly, you do not need an additional workflow to notify Raigal when PRs close.
{% endhint %}
{% endstep %}

{% step %}
### 5. Require status check in branch protection
1. In your GitHub repository, open **Settings** > **Branches**.
2. Under **Branch protection rules**, edit the rule for your default branch.
3. Check **Require status checks to pass before merging**.
4. Search for and select **Raigal Quality Gate**.
5. Save changes.
{% endstep %}

{% endstepper %}

## Standalone CI mode (without GitHub App)

If your organization does not allow GitHub App installations, you can run Raigal using GitHub Actions alone:

{% stepper %}

{% step %}
### 1. Configure secret
In **Settings** > **Secrets and variables** > **Actions**, add `RAIGAL_TOKEN`.
{% endstep %}

{% step %}
### 2. Add workflow
Add `.github/workflows/raigal.yml` using the workflow above. In standalone mode, the gate reports status as a standard Actions job.
{% endstep %}

{% step %}
### 3. Add close sync workflow (optional)
To update the board when PRs close without the GitHub App, add a second workflow:

```yaml
name: Raigal PR Lifecycle

on:
  pull_request:
    types: [closed]

jobs:
  sync-pr-closed:
    name: Sync Closed PR Status
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 22

      - name: Report PR Closure to Raigal Cloud
        env:
          RAIGAL_TOKEN: ${{ secrets.RAIGAL_TOKEN }}
          RAIGAL_API_URL: ${{ secrets.RAIGAL_API_URL || 'https://app.raigal.dev' }}
        run: |
          npx @methiu/raigal report-pr-closed \
            --pr ${{ github.event.pull_request.number }} \
            ${{ github.event.pull_request.merged && '--merged' || '' }}
```
{% endstep %}

{% endstepper %}

## Tracking PRs on the board

In [Raigal Cloud](https://app.raigal.dev), open your repository board:

1. Pull requests column:
   - Passed: shows the quality score (e.g. `94/100`), author, source branch, and target branch.
   - Blocked: shows blocker counts and failing rule categories.
2. Inspection drawer:
   - Click a card to open its detail panel.
   - Review blocking findings with rule IDs, file paths, and line numbers.
   - Inspect diff statistics and unified diff preview.
   - Copy the `git checkout` command to inspect changes locally, or select **Open on GitHub** to jump to the PR.

## PRs from public forks

{% hint style="danger" %}
GitHub Actions suppresses repository secrets on `pull_request` triggers from external forks to prevent credential leaks.
{% endhint %}

If your repository accepts pull requests from forks, use GitHub Actions OIDC:

```yaml
permissions:
  contents: read
  id-token: write
```

With `id-token: write`, the CLI requests a short-lived OIDC token from GitHub and passes it via `X-GitHub-OIDC`. Raigal Cloud verifies the token with `token.actions.githubusercontent.com` and confirms that the repository owner is registered to your organization.

## Troubleshooting

| Symptom | Cause | Solution |
| :--- | :--- | :--- |
| Check remains pending | CI workflow did not run or timed out | Check that the workflow completed for the commit `head_sha`. Raigal Cloud cleans up orphaned pending checks after 20 minutes. |
| Check run does not appear on PR | GitHub App not installed on repository | Ensure the [raigal-cloud GitHub App](https://github.com/apps/raigal-cloud) is installed on the repository. |
| HTTP 401 or invalid token in CI | Missing or expired `RAIGAL_TOKEN` | Confirm that `RAIGAL_TOKEN` is set in GitHub repository secrets and matches an active token in [Settings](https://app.raigal.dev/settings). |
| HTTP 403 `owner_not_allowed` in CI | Repository owner not in allowlist | Go to **Settings > Allowed Owners** in Raigal Cloud and select **Sync GitHub**. |
| Diff preview shows 0 files changed | Shallow checkout (`fetch-depth: 1`) | Set `fetch-depth: 0` in `actions/checkout@v4` so full git history is available for comparison against the base branch. |
| PR card does not update on merge | Standalone mode without sync workflow | If running without the GitHub App, add the `report-pr-closed` workflow. |
