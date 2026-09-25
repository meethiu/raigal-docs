---
description: "Enterprise architecture, zero-code exfiltration guarantee, hybrid GitHub App model, and security controls in Raigal Cloud."
icon: network-wired
---

# Platform Architecture & Security

Raigal Cloud (`https://app.raigal.dev`) serves as the centralized code-quality control plane for enterprise engineering teams. It coordinates automated Pull Request quality gates, remediates AI slop patterns, powers developer dashboards, and issues cryptographic entitlement leases while strictly adhering to a **Zero Code Exfiltration** security model.

---

## High-Level Topology

```mermaid
graph TD
    subgraph Client Compute Environments
        Dev["Local Workstation: raigal scan / fix"]
        Agent["Autonomous Agent: raigal agent --pr"]
        CI["GitHub Actions / CI Runners"]
    end

    subgraph GitHub Cloud
        GHApp["Official GitHub App (raigal-cloud)"]
        GHChecks["GitHub Checks API & Status"]
    end

    subgraph Raigal Cloud Control Plane (app.raigal.dev)
        Gateway["REST API Gateway (HTTPS / TLS 1.3)"]
        Auth["Clerk Identity & GitHub Organization Sync"]
        OIDC["Zero-Token GitHub Actions OIDC Verifier"]
        CheckManager["Check Run & Status Orchestrator"]
        LeaseSigner["Ed25519 Cryptographic Lease Signer"]
        Store[("Encrypted Multi-Tenant Metadata Store")]
    end

    subgraph Customer Management UI
        Board["4-Stage Remediation Kanban"]
        Drawers["Contextual Inspection Drawers"]
        Telemetry["Execution History & Diff Audits"]
        Settings["Token & Repository Governance"]
    end

    Dev -- "raigal login (OAuth Device Flow)" --> Auth
    Dev -- "POST /v1/runs (Metadata & Score)" --> Gateway
    Agent -- "raigal agent --pr (PR & Diff Stats)" --> Gateway
    CI -- "X-GitHub-OIDC or RAIGAL_TOKEN" --> Gateway
    GHApp -- "Webhook (PR & Installation Events)" --> Gateway

    Gateway --> Store
    Gateway --> LeaseSigner
    Gateway --> CheckManager
    CheckManager -- "Create / Resolve Checks" --> GHChecks

    Store --> Board
    Store --> Drawers
    Store --> Telemetry
    Auth --> Settings
```

---

## The Zero Code Exfiltration Guarantee

Enterprise engineering teams require absolute certainty that proprietary intellectual property, trade secrets, and source code remain completely confidential.

{% hint style="success" %}
**Zero Code Exfiltration:** Raigal is explicitly architected so that **no customer source code is ever transmitted to, processed by, or stored on Raigal Cloud servers.**
{% endhint %}

### How Compute vs. Control Plane Separation Works:
1. **Local & Runner Execution**: All AST parsing, linters (oxlint, ruff, biome, etc.), complexity analyzers, and AI slop detection algorithms execute **entirely inside your compute environment** (your local terminal or your GitHub Actions runner).
2. **Metadata-Only Ingestion**: When a scan completes, only anonymized metadata is sent to Raigal Cloud:
   - Final Quality Score (0–100) and category breakdowns.
   - Diagnostic rule IDs (e.g. `ai-slop/trivial-comment`, `security/no-eval`).
   - Relative file paths and line numbers of flagged findings.
   - Diff statistics (number of files changed, lines added, lines deleted).
3. **In-Flight Secret Scrubbing**: Before any telemetry payload leaves your workstation or runner, Raigal's CLI scanner redacts authorization headers, tokens, private keys, and credential strings.

---

## Hybrid GitHub App Integration

Raigal integrates natively with GitHub via the official **`raigal-cloud`** GitHub App (`https://github.com/apps/raigal-cloud`):

- **Event-Driven Check Runs**: When a developer opens or updates a Pull Request, GitHub notifies Raigal Cloud via secure webhooks. Raigal immediately creates a pending `"Raigal Quality Gate"` check run on the commit head SHA.
- **Fail-Open Check Resolution**: When your CI runner completes the quality evaluation, it submits the score and metadata back to Raigal Cloud. Raigal instantly marks the check as passed or failed, resolving branch protection constraints.
- **Automated Lifecycle Sync**: Merges and closures are reported automatically by GitHub's webhook bus, ensuring your Kanban board always reflects real-time repository state without custom cleanup scripts.
- **Orphan Sweeper**: If a CI job crashes or is cancelled upstream, Raigal Cloud sweeps orphaned pending checks after 20 minutes to prevent PRs from being blocked indefinitely.

---

## Authentication & Access Control

### 1. Multi-Tenant Organization Isolation
Raigal Cloud enforces multi-tenant organization boundaries powered by Clerk enterprise identity. Every repository, run, finding, and API token is strictly scoped to your organization. Cross-tenant access is cryptographically and logically prohibited.

### 2. Zero-Token GitHub Actions OIDC
For enterprise CI pipelines, Raigal eliminates the risks associated with static API keys:
- Your GitHub Actions runner requests a short-lived OIDC JSON Web Token (JWT) from GitHub's OpenID Connect provider (`token.actions.githubusercontent.com`).
- The runner transmits the token to Raigal Cloud via the `X-GitHub-OIDC` header.
- Raigal validates the token's cryptographic signature and matches the repository owner against your organization's verified GitHub account.
- If verified, the runner is granted ephemeral ingestion rights for that specific run.

### 3. Developer CLI Authentication (RFC 8628)
Developers authenticate their local workstations seamlessly via `raigal login`. This initiates an RFC 8628 OAuth 2.0 Device Authorization flow, opening a browser verification page without requiring engineers to manually copy secret tokens.

### 4. Cryptographic Offline Leases (Ed25519)
To prevent network latency from slowing down developer workflows and allow full offline operation:
- Upon authentication, Raigal Cloud issues a cryptographically signed Ed25519 entitlement token.
- The local CLI verifies this token offline using a public key bundled into the binary.
- Leases remain valid for 72 hours, allowing developers to scan and fix code while travelling or in air-gapped environments without continuous internet connectivity.

---

## Data Governance & Repository Lifecycle

Raigal Cloud provides administrators with complete control over their repository footprint:

- **Hashed Secrets**: API keys are hashed using SHA-256 before storage; raw tokens are never persisted in databases.
- **On-Demand Repository Purge**: Administrators can remove tracked repositories at any time from **Settings** > **Repositories**. Deleting a repository cascades through the database, permanently purging all associated historical runs, Pull Request cards, and diagnostic findings.
