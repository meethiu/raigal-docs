---
description: "How to install the official Raigal Cloud GitHub App, enforce Pull Request quality gates, set up branch protection, and monitor PRs on the Kanban board."
icon: git-pull-request
---

# Pull Request Quality Gate & Kanban

Integrate Raigal into your Pull Request lifecycle to automatically enforce quality thresholds, detect AI slop, block merge hazards before they hit your main branch, and stream live PR evaluations directly onto the Raigal Cloud Kanban board (`https://app.raigal.dev`).

---

## Architecture: Hybrid GitHub App + Actions Model

Raigal employs an enterprise-grade **Hybrid Architecture** combining the official GitHub App (`raigal-cloud`) and your GitHub Actions CI workflows:

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

### Why Hybrid?
- **Zero Code Exfiltration**: Your proprietary source code stays 100% inside your GitHub Actions runner or sandbox. Raigal Cloud never clones, stores, or accesses your source code.
- **Immediate Pending Feedback**: The moment a PR is opened or synchronized, the official GitHub App instantly registers a `"Raigal Quality Gate"` check in `in_progress` status on the commit head SHA and creates the PR card on your Kanban board.
- **Native Branch Protection**: The `"Raigal Quality Gate"` check appears directly in your repository's Branch Protection rules as an enforceable required check.
- **Automated Merge & Close Detection**: When a PR is merged or closed, GitHub delivers the event directly to the Raigal App, automatically updating the Kanban card without requiring extra CI jobs or manual pings.

---

## Setup: Official GitHub App + Actions (Recommended)

Follow these steps to connect your repository in under 2 minutes:

{% stepper %}

