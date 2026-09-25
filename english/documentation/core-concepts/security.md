---
description: "Security model, credential storage, environment variables, and offline leases."
icon: lock
---

# Security model and credentials

Raigal stores local credentials securely, scrubs secrets before sending metrics, and supports tokenless authentication through GitHub Actions OIDC.

## Credential storage

On local machines, credentials and offline leases are stored in your home directory:

- Path: `~/.raigal/credentials.json`
- Permissions: `0600` (read and write for the file owner only)
- File format:
  ```json
  {
    "token": "rgl_sess_...",
    "org": { "id": "org_123", "name": "Acme Corp" },
    "user": { "id": "user_456", "email": "dev@acme.com" },
    "created_at": "2026-09-23T12:00:00.000Z"
  }
  ```

## Setting `RAIGAL_TOKEN` and `RAIGAL_API_URL`

In CI pipelines, Docker containers, or scripts, pass authentication using environment variables:

- `RAIGAL_TOKEN`: your organization's API token (`rgl_live_...`). This overrides `~/.raigal/credentials.json`.
- `RAIGAL_API_URL`: custom backend URL (defaults to `https://app.raigal.dev`).

{% tabs %}
{% tab title="GitHub Actions" %}
```yaml
- name: Run Raigal Scan
  env:
    RAIGAL_TOKEN: ${{ secrets.RAIGAL_TOKEN }}
    RAIGAL_API_URL: ${{ secrets.RAIGAL_API_URL || 'https://app.raigal.dev' }}
  run: npx @methiu/raigal scan .
```
{% endtab %}

{% tab title="Terminal (macOS/Linux)" %}
```bash
export RAIGAL_TOKEN="rgl_live_your_token_here"
export RAIGAL_API_URL="https://app.raigal.dev"
npx @methiu/raigal whoami
```
{% endtab %}

{% tab title="Windows PowerShell" %}
```powershell
$env:RAIGAL_TOKEN="rgl_live_your_token_here"
$env:RAIGAL_API_URL="https://app.raigal.dev"
npx @methiu/raigal whoami
```
{% endtab %}
{% endtabs %}

## Metric scrubbing and data handling

Before telemetry is sent to the cloud:
1. The CLI redacts authorization headers, tokens, private keys, and credential patterns.
2. Only aggregate counts, rule IDs, file paths, and score deltas are uploaded. Source code is never sent to Raigal Cloud.
3. The backend stores run records in PostgreSQL tables (`telemetry_runs`, `pull_requests`) using idempotent upserts.

## GitHub Actions OIDC authentication

To run in GitHub Actions without static repository secrets:

1. Add `permissions: id-token: write` to your workflow job.
2. The GitHub runner mints an OIDC token signed by GitHub.
3. Raigal passes this token in the `X-GitHub-OIDC` header to `/api/v1/telemetry/runs`.
4. Raigal Cloud validates the signature against `https://token.actions.githubusercontent.com` and confirms that the repository owner is registered to your organization.

## Offline leases (Ed25519)

Raigal does not contact the server on every scan:

1. When you run `raigal login` or supply `RAIGAL_TOKEN`, the server signs a lease token using an Ed25519 private key.
2. The CLI verifies this signature offline using a public key bundled in the binary (`src/cloud/keys.ts`).
3. The lease is saved to `~/.raigal/entitlement.json` for up to 72 hours, allowing scans to run without an active internet connection.
