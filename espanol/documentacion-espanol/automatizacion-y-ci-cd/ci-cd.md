---
description: "Integre Raigal en GitHub Actions, GitLab CI y Bitbucket Pipelines."
icon: route
---

# Pipelines CI/CD y puertas de calidad automatizadas

Configure Raigal para evaluar pull requests en CI, aplicar puertas de calidad y enviar métricas a su panel de control.

Al ejecutar `raigal init`, el CLI detecta automáticamente su proveedor de CI y genera el flujo adecuado sin conjeturas.

## Variables de entorno

Proporcione estas variables en su entorno de CI:

| Variable | Requerida | Descripción | Ejemplo |
|---|---|---|---|
| `RAIGAL_TOKEN` | Sí* | Clave de API de la organización (`rgl_live_...`) | `rgl_live_8f3a9b...` |
| `RAIGAL_ORG_ID` | Opcional | ID de organización (`org_...`), requerido sólo si el propietario del repositorio pertenece a múltiples organizaciones de Raigal | `org_2tQ...` |
| `RAIGAL_API_URL` | Opcional | URL del backend (por defecto `https://app.raigal.dev`) | `https://app.raigal.dev` |

*Nota: En GitHub Actions configurado con `permissions: id-token: write`, los tokens estáticos no son obligatorios porque Raigal valida el runner mediante OIDC criptográfico sin token.*

## GitHub Actions

### 1. Control de calidad de pull requests

Cree `.github/workflows/raigal.yml` para evaluar pull requests, notificar al dashboard y generar anotaciones SARIF opcionales:

```yaml
name: Raigal Quality Gate

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main, develop]

permissions:
  contents: read
  id-token: write        # Habilita autenticación OIDC criptográfica sin token
  security-events: write # Requerido para subir SARIF a GitHub Code Scanning

jobs:
  gate:
    name: Code Quality & Slop Gate
    runs-on: ubuntu-latest

    steps:
      - name: Checkout Code
        uses: actions/checkout@v4
        with:
          fetch-depth: 0 # Historial completo necesario para calcular diffs con la rama base

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: 22

      - name: Run Raigal Quality Gate
        env:
          RAIGAL_TOKEN: ${{ secrets.RAIGAL_TOKEN }}
          # RAIGAL_ORG_ID: ${{ vars.RAIGAL_ORG_ID }} # Requerido si el propietario pertenece a varias orgs
        run: |
          if [ "${{ github.event_name }}" = "pull_request" ]; then
            npx --yes @methiu/raigal@latest ci --changes --base origin/${{ github.base_ref }}
          else
            npx --yes @methiu/raigal@latest ci .
          fi

      - name: Generate SARIF Security Report
        if: always()
        run: npx --yes @methiu/raigal@latest scan . --sarif > raigal-results.sarif

      - name: Upload SARIF to GitHub Code Scanning
        if: always() && github.event.pull_request.head.repo.full_name == github.repository
        uses: github/codeql-action/upload-sarif@v3
        continue-on-error: true
        with:
          sarif_file: raigal-results.sarif
          category: raigal
```

{% hint style="info" %}
Subir informes SARIF a GitHub Code Scanning en repositorios privados requiere GitHub Advanced Security (GHAS). En repositorios públicos es gratuito.
{% endhint %}

### 2. Sincronización al fusionar o cerrar PRs

{% hint style="info" %}
Si utiliza la **GitHub App de Raigal**, este paso es innecesario. La aplicación recibe los webhooks de GitHub y actualiza el estado automáticamente.
{% endhint %}

Para entornos sin la GitHub App, sincronice el estado al fusionar o cerrar:

```yaml
name: Raigal PR Lifecycle

on:
  pull_request:
    types: [closed]

jobs:
  sync:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 22

      - name: Synchronize PR with Raigal Cloud
        env:
          RAIGAL_TOKEN: ${{ secrets.RAIGAL_TOKEN }}
        run: |
          npx --yes @methiu/raigal@latest report-pr-closed --pr ${{ github.event.pull_request.number }} ${{ github.event.pull_request.merged && '--merged' || '' }}
```

## GitLab CI

En `.gitlab-ci.yml` (autogenerado por `raigal init`):

```yaml
raigal_quality_gate:
  image: node:24
  stage: test
  script:
    - if [ -n "$CI_MERGE_REQUEST_IID" ]; then npx --yes @methiu/raigal@latest ci --changes --base "$CI_MERGE_REQUEST_DIFF_BASE_SHA"; else npx --yes @methiu/raigal@latest ci .; fi
  variables:
    RAIGAL_TOKEN: $RAIGAL_TOKEN
    # RAIGAL_ORG_ID: $RAIGAL_ORG_ID # Requerido si el propietario pertenece a varias orgs
  rules:
    - if: $CI_PIPELINE_SOURCE == "merge_request_event"
    - if: $CI_COMMIT_BRANCH == $CI_DEFAULT_BRANCH
```

Configure `RAIGAL_TOKEN` en **Settings > CI/CD > Variables**.

## Bitbucket Pipelines

En `bitbucket-pipelines.yml` (autogenerado por `raigal init`):

```yaml
pipelines:
  default:
    - step:
        name: raigal quality gate
        image: node:24
        script:
          - npx --yes @methiu/raigal@latest ci .
        secrets:
          - RAIGAL_TOKEN
          # - RAIGAL_ORG_ID
  pull-requests:
    "**":
      - step:
          name: raigal quality gate
          image: node:24
          clone:
            depth: full   # Se requiere historial completo para diffs
          script:
            - git fetch origin "$BITBUCKET_PR_DESTINATION_BRANCH"
            - npx --yes @methiu/raigal@latest ci --changes --base FETCH_HEAD
          secrets:
            - RAIGAL_TOKEN
            # - RAIGAL_ORG_ID
```

Configure `RAIGAL_TOKEN` en **Repository settings > Repository variables**.