{% step %}
### Install the Official GitHub App
1. Navigate to the official Raigal Cloud GitHub App:  
   👉 [**https://github.com/apps/raigal-cloud**](https://github.com/apps/raigal-cloud)
2. Click **Install**.
3. Select your GitHub organization or personal account.
4. Choose **All repositories** or select specific target repositories you want Raigal to guard.
5. Click **Install & Authorize**.
{% endstep %}

{% step %}
### Obtain Your Organization Token
1. Sign in to your dashboard at [**https://app.raigal.dev**](https://app.raigal.dev).
2. Open **Settings** (or press `⌘ K` and select *Settings*).
3. Under **API Tokens**, click **Create New Token**.
4. Give your token a name (e.g. `GitHub Actions CI`) and copy the generated key (`rgl_live_...`).
{% endstep %}

{% step %}
### Store Secret in GitHub Repository
1. In your GitHub repository, navigate to **Settings** > **Secrets and variables** > **Actions**.
2. Click **New repository secret**.
3. Name: `RAIGAL_TOKEN`
4. Value: Paste your `rgl_live_...` token.
5. Click **Add secret**.

{% hint style="tip" %}
**Zero-Token GitHub Actions OIDC:** If your organization prefers not to store static secrets, you can skip adding `RAIGAL_TOKEN` by enabling `permissions: id-token: write` in your workflow. Raigal Cloud will cryptographically authenticate your runner via GitHub OIDC!
{% endhint %}
{% endstep %}

{% step %}
### Add GitHub Actions Quality Gate Workflow
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
      id-token: write  # Enables Zero-Token GitHub OIDC

    steps:
      - name: Checkout Code
        uses: actions/checkout@v4
        with:
          # CRITICAL: fetch-depth: 0 ensures full git history is fetched
          # so Raigal can compute accurate diffs against the base branch.
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
**Zero Extra Steps on Merge:** Because the official GitHub App receives `pull_request.closed` webhooks natively, you do not need any secondary workflow or `report-pr-closed` script! When the PR is merged or closed, your Kanban board reflects the change automatically.
{% endhint %}
{% endstep %}

{% step %}
### Enforce Branch Protection
1. In your GitHub repository, go to **Settings** > **Branches**.
2. Under **Branch protection rules**, click **Add branch ruleset** (or edit your existing rule for `main`).
3. Check **Require status checks to pass before merging**.
4. In the search box, search for and select **Raigal Quality Gate**.
5. Save changes.

Now, pull requests cannot be merged unless Raigal passes all quality and AI-slop checks!
{% endstep %}

{% endstepper %}

---

## Alternative: Standalone CI Mode (Without GitHub App)

If your enterprise security policy restricts GitHub App installations, you can run Raigal in standalone CI mode:

{% stepper %}

{% step %}
### Configure Repository Secret
In GitHub **Settings** > **Secrets and variables** > **Actions**, add `RAIGAL_TOKEN`.
{% endstep %}

{% step %}
### Add CI Workflow
Add `.github/workflows/raigal.yml` with the workflow shown above. In standalone mode, the check will run as standard GitHub Actions job status.
{% endstep %}

{% step %}
### Add PR Lifecycle Sync Workflow (Optional)
Without the GitHub App, you can notify Raigal Cloud when a PR is merged or closed with a lightweight lifecycle action:

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

---

## Monitoring PRs on the Remediation Kanban

Open [**Raigal Cloud**](https://app.raigal.dev) to track Pull Requests in real time:

1. **Active Cards**: Every evaluated PR appears in the **PULL REQUESTS** column.
   * **Passed Gate**: Highlights with the overall quality score (e.g. `94/100`), author login, source branch, and target branch.
   * **Merge Blocked**: Highlights in red with blocker count and warning tags.
2. **Contextual Inspection Drawer**:
   * Click any PR card to slide open the **Pull Request Drawer**.
   * Inspect all **Blocking Findings** with exact rule IDs, file paths, and offending lines.
   * Review **Diff Statistics** (+additions, -deletions, files changed).
   * Inspect the formatted **Unified Diff Preview**.
   * One-click **Copy git checkout** command to inspect the branch locally, or **Open on GitHub** to jump to the PR.

---

## Handling PRs from Public Forks

{% hint style="danger" %}
**Fork Security:** GitHub Actions automatically suppresses repository secrets (`${{ secrets.RAIGAL_TOKEN }}`) on `pull_request` runs from external forks to prevent secret leakage.
{% endhint %}

If your repository accepts pull requests from public forks, use **GitHub Actions OIDC**:

```yaml
permissions:
  contents: read
  id-token: write  # Required for Zero-Token OIDC
```

When `id-token: write` is set, the Raigal CLI automatically requests a short-lived OIDC token from GitHub and passes it via the `X-GitHub-OIDC` header. Raigal Cloud verifies the cryptographic JWT against GitHub's public OIDC provider (`token.actions.githubusercontent.com`) and confirms that the repository owner is registered under your organization's allowed owners. No static secrets are required.

---

## Troubleshooting Guide

| Symptom | Cause | Solution |
| :--- | :--- | :--- |
| **Check shows Pending indefinitely** | CI workflow hasn't run or timed out | Verify that your GitHub Actions workflow ran successfully for the commit `head_sha`. Raigal Cloud automatically sweeps orphaned pending checks after 20 minutes. |
| **Check Run not appearing on PR** | GitHub App not installed on repository | Ensure the official [raigal-cloud GitHub App](https://github.com/apps/raigal-cloud) is installed on the specific repository. |
| **HTTP 401 / Invalid Token in CI** | Missing or invalid `RAIGAL_TOKEN` | Confirm that `RAIGAL_TOKEN` is correctly set in your GitHub repository secrets and matches an active token in [Settings](https://app.raigal.dev/settings). |
| **HTTP 403 `owner_not_allowed` in CI** | Repo owner isn't synced to Raigal allowlist | Go to **Settings > Allowed Owners** in Raigal Cloud (`https://app.raigal.dev`) and click **Sync GitHub**. |
| **Diff preview shows 0 files changed** | Shallow checkout (`fetch-depth: 1`) | Add `fetch-depth: 0` to your `actions/checkout@v4` step so full git history is available for diffing against the base branch. |
| **PR card doesn't update on merge** | Standalone mode without sync workflow | If not using the GitHub App, add the `report-pr-closed` lifecycle workflow to sync merge and close events. |
