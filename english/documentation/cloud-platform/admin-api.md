---
description: "Organization API endpoints, authentication, repository management, and token rotation for Raigal Cloud clients."
icon: key
---

# Organization API & Token Management

Raigal Cloud provides a REST API that enables customer organizations to manage API tokens, query repository quality scores, ingest custom CI telemetry, and programmatically purge repository data.

---

## Authentication

All API requests from scripts, automated pipelines, or custom integrations must authenticate using an Organization API Key (`rgl_live_...`):

{% tabs %}
{% tab title="Bearer Header" %}
```http
Authorization: Bearer rgl_live_your_api_key_here
```
{% endtab %}

{% tab title="API Key Header" %}
```http
x-api-key: rgl_live_your_api_key_here
```
{% endtab %}
{% endtabs %}

{% hint style="info" %}
Generate API keys from your dashboard at [**https://app.raigal.dev/settings**](https://app.raigal.dev/settings) under the **API Tokens** section.
{% endhint %}

---

## Client Endpoints Reference

### 1. Identity & Organization Introspection
Verify your active token and inspect your organization identity.

```http
GET /v1/whoami
```

**Response:**
```json
{
  "orgId": "org_2tQ...",
  "orgName": "Acme Engineering",
  "tier": "enterprise",
  "authenticated": true
}
```

---

### 2. Organization Quality Policy
Retrieve the active quality policy and enforcement thresholds configured for your organization.

```http
GET /v1/policy
```

**Response:**
```json
{
  "minScore": 80,
  "failOnSeverity": "error",
  "blockedRules": [
    "ai-slop/trivial-comment",
    "security/no-eval",
    "quality/silent-catch"
  ],
  "engines": {
    "format": true,
    "lint": true,
    "quality": true,
    "aiSlop": true,
    "security": true
  }
}
```

---

### 3. List Tracked Repositories
Retrieve a list of all active repositories reporting to your organization.

```http
GET /v1/runs/repos
```

**Response:**
```json
{
  "repositories": [
    {
      "id": "a9b8c7d6-...",
      "repoId": "acme/payment-service",
      "latestScore": 92,
      "lastRunAt": "2026-09-24T14:32:00Z",
      "activeBranches": ["main", "feat/apple-pay"]
    }
  ]
}
```

---

### 4. Remove a Repository
Permanently delete a repository and cascade purge all of its historical runs, PR cards, and diagnostic findings from your organization.

```http
DELETE /v1/runs/repos/:repoId
```

* `:repoId` can be the repository UUID or URL-encoded repository name (e.g. `acme%2Fpayment-service`).

**Response:**
```json
{
  "ok": true,
  "message": "Repository acme/payment-service and all associated runs removed successfully."
}
```

{% hint style="warning" %}
This operation is irreversible. You can also perform this action visually from the **Settings** view in the Web UI.
{% endhint %}

---

### 5. Fetch Remediation Kanban Board Data
Retrieve the active four-stage Kanban board data formatted as JSON. Useful for feeding internal metrics dashboards, executive summaries, or custom Slack bots.

```http
GET /v1/runs/board?repoId=acme/payment-service&branch=main
```

**Query Parameters:**
- `repoId`: (Optional) Filter cards by repository name.
- `branch`: (Optional) Filter cards by Git branch (defaults to primary branch).

**Response:**
```json
{
  "mediumPriority": [],
  "highRisk": [],
  "pullRequests": [
    {
      "prNumber": 142,
      "title": "feat: introduce stripe webhooks",
      "author": "octocat",
      "status": "passed",
      "score": 94,
      "headSha": "3b2a1c...",
      "diff": { "filesChanged": 6, "additions": 140, "deletions": 12 }
    }
  ],
  "agentRemediations": []
}
```

---

### 6. Ingest Telemetry Run (Custom CI)
If you run CI in an environment other than GitHub Actions (e.g., Jenkins, GitLab CI, CircleCI), ingest your run results directly via the API:

```http
POST /v1/runs
Content-Type: application/json
```

**Request Body:**
```json
{
  "repoId": "acme/backend-api",
  "branch": "main",
  "commitSha": "e4f5a6b7...",
  "score": 88,
  "exitCode": 0,
  "engines": ["format", "lint", "aiSlop", "security"],
  "findingsCount": 2,
  "findings": [
    {
      "ruleId": "ai-slop/redundant-type-assertion",
      "severity": "warn",
      "file": "src/auth/jwt.ts",
      "line": 45,
      "message": "Unnecessary type assertion detected."
    }
  ]
}
```

---

## Token Lifecycle Management

Tokens can be created and rotated via the Web Dashboard at `https://app.raigal.dev/settings` or programmatically via the tokens endpoint:

### Create API Token
```http
POST /api/tokens
Content-Type: application/json

{
  "name": "Production Deploy Runner"
}
```

**Response:**
```json
{
  "token": "rgl_live_8f3a9b1c7e2d...",
  "prefix": "rgl_live_8f3a",
  "name": "Production Deploy Runner",
  "created_at": "2026-09-24T18:00:00Z"
}
```

{% hint style="danger" %}
Store your token immediately. The raw token value is returned **only once** at creation. Raigal Cloud encrypts and hashes tokens using SHA-256 and cannot recover lost keys.
{% endhint %}

### Revoke API Token
```http
DELETE /api/tokens/:tokenId
```

Revokes the specified key immediately, rejecting all subsequent requests presenting that token.
