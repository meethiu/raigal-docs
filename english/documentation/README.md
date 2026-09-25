---
description: "Raigal is the enterprise code-quality and anti-AI-slop platform designed for modern engineering teams and coding agents."
icon: shield-check
layout:
  width: default
  tableOfContents:
    visible: true
  pagination:
    visible: true
---

# Welcome to Raigal

Raigal is an enterprise-grade code-quality platform and CLI engine that prevents the fragile, bloated, and insecure patterns AI coding tools leave behind. It unifies formatting, linting, complexity checks, security audits, and specialized AI-slop detection behind a single command, returning an actionable score from 0 to 100.

Paired with the **[Raigal Cloud Platform](https://app.raigal.dev)** and the official **[Raigal GitHub App](https://github.com/apps/raigal-cloud)**, engineering organizations get automated Pull Request quality gating, branch protection enforcement, real-time remediation kanbans, and autonomous AI repair agents.

{% hint style="success" %}
**Zero Code Exfiltration:** Raigal runs all AST parsing, security audits, and slop detection 100% inside your developer environment or CI runner. Proprietary source code never leaves your infrastructure.
{% endhint %}

## Why Modern Teams Use Raigal

Generative AI coding assistants (Claude Code, Cursor, Copilot, Codex, OpenCode) accelerate development, but frequently introduce:
- **Trivial comments & documentation fluff:** Verbose explanations of obvious code or repeated signatures.
- **Defensive hallucinations:** Pointless polyfills, redundant null-checks, and dead type assertions.
- **Silent failure antipatterns:** Empty `catch` blocks or swallowed errors with silent recovery.
- **Security vulnerabilities:** Inadvertent use of `eval()`, unescaped raw SQL, or unsafe script injections.
- **Architectural drift:** Bypassing clean architecture boundaries and accumulating untracked tech debt.

Raigal provides an automated quality gate across every phase of your development lifecycle: locally via agent hooks, in CI pipelines on every commit, and at the Pull Request review gate.

## The Raigal Platform Ecosystem

<table data-view="cards">
  <thead>
    <tr>
      <th>Component</th>
      <th>Description</th>
      <th>Link</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><strong>Raigal Cloud Platform</strong></td>
      <td>Centralized SaaS dashboard at <code>app.raigal.dev</code> featuring a 4-stage Remediation Kanban and team telemetry.</td>
      <td><a href="cloud-platform/dashboard.md">Cloud Dashboard</a></td>
    </tr>
    <tr>
      <td><strong>Official GitHub App</strong></td>
      <td>Automated PR quality gate, native check runs, and branch protection enforcement.</td>
      <td><a href="automation/pull-requests.md">GitHub App Setup</a></td>
    </tr>
    <tr>
      <td><strong>Deterministic Scoring (0–100)</strong></td>
      <td>Calibrated, density-aware scoring engine that measures true code health without noise.</td>
      <td><a href="core-concepts/scoring.md">Scoring Engine</a></td>
    </tr>
    <tr>
      <td><strong>Local CLI & Agent Hooks</strong></td>
      <td>Pre-commit verification, auto-fixing (<code>raigal fix</code>), and real-time interception for coding agents.</td>
      <td><a href="cli-reference/commands.md">CLI Reference</a></td>
    </tr>
    <tr>
      <td><strong>Autonomous Agent Repairs</strong></td>
      <td>Isolated git worktrees that invoke LLM coding agents to fix findings and generate ready-to-merge PRs.</td>
      <td><a href="automation/agent-repairs.md">Agent Repairs</a></td>
    </tr>
    <tr>
      <td><strong>Zero-Secret Enterprise OIDC</strong></td>
      <td>GitHub Actions OIDC federation eliminating the need for long-lived static tokens in CI.</td>
      <td><a href="core-concepts/security.md">Security Model</a></td>
    </tr>
  </tbody>
</table>

## Getting Started in 4 Steps

{% stepper %}
{% step %}
### 1. Sign in to Raigal Cloud
Create your organization workspace at **[app.raigal.dev](https://app.raigal.dev)** and invite your team members.
[Organization Onboarding Guide](getting-started/first-client-setup.md)
{% endstep %}

{% step %}
### 2. Install the Raigal GitHub App
Connect your organization's repositories with the official **[Raigal GitHub App](https://github.com/apps/raigal-cloud)** to enable automatic PR status checks.
[Pull Request Quality Gate](automation/pull-requests.md)
{% endstep %}

{% step %}
### 3. Add the CI Quality Gate
Add a 20-line GitHub Actions workflow to gate pull requests and track deterministic quality scores on your Kanban board.
[CI/CD Pipelines & GitHub Actions](automation/ci-cd.md)
{% endstep %}

{% step %}
### 4. Enable Local Developer Scans & Hooks
Equip developers with the CLI (`raigal scan`, `raigal fix`) and install agent interception hooks for Claude Code and Cursor.
[Quickstart Guide](getting-started/quickstart.md)
{% endstep %}
{% endstepper %}
