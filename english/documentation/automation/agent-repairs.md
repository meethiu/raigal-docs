---
description: "Run automated worktree repair sessions with local coding agents."
icon: robot
---

# Automated agent repairs

For findings that require semantic edits or refactoring, `raigal agent` runs a repair session with a local coding tool inside a git worktree.

## How worktree repairs work

To protect your active working branch from unverified edits, `raigal agent` isolates all changes:

1. Raigal creates a temporary git worktree at `.git/raigal-worktrees/<session-id>`.
2. The agent provider (Claude Code, OpenCode, or Codex) receives the diagnostic output and rule expectations.
3. The agent edits code and runs tests inside the worktree.
4. After reviewing the diff, you can run `raigal agent apply <session-id>` to merge changes locally, or pass `--pr` to open a pull request on GitHub.

## Supported providers

Raigal supports three local coding tools:

| Provider | Invocation | Setup command |
|---|---|---|
| Claude Code | `claude` | `raigal agent connect claude` |
| OpenCode | `opencode` | `raigal agent connect opencode` |
| Codex | `codex` | `raigal agent connect codex` |
| Auto-detect | Automatic selection | `raigal agent use auto` |

Check which providers are installed on your machine:
```bash
raigal agent providers
```

## Workflows

### 1. Interactive local repair session

Preview and test changes locally before committing:

```bash
# Preview what the repair session will target
raigal agent plan .

# Start the interactive repair session
raigal agent --provider opencode .

# Check active and recent repair sessions
raigal agent sessions .

# Inspect the diff and diagnostics from a session
raigal agent show <session-id>

# Apply the approved changes back to your working branch
raigal agent apply <session-id>
```

### 2. Automated pull request creation (`--pr`)

To create a branch and open a pull request automatically, run:

```bash
raigal agent --provider claude --pr --target-score 95
```

With `--pr`, Raigal:
1. Runs safe autofixes and passes remaining issues to the agent provider.
2. Checks that tests pass and the quality score meets `--target-score` (default: 90).
3. Commits the changes to an isolated branch (`raigal/repair-...`).
4. Pushes the branch and opens a GitHub pull request.
5. Sends telemetry to Raigal Cloud to create a card in the **Agent Remediations** column.

### 3. Lifecycle synchronization (`report-pr-closed`)

When an agent pull request is merged or closed on GitHub, update your board status with a GitHub Actions workflow:

```yaml
# .github/workflows/raigal-pr-closed.yml
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
      - name: Report PR Closed to Raigal Cloud
        env:
          RAIGAL_TOKEN: ${{ secrets.RAIGAL_TOKEN }}
          RAIGAL_API_URL: ${{ secrets.RAIGAL_API_URL }}
        run: |
          npx @methiu/raigal report-pr-closed --pr ${{ github.event.pull_request.number }} ${{ github.event.pull_request.merged && '--merged' || '' }}
```

{% hint style="success" %}
Opening a pull request with `raigal agent --pr` creates a card in the **Agent Remediations** column. When the PR merges, `report-pr-closed` moves the card to **Merged** and records the score change.
{% endhint %}
