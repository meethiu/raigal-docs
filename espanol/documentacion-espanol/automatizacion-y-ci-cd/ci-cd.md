---
description: "Integración de Raigal en GitHub Actions, GitLab CI y pipelines de despliegue."
icon: route
---

# Pipelines de CI/CD y GitHub Actions

Configure Raigal para evaluar pull requests en CI y enviar las puntuaciones a su panel de control.

## Variables de entorno

Configure estas variables en su entorno de CI:

| Variable | Requerida | Descripción | Ejemplo |
|---|---|---|---|
| `RAIGAL_TOKEN` | Sí* | Clave de API de la organización (`rgl_live_...`) | `rgl_live_8f3a9b...` |
| `RAIGAL_API_URL` | Opcional | Dirección del backend (por defecto `https://app.raigal.dev`) | `https://app.raigal.dev` |

*Nota: En flujos de GitHub Actions con `permissions: id-token: write`, no es necesario definir claves fijas porque Raigal autentica al runner directamente mediante OIDC de GitHub.*

## GitHub Actions

### 1. Validación reforzada de pull requests

Cree `.github/workflows/raigal.yml` para analizar pull requests, enviar resultados al panel y generar anotaciones SARIF:

```yaml
name: Puerta de Calidad Raigal

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main, develop]

permissions:
  contents: read
  id-token: write        # Autenticación criptográfica OIDC sin contraseñas
  pull-requests: write   # Publicación de resumen de calidad en un único comentario
  security-events: write # Requerido para subir anotaciones SARIF a GitHub Code Scanning

jobs:
  gate:
    name: Filtro de Calidad y AI Slop
    runs-on: ubuntu-latest

    steps:
      - name: Descargar Código
        uses: actions/checkout@v4
        with:
          fetch-depth: 0 # Historial completo requerido para resolver la base del diff

      - name: Configurar Node.js
        uses: actions/setup-node@v4
        with:
          node-version: 22

      - name: Descargar Rama Base para Comparación
        if: github.event_name == 'pull_request'
        run: git fetch origin ${{ github.base_ref }} --depth=1

      - name: Ejecutar Puerta de Calidad Raigal
        env:
          RAIGAL_TOKEN: ${{ secrets.RAIGAL_TOKEN }}
          RAIGAL_API_URL: https://app.raigal.dev
          IS_FORK: ${{ github.event.pull_request.head.repo.full_name != github.repository }}
        run: |
          if [ "$IS_FORK" = "true" ]; then
            echo "PR desde bifurcación (fork): ejecutando en modo consultivo sin permisos de escritura"
            npx -y @methiu/raigal@0.2.2 ci --changes --base origin/${{ github.base_ref }} --human
          else
            npx -y @methiu/raigal@0.2.2 ci --changes --base origin/${{ github.base_ref || 'main' }} --report-pr --sarif-output raigal-results.sarif
          fi

      - name: Subir SARIF a GitHub Code Scanning
        if: always() && github.event.pull_request.head.repo.full_name == github.repository
        uses: github/codeql-action/upload-sarif@v3
        continue-on-error: true
        with:
          sarif_file: raigal-results.sarif
          category: raigal
```

{% hint style="info" %}
La subida de informes SARIF a GitHub Code Scanning en repositorios privados requiere que la cuenta empresarial tenga habilitado GitHub Advanced Security (GHAS). En repositorios públicos, el análisis de código se incluye sin coste adicional.
{% endhint %}

### 2. Sincronización de cierre y fusión

{% hint style="info" %}
Si utiliza la **GitHub App de Raigal**, este paso no es necesario. La app recibe los webhooks de cierre y fusión directamente de GitHub y actualiza el tablero de forma automática.
{% endhint %}

Para entornos de CI sin la GitHub App, sincronice el estado cuando un PR se fusione o se cierre:

```yaml
name: Ciclo de Vida de PRs Raigal

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

      - name: Sincronizar PR con Raigal Cloud
        env:
          RAIGAL_TOKEN: ${{ secrets.RAIGAL_TOKEN }}
          RAIGAL_API_URL: ${{ secrets.RAIGAL_API_URL || 'https://app.raigal.dev' }}
        run: |
          npx -y @methiu/raigal@0.2.2 report-pr-closed --pr ${{ github.event.pull_request.number }} ${{ github.event.pull_request.merged && '--merged' || '' }}
```

## GitLab CI

En `.gitlab-ci.yml`:

```yaml
raigal_gate:
  stage: test
  image: node:22
  variables:
    RAIGAL_TOKEN: $RAIGAL_TOKEN
    RAIGAL_API_URL: $RAIGAL_API_URL
  script:
    - npx -y @methiu/raigal@0.2.2 ci --human
```
