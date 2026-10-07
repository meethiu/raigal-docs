---
description: Code quality and AI-generated pattern detection for engineering teams.
icon: shield-check
layout:
  width: default
  title:
    visible: true
  description:
    visible: true
  tableOfContents:
    visible: true
  outline:
    visible: true
  pagination:
    visible: true
  metadata:
    visible: true
  tags:
    visible: true
  actions:
    visible: true
  anchors:
    visible: true
---

# Raigal Overview

Raigal checks code for patterns introduced by AI coding tools, including verbose comments, unnecessary null checks, swallowed errors, and insecure calls. It combines formatting, linting, complexity checks, security rules, and pattern detection into one CLI command that returns a score from 0 to 100.

Used with the [Raigal Cloud Platform](https://app.raigal.dev) and the [Raigal GitHub App](https://github.com/apps/raigal-cloud), it runs checks on pull requests, updates status checks for branch protection, tracks issues on a remediation board, and can run automated repair sessions with coding agents.

{% hint style="success" %}
Raigal runs AST parsing, security audits, and pattern checks inside your local machine or CI runner. Source code is never sent to Raigal servers.
{% endhint %}

## What Raigal catches

Coding assistants such as Claude Code, Cursor, Copilot, and OpenCode speed up writing code, but they often leave behind specific defects:

* Comments that repeat obvious function signatures or variable names.
* Defensive checks on values already guaranteed by the type system.
* Empty `catch` blocks and unlogged errors that hide failures.
* Unescaped SQL queries, raw command injection, and `eval()` calls.
* Circular dependencies and references that bypass project layer rules.

Raigal runs locally through agent hooks, in CI on each commit, and on pull requests before merge.

## Components

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
      <td><strong>Raigal Cloud</strong></td>
      <td>Web dashboard at <code>app.raigal.dev</code> with a 4-stage remediation board and team scan history.</td>
      <td><a href="cloud-platform/dashboard.md">Cloud Dashboard</a></td>
    </tr>
    <tr>
      <td><strong>GitHub App</strong></td>
      <td>PR checks, commit status reporting, and branch protection enforcement.</td>
      <td><a href="automation-and-ci-cd/pull-requests.md">GitHub App Setup</a></td>
    </tr>
    <tr>
      <td><strong>Scoring engine (0-100)</strong></td>
      <td>Density-weighted score that normalizes findings by lines of code.</td>
      <td><a href="core-concepts/scoring.md">Scoring Engine</a></td>
    </tr>
    <tr>
      <td><strong>CLI and agent hooks</strong></td>
      <td>Pre-commit checks, safe autofixes (<code>raigal fix</code>), and real-time interception for coding agents.</td>
      <td><a href="cli-reference/commands.md">CLI Reference</a></td>
    </tr>
    <tr>
      <td><strong>Agent repairs</strong></td>
      <td>Isolated git worktrees where local coding agents fix issues and generate pull requests.</td>
      <td><a href="automation-and-ci-cd/agent-repairs.md">Agent Repairs</a></td>
    </tr>
    <tr>
      <td><strong>GitHub Actions OIDC</strong></td>
      <td>Tokenless authentication for CI runners using GitHub OIDC tokens instead of static secrets.</td>
      <td><a href="core-concepts/security.md">Security Model</a></td>
    </tr>
    <tr>
      <td><strong>Raigal MCP</strong></td>
      <td>Model Context Protocol server connecting Raigal tools to Claude Code, Cursor, Windsurf, Cline, and Antigravity.</td>
      <td><a href="automation-and-ci-cd/mcp-server.md">MCP Server</a></td>
    </tr>
  </tbody>
</table>

## Getting started

{% stepper %}
{% step %}
### 1. Sign in to Raigal Cloud

Create an organization workspace at [app.raigal.dev](https://app.raigal.dev) and invite your team. [Organization Onboarding Guide](getting-started/first-client-setup.md)
{% endstep %}

{% step %}
### 2. Install the Raigal GitHub App

Connect your repositories through the [Raigal GitHub App](https://github.com/apps/raigal-cloud) to run status checks on pull requests. [Pull Request Quality Gate](automation-and-ci-cd/pull-requests.md)
{% endstep %}

{% step %}
### 3. Add the CI workflow

Add a GitHub Actions workflow to check pull requests and send scores to your dashboard. [CI/CD Pipelines & GitHub Actions](automation-and-ci-cd/ci-cd.md)
{% endstep %}

{% step %}
### 4. Set up local tools and hooks

Install the CLI (`raigal scan`, `raigal fix`) and set up interception hooks for Claude Code or Cursor. [Quickstart Guide](getting-started/quickstart.md)
{% endstep %}
{% endstepper %}
