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
  run: npx -y @methiu/raigal@0.2.2 ci --changes --base origin/${{ github.base_ref || 'main' }}
```
{% endtab %}

{% tab title="Terminal (macOS/Linux)" %}
```bash
export RAIGAL_TOKEN="rgl_live_your_token_here"
export RAIGAL_API_URL="https://app.raigal.dev"
raigal whoami
```
{% endtab %}

{% tab title="Windows PowerShell" %}
```powershell
$env:RAIGAL_TOKEN="rgl_live_your_token_here"
$env:RAIGAL_API_URL="https://app.raigal.dev"
raigal whoami
```
{% endtab %}
{% endtabs %}

## Metric scrubbing and data handling

Before telemetry or PR summaries are sent to the cloud:
1. Local scans execute 100% in-process. Source code files are never uploaded to Raigal Cloud.
2. `maskSecrets` automatically replaces high-entropy credentials, private keys, database URLs, and API tokens with redaction placeholders (`[REDACTED_SECRET]`).
3. When using `--report-pr`, a git patch diff preview (up to 20 KB) is transmitted after in-memory secret masking to populate the dashboard diff viewer. Diff uploads can be disabled organization-wide.
4. Autonomous repair (`raigal agent`) passes relevant source snippets to the customer's configured LLM provider CLI (Claude Code, OpenCode, Codex).

## GitHub Actions OIDC authentication

To run in GitHub Actions without static repository secrets:

1. Add `permissions: id-token: write` to your workflow job.
2. The GitHub runner mints a short-lived OIDC token signed by GitHub.
3. Raigal passes this token in the `X-GitHub-OIDC` header to `/api/v1/telemetry/runs`.
4. Raigal Cloud validates the signature against `https://token.actions.githubusercontent.com` and confirms that the repository owner is registered to your organization.

## Offline operation and air-gap options

Raigal does not contact the server on every scan:

1. **Offline-Tolerant SaaS (72-hour lease)**: When you authenticate, Raigal Cloud issues a signed Ed25519 lease token. The CLI verifies this signature offline using a public key bundled in the binary. Scans run without network access for up to 72 hours.
2. **True Air-Gapped Environments (Banking / Defense)**:
   - **Self-Hosted Raigal Cloud**: Deploy the official Docker container in your private VPC with zero public internet egress.
   - **Static Offline License Key**: Statically signed JWT valid for 1 to 3 years, verified locally against the public key with no outbound network calls.
