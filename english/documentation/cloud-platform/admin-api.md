---
description: "API endpoints for token management, repository metrics, and telemetry ingestion."
icon: key
---

# Organization API and token management

Raigal Cloud provides a REST API to query scores, manage tokens, ingest custom CI run data, and delete repositories.

## Authentication

Requests must include an organization API key (`rgl_live_...`) in the authorization header:

{% tabs %}
{% tab title="Bearer header" %}
```http
Authorization: Bearer rgl_live_your_api_key_here
```
{% endtab %}

{% tab title="API key header" %}
```http
x-api-key: rgl_live_your_api_key_here
```
{% endtab %}
{% endtabs %}

{% hint style="info" %}
Generate API keys at [app.raigal.dev/settings](https://app.raigal.dev/settings) under **API Tokens**.
{% endhint %}

## Endpoints

### 1. Identity check
Verify the active token and return organization details.

```http
GET /v1/whoami
```

Response:
```json
{
  "orgId": "org_2tQ...",
  "orgName": "Acme Engineering",
  "tier": "enterprise",
  "authenticated": true
}
```

### 2. Organization quality policy
Retrieve active quality thresholds configured for the organization.

```http
GET /v1/policy
```

Response:
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

### 3. List tracked repositories
List all active repositories reporting to the organization.

```http
GET /v1/runs/repos
```

Response:
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

### 4. Remove a repository
Delete a repository and its associated run history, pull request records, and findings.

```http
DELETE /v1/runs/repos/:repoId
```

`:repoId` can be the repository UUID or the URL-encoded repository name (such as `acme%2Fpayment-service`).

Response:
```json
{
  "ok": true,
  "message": "Repository acme/payment-service and all associated runs removed successfully."
}
```

{% hint style="warning" %}
This operation is permanent. You can also delete repositories from the **Settings** page in the web dashboard.
{% endhint %}

### 5. Fetch board data
Retrieve board columns and cards formatted as JSON.

```http
GET /v1/runs/board?repoId=acme/payment-service&branch=main
```

Query parameters:
- `repoId`: (optional) filter by repository name.
- `branch`: (optional) filter by git branch (defaults to primary branch).

Response:
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

### 6. Ingest custom CI run
For CI systems other than GitHub Actions (Jenkins, GitLab CI, CircleCI), submit run results directly:

```http
POST /v1/runs
Content-Type: application/json
```

Request body:
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

## Token management

Tokens can be created and revoked in the web dashboard or via the tokens API:

### Create API token
```http
POST /api/tokens
Content-Type: application/json

{
  "name": "Production Deploy Runner"
}
```

Response:
```json
{
  "token": "rgl_live_8f3a9b1c7e2d...",
  "prefix": "rgl_live_8f3a",
  "name": "Production Deploy Runner",
  "created_at": "2026-09-24T18:00:00Z"
}
```

{% hint style="danger" %}
Store your token when created. The full token string is shown once. Raigal Cloud stores a SHA-256 hash and cannot recover lost tokens.
{% endhint %}

### Revoke API token
```http
DELETE /api/tokens/:tokenId
```

Revoking a token disables it immediately for future requests.
