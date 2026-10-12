---
description: "Run an initial scan, review findings, and apply automated fixes."
icon: bolt
---

# Quickstart

Scan a repository and review results in three steps.

{% stepper %}
{% step %}
### 1. Initialize configuration

Create the default configuration files:

```bash
raigal init .
```

This generates:
- `.raigal/config.yml`: score threshold, engine toggles, and exclude paths.
- `.raigal/rules.yml`: rule severity overrides.
{% endstep %}

{% step %}
### 2. Run a scan

Run the scan command from your repository root:

```bash
raigal scan .
```

Raigal runs formatting, linting, quality, and pattern engines in parallel and prints the quality score (0-100):

```text
╭───────────────────────────────────────────────────╮
│  Raigal Quality Score: 94 / 100 [Healthy]         │
╰───────────────────────────────────────────────────╯
Category breakdown:
  • AI Slop:       98/100 (1 finding)
  • Lint & Format: 92/100 (3 findings)
  • Security:      100/100 (Clean)
```
{% endstep %}

{% step %}
### 3. Fix issues

Apply safe automated fixes for formatting, trivial comments, and unused imports:

```bash
raigal fix .
```

For issues that need semantic edits, run `raigal agent .` to start a worktree repair session with Claude Code, Cursor, or OpenCode.
{% endstep %}
{% endstepper %}
