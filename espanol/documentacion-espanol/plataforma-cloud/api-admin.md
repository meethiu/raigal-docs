---
description: "Puntos de conexión de la API de organización, autenticación, gestión de repositorios y rotación de tokens para clientes de Raigal Cloud."
icon: key
---

# API de la Organización y Gestión de Tokens

Raigal Cloud proporciona una API REST que permite a las organizaciones clientes gestionar tokens de API, consultar puntuaciones de calidad de repositorios, ingerir telemetría de CI personalizada y purgar datos de repositorios de manera programática.

---

## Autenticación

Todas las solicitudes a la API desde scripts, pipelines automatizados o integraciones personalizadas deben autenticarse mediante una Clave de API de la Organización (`rgl_live_...`):

{% tabs %}
{% tab title="Encabezado Bearer" %}
```http
Authorization: Bearer rgl_live_su_clave_aqui
```
{% endtab %}

{% tab title="Encabezado x-api-key" %}
```http
x-api-key: rgl_live_su_clave_aqui
```
{% endtab %}
{% endtabs %}

{% hint style="info" %}
Puede generar claves de API desde su panel de control en [**https://app.raigal.dev/settings**](https://app.raigal.dev/settings) en la sección **Tokens de API**.
{% endhint %}

---

## Referencia de Endpoints para Clientes

### 1. Introspección de Identidad y Organización
Verifique el token activo y compruebe los datos de su organización.

```http
GET /v1/whoami
```

**Respuesta:**
```json
{
  "orgId": "org_2tQ...",
  "orgName": "Acme Engineering",
  "tier": "enterprise",
  "authenticated": true
}
```

---

### 2. Política de Calidad de la Organización
Recupere la política de calidad activa y los umbrales configurados para su organización.

```http
GET /v1/policy
```

**Respuesta:**
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

### 3. Listar Repositorios Monitorizados
Obtenga la lista de todos los repositorios activos vinculados a su organización.

```http
GET /v1/runs/repos
```

**Respuesta:**
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

### 4. Eliminar un Repositorio
Elimine de forma definitiva un repositorio y purgue en cascada todas sus ejecuciones históricas, tarjetas de PR y hallazgos diagnósticos.

```http
DELETE /v1/runs/repos/:repoId
```

* `:repoId` puede ser el UUID del repositorio o el nombre del repositorio codificado en la URL (p. ej., `acme%2Fpayment-service`).

**Respuesta:**
```json
{
  "ok": true,
  "message": "Repositorio acme/payment-service y todas sus ejecuciones asociadas han sido eliminados correctamente."
}
```

{% hint style="warning" %}
Esta operación es irreversible. También puede realizar esta acción visualmente desde la sección **Configuración** de la interfaz web.
{% endhint %}

---

### 5. Obtener Datos del Tablero Kanban de Remediación
Obtenga la información del tablero Kanban en 4 etapas en formato JSON. Ideal para alimentar paneles de métricas internos, resúmenes ejecutivos o bots corporativos de Slack.

```http
GET /v1/runs/board?repoId=acme/payment-service&branch=main
```

**Parámetros de Consulta:**
- `repoId`: (Opcional) Filtrar tarjetas por nombre de repositorio.
- `branch`: (Opcional) Filtrar tarjetas por rama de Git (por defecto, rama principal).

**Respuesta:**
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

### 6. Ingesta de Ejecuciones de CI Personalizadas
Si ejecuta CI en un entorno distinto a GitHub Actions (p. ej., Jenkins, GitLab CI, CircleCI), puede enviar los resultados de sus escaneos directamente a través de la API:

```http
POST /v1/runs
Content-Type: application/json
```

**Cuerpo de la Solicitud:**
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
      "message": "Aserción de tipo redundante detectada."
    }
  ]
}
```

---

## Ciclo de Vida y Rotación de Tokens

Las claves de API pueden crearse y rotarse desde el panel web en `https://app.raigal.dev/settings` o de manera programática:

### Crear Token de API
```http
POST /api/tokens
Content-Type: application/json

{
  "name": "Runner Despliegue Producción"
}
```

**Respuesta:**
```json
{
  "token": "rgl_live_8f3a9b1c7e2d...",
  "prefix": "rgl_live_8f3a",
  "name": "Runner Despliegue Producción",
  "created_at": "2026-09-24T18:00:00Z"
}
```

{% hint style="danger" %}
Guarde su token inmediatamente en un gestor seguro. El valor en texto claro **solo se devuelve una vez** en el momento de su creación. Raigal Cloud cifra y genera un hash SHA-256 de todas las claves y no puede recuperar tokens perdidos.
{% endhint %}

### Revocar Token de API
```http
DELETE /api/tokens/:tokenId
```

Revoca de inmediato la clave indicada, rechazando cualquier solicitud posterior que utilice dicho token.
