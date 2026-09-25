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

*Nota: En flujos de GitHub Actions con `permissions: id-token: write`, no es necesario definir `RAIGAL_TOKEN` porque Raigal autentica al runner mediante OIDC.*

## GitHub Actions

### 1. Validación de pull requests

Cree `.github/workflows/raigal.yml` para analizar pull requests y reportar el resultado:

```yaml
name: Puerta de Calidad Raigal

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main, develop]

jobs:
  gate:
    name: Filtro de Calidad y AI Slop
    runs-on: ubuntu-latest
    permissions:
      contents: read
      id-token: write

    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0
      - uses: actions/setup-node@v4
        with:
          node-version: 22

      - name: Ejecutar Puerta de Calidad Raigal
        env:
          RAIGAL_TOKEN: ${{ secrets.RAIGAL_TOKEN }}
          RAIGAL_API_URL: ${{ secrets.RAIGAL_API_URL }}
        run: |
          npx @methiu/raigal ci --changes --base origin/${{ github.base_ref || 'main' }} --human
```

{% hint style="tip" %}
Al configurar `permissions: id-token: write`, el runner genera un token OIDC temporal. Raigal lo envía mediante `X-GitHub-OIDC`, evitando la necesidad de guardar claves estáticas en los secretos del repositorio.
{% endhint %}

### 2. Sincronización de cierre y fusión

{% hint style="info" %}
Si utiliza la **GitHub App de Raigal**, este paso es innecesario. La app recibe los webhooks de cierre y fusión directamente de GitHub y actualiza el tablero de forma automática.
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
          npx @methiu/raigal report-pr-closed --pr ${{ github.event.pull_request.number }} ${{ github.event.pull_request.merged && '--merged' || '' }}
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
    - npx @methiu/raigal ci --human
```

## Anotaciones en pull requests y SARIF

Para ver los hallazgos directamente en la pestaña de seguridad de GitHub:

```bash
npx @methiu/raigal scan --sarif > results.sarif
```

Suba el archivo generado utilizando la acción oficial de GitHub (`github/codeql-action/upload-sarif@v3`).
