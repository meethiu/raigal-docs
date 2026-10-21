---
description: "Authenticate, initialize your configuration, run an initial scan, and apply automated fixes."
icon: bolt
---

# Quickstart

Scan a repository and review results in four steps.

{% stepper %}
{% step %}
### 1. Authenticate with your organization

Log in to link your local CLI session with your Raigal organization:

```bash
raigal login
```

Alternatively, export your organization API key or machine token:

```bash
export RAIGAL_TOKEN="rgl_live_your_token_here"
```
{% endstep %}

{% step %}
### 2. Initialize configuration

Create your configuration and choose your CI pipeline:

```bash
raigal init
```

This interactive wizard detects your project structure and generates:
- `.raigal/config.yml`: score thresholds, engine toggles, and exclude paths.
- `.raigal/rules.yml`: architecture and severity rule overrides.
- CI pipeline workflow (GitHub Actions, GitLab CI, or Bitbucket Pipelines).
{% endstep %}

{% step %}
### 3. Run a scan

Run the scan command from your repository root:

```bash
raigal scan .
```

Raigal runs formatting, linting, code-quality, AI-pattern, and security engines in parallel and outputs the calibrated score (0-100):

```text
   94 / 100  Healthy       0 errors  ·  1 warning  ·  1 fixable
   12 files  ·  6 engines  ·  48ms

 Top findings
   #  Finding                 Rule                       Status
   1  console.log leftover    ai-slop/console-leftover   warn [auto]
```
{% endstep %}

{% step %}
### 4. Fix issues

Apply safe automated fixes for formatting, trivial comments, and unused imports:

```bash
raigal fix .
```

For structural issues that require generative edits, run `raigal agent .` to start an isolated worktree repair session with Claude Code, Cursor, or OpenCode.
{% endstep %}
{% endstepper %}
