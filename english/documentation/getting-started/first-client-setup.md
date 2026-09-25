---
description: "Step-by-step onboarding guide for engineering teams, CTOs, and organization administrators using Raigal Cloud."
icon: building
---

# Organization Setup & Onboarding

This guide walks through setting up Raigal for your engineering organization using the hosted platform at **[app.raigal.dev](https://app.raigal.dev)** and the official **[Raigal GitHub App](https://github.com/apps/raigal-cloud)**.

---

## Onboarding Checklist

| Step | Action | Where | Time |
|---|---|---|---|
| **1. Create Organization** | Sign up with GitHub, Google, or corporate email | [app.raigal.dev](https://app.raigal.dev) | 1 min |
| **2. Install GitHub App** | Install the official Raigal App on your repos | [github.com/apps/raigal-cloud](https://github.com/apps/raigal-cloud) | 1 min |
| **3. Get Organization Token** | Copy your `rgl_live_...` API token | Settings &rarr; API Tokens | 30 sec |
| **4. Configure CI/CD** | Add `.github/workflows/raigal.yml` | Your repositories | 2 min |
| **5. Invite Engineers** | Send team invites or configure SSO | Settings &rarr; Organization Members | 1 min |

---

## Step 1: Sign Up & Create Your Organization

1. Navigate to **[app.raigal.dev](https://app.raigal.dev)**.
2. Sign in with GitHub, Google, or your corporate email.
3. In the top-left organization switcher, select **Create Organization** (or choose your existing team workspace).
4. Enter your organization name (e.g. *Acme Engineering*).

{% hint style="info" %}
Raigal supports multi-tenant organization switching. If you belong to multiple teams or manage separate client accounts, you can seamlessly toggle between them using the top-left switcher.
{% endhint %}

---

## Step 2: Install the Official Raigal GitHub App

The Raigal GitHub App manages the Pull Request quality gate lifecycle: it posts immediate pending checks when PRs open, updates the check run when CI completes, and tracks merged/closed states automatically.

1. In the Raigal Cloud dashboard, navigate to **Settings** &rarr; click **Sync GitHub Accounts** (or visit **[github.com/apps/raigal-cloud](https://github.com/apps/raigal-cloud)** directly).
2. Click **Install App**.
3. Select your GitHub organization or personal account.
4. Choose **All repositories** or select specific target repositories you wish to gate with Raigal.
5. Click **Install & Authorize**.

Once installed, GitHub automatically links your repositories to your Raigal Cloud organization.

---

## Step 3: Access Organization Credentials

To allow GitHub Actions or developer CLI sessions to submit quality metrics to your cloud dashboard, retrieve your organization API key:

1. Open **[app.raigal.dev/settings](https://app.raigal.dev)**.
2. In the **Active Organization Token** card, your live key (`rgl_live_...`) is displayed.
3. Click **Copy** to copy the token.
4. If you need to generate a new key or rotate credentials, click **Generate New Token** under **API Keys**.

{% hint style="tip" %}
**Zero-Secret GitHub Actions (Enterprise):** If you prefer not to manage static API tokens in GitHub Secrets, enable `permissions: id-token: write` in your workflow. Raigal Cloud natively authenticates GitHub runner OIDC tokens without requiring static secrets.
{% endhint %}

---

## Step 4: Add the GitHub Actions Quality Gate

Add `.github/workflows/raigal.yml` to each repository you want to protect:

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
      id-token: write  # Enables Zero-Secret GitHub OIDC

    steps:
      - name: Checkout Code
        uses: actions/checkout@v4
        with:
          fetch-depth: 0 # Required to accurately diff PR changes against the base branch

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

In your repository on GitHub:
- Navigate to **Settings** &rarr; **Secrets and variables** &rarr; **Actions**.
- Add a new repository secret:
  - Name: `RAIGAL_TOKEN`
  - Value: Your `rgl_live_...` token from Step 3.

---

## Step 5: Invite Your Engineering Team

1. In **Settings**, click your organization name or the **Members** tab.
2. Click **Invite Members** to send invitations via email.
3. Assign roles (**Admin** for platform leads, **Member** for engineers).
4. Invited engineers receive an email invite to join your organization workspace on **[app.raigal.dev](https://app.raigal.dev)**.

---

## Developer Machine Setup

Developers can authenticate their local machines to stream scan metrics, access organization policies, and run autonomous repairs:

```bash
# Authenticate via browser
npx @methiu/raigal login

# Verify your active organization
npx @methiu/raigal whoami
```

To install real-time interception hooks for Claude Code or Cursor:

```bash
npx @methiu/raigal hook install
```

---

## Managing Connected Repositories

In **Settings &rarr; Connected Repositories**, administrators can:
- **Inspect Live Activity:** Expand any repository to see recent scans, top findings, and branch quality scores.
- **Access the Kanban Board:** Click **Board** to jump directly to that repository's 4-stage remediation board.
- **Remove Retired Repositories:** Click **Remove** (trash icon) to permanently delete an inactive repository, cascading cleanly to remove historical runs and PR records.
