---
description: "Architecture, data handling, GitHub App integration, and security controls in Raigal Cloud."
icon: network-wired
---

# Platform architecture and security

Raigal Cloud (`https://app.raigal.dev`) is the web application and coordination service for Raigal. It manages pull request status checks, tracks findings on remediation boards, authenticates developers, and signs offline licenses.

## System topology

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

## Source code handling

Customer source code is never sent to, processed by, or stored on Raigal Cloud servers.

Separation between client compute and cloud control plane:
1. Local and runner execution: AST parsing, linters (oxlint, ruff, biome), complexity checks, and pattern rules run entirely inside your workstation or CI runner.
2. Metadata-only reporting: when a scan finishes, only diagnostic summaries are sent to the cloud:
   - Quality score (0-100) and category breakdowns
   - Rule IDs (such as `ai-slop/trivial-comment` and `security/no-eval`)
   - Relative file paths and line numbers
   - Diff metrics (files changed, lines added, lines deleted)
3. Secret scrubbing: the CLI strips authorization headers, access tokens, private keys, and common credential patterns before sending payloads.

## GitHub App integration

Raigal connects with GitHub through the official `raigal-cloud` app:

- Webhook status checks: when a pull request opens or updates, GitHub notifies Raigal Cloud, which registers a pending `"Raigal Quality Gate"` check on the commit SHA.
- Check resolution: after the CI runner finishes the scan, it reports the score back to Raigal Cloud. Raigal then marks the check as passed or failed, resolving required branch protection rules.
- Lifecycle events: PR merges and closures are delivered via GitHub webhooks, keeping board status current without custom cleanup scripts.
- Orphan cleanup: if a CI job is cancelled or interrupted, Raigal Cloud clears pending checks after 20 minutes to prevent PRs from being blocked indefinitely.

## Authentication and access control

### Multi-tenant organization isolation
Raigal Cloud isolates tenant data using Clerk identity. Repositories, scan records, findings, and tokens are partitioned by organization ID.

### GitHub Actions OIDC
For CI workflows, runners authenticate using ephemeral GitHub tokens:
- The runner requests a short-lived OIDC token from `token.actions.githubusercontent.com`.
- The runner sends the token in the `X-GitHub-OIDC` header.
- Raigal validates the token signature and verifies that the repository owner matches your organization.
- Once verified, the runner is granted one-time access to record that run.

### Developer CLI login (RFC 8628)
Developers log in from the terminal with `raigal login`. This uses the OAuth 2.0 Device Authorization flow (RFC 8628), opening a browser window to approve the session.

### Offline leases (Ed25519)
To allow scanning without continuous network access:
- When you authenticate, Raigal Cloud issues a signed Ed25519 lease token.
- The CLI verifies the signature locally with a public key embedded in the binary.
- Leases remain valid for 72 hours, enabling offline scans in air-gapped environments or while travelling.

## Repository management

Administrators manage repository data in **Settings** > **Repositories**:
- Tokens are hashed with SHA-256 before storage; raw tokens are never written to the database.
- Deleting a repository cascades through the database, permanently removing associated historical runs, pull request cards, and findings.
