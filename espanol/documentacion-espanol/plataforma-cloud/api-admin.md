---
description: "Endpoints de la API de organización, autenticación, gestión de repositorios y rotación de tokens."
icon: key
---

# API de la organización y gestión de tokens

Raigal Cloud ofrece una API REST para consultar puntuaciones, gestionar tokens, registrar resultados de CI y eliminar repositorios.

## Autenticación

Las solicitudes deben incluir una clave de API de la organización (`rgl_live_...`) en la cabecera de autorización:

{% tabs %}
{% tab title="Cabecera Bearer" %}
```http
Authorization: Bearer rgl_live_su_clave_aqui
```
{% endtab %}

{% tab title="Cabecera x-api-key" %}
```http
x-api-key: rgl_live_su_clave_aqui
```
{% endtab %}
{% endtabs %}

{% hint style="info" %}
Puede generar claves de API en [app.raigal.dev/settings](https://app.raigal.dev/settings), dentro de **Tokens de API**.
{% endhint %}

## Endpoints

### 1. Comprobación de identidad
Verifica el token activo y devuelve los datos de la organización.

```http
GET /v1/whoami
```

Respuesta:
```json
{
  "orgId": "org_2tQ...",
  "orgName": "Acme Engineering",
  "tier": "enterprise",
  "authenticated": true
}
```

### 2. Política de calidad
Recupera los umbrales de calidad configurados para la organización.

```http
GET /v1/policy
```

Respuesta:
```json
{
  "minScore": 75,
  "failOnSeverity": "error",
  "blockedRules": [
    "ai-slop/trivial-comment",
    "security/eval",
    "ai-slop/swallowed-exception"
  ],
  "engines": {
    "format": true,
    "lint": true,
    "codeQuality": true,
    "aiSlop": true,
    "security": true
  }
}
```

### 3. Listar repositorios
Obtiene la lista de repositorios vinculados a la organización.

```http
GET /v1/runs/repos
```

Respuesta:
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

### 4. Eliminar un repositorio
Elimina un repositorio y su historial de escaneos, registros de pull requests y hallazgos asociados.

```http
DELETE /v1/runs/repos/:repoId
```

`:repoId` puede ser el UUID del repositorio o el nombre del repositorio codificado en la URL (ej: `acme%2Fpayment-service`).

Respuesta:
```json
{
  "ok": true,
  "message": "Repository acme/payment-service and all associated runs removed successfully."
}
```

{% hint style="warning" %}
Esta operación es definitiva. También puede eliminar repositorios desde la sección **Configuración** del panel web.
{% endhint %}

### 5. Obtener datos del tablero
Descarga las columnas y tarjetas del tablero en formato JSON.

```http
GET /v1/runs/board?repoId=acme/payment-service&branch=main
```

Parámetros de consulta:
- `repoId`: (opcional) filtrar por nombre de repositorio.
- `branch`: (opcional) filtrar por rama git (por defecto, rama principal).

Respuesta:
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

### 6. Registrar ejecución de CI personalizada
Para sistemas de CI distintos a GitHub Actions (Jenkins, GitLab CI, CircleCI), envíe los resultados del escaneo directamente:

```http
POST /v1/runs
Content-Type: application/json
```

Cuerpo de la solicitud:
```json
{
  "repoId": "acme/backend-api",
  "branch": "main",
  "commitSha": "e4f5a6b7...",
  "score": 88,
  "exitCode": 0,
  "engines": ["format", "lint", "code-quality", "ai-slop", "security"],
  "findingsCount": 2,
  "findings": [
    {
      "ruleId": "ai-slop/unsafe-type-assertion",
      "severity": "warn",
      "file": "src/auth/jwt.ts",
      "line": 45,
      "message": "Aserción de tipo insegura que elude la comprobación de tipos."
    }
  ]
}
```

## Gestión de tokens

Los tokens pueden crearse y revocarse desde el panel web o mediante la API de tokens:

### Crear token de API
```http
POST /api/tokens
Content-Type: application/json

{
  "name": "Production Deploy Runner"
}
```

Respuesta:
```json
{
  "token": "rgl_live_8f3a9b1c7e2d...",
  "prefix": "rgl_live_8f3a",
  "name": "Production Deploy Runner",
  "created_at": "2026-09-24T18:00:00Z"
}
```

{% hint style="danger" %}
Guarde su token al crearlo. El valor completo solo se muestra una vez. Raigal Cloud almacena un hash SHA-256 y no puede recuperar tokens perdidos.
{% endhint %}

### Revocar token de API
```http
DELETE /api/tokens/:tokenId
```

Revoca de inmediato el token indicado, denegando solicitudes posteriores.
