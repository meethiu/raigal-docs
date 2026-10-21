---
description: "Endpoints de la API para gestión de tokens, métricas de repositorios, inspección de políticas e ingesta de ejecuciones."
icon: key
---

# API de organización y gestión de tokens

Raigal Cloud proporciona una API REST para consultar puntuaciones, inspeccionar políticas activas, gestionar tokens e ingerir resultados del CLI.

## Autenticación

Todas las solicitudes deben incluir un token Bearer válido en el encabezado `Authorization`:

```http
Authorization: Bearer rgl_live_su_clave_api_aqui
```

{% hint style="info" %}
Genere claves de API en [app.raigal.dev/settings](https://app.raigal.dev/settings) en la sección **API Tokens**.
{% endhint %}

## Endpoints

### 1. Verificación de identidad
Verifica el token activo y devuelve el usuario, la organización y los propietarios de repositorios permitidos.

```http
GET /v1/whoami
```

Respuesta:
```json
{
  "user": {
    "id": "user_2tQ...",
    "email": "dev@acme.com"
  },
  "org": {
    "id": "org_2tQ...",
    "name": "Acme Engineering",
    "status": "active"
  },
  "plan": "active",
  "trial_started_at": null,
  "trial_ends_at": null,
  "allowed_owners": [
    {
      "id": 1234567,
      "login": "acme"
    }
  ]
}
```

### 2. Política de calidad de la organización
Recupera los umbrales de calidad y las reglas bloqueadas para la organización o un repositorio específico.

```http
GET /v1/policy?repo=acme%2Fbackend-api
```

Respuesta:
```json
{
  "version": 1,
  "hash": "7b8f9a2e...",
  "config": {
    "ci": {
      "failBelow": 80
    },
    "rules": {
      "ai-slop/swallowed-exception": "error"
    },
    "exclude": []
  },
  "locked": [
    "ci.failBelow",
    "rules.ai-slop/swallowed-exception"
  ]
}
```

### 3. Ingesta de ejecuciones del CLI
Envía un informe de ejecución generado por el CLI (`raigal ci` o `raigal scan`):

```http
POST /v1/runs
Content-Type: application/json
```

Cuerpo de la solicitud (esquema estricto del contrato `Run`):
```json
{
  "run_id": "c9bf9e57-1685-4c89-bafb-ff5af830be8a",
  "command": "ci",
  "repo": {
    "provider": "github",
    "slug": "acme/backend-api"
  },
  "owner_verified": true,
  "git": {
    "branch": "main",
    "head_sha": "e4f5a6b7c8d9e0f1a2b3c4d5e6f7a8b9c0d1e2f3",
    "base_ref": "main"
  },
  "ci": {
    "provider": "github-actions",
    "run_url": "https://github.com/acme/backend-api/actions/runs/123456"
  },
  "flags": ["changes"],
  "cli_version": "0.2.5",
  "started_at": "2026-10-21T10:00:00.000Z",
  "ended_at": "2026-10-21T10:00:12.000Z",
  "exit_code": 0,
  "steps": [
    { "seq": 0, "name": "engine:ai-slop", "ok": true, "ms": 45 },
    { "seq": 1, "name": "engine:security", "ok": true, "ms": 32 }
  ],
  "report": {
    "score": 92,
    "scoreable": true,
    "files_scanned": 12,
    "findings": []
  }
}
```

Respuesta:
```json
{
  "ok": true,
  "run_id": "c9bf9e57-1685-4c89-bafb-ff5af830be8a"
}
```

## Gestión de tokens

Los tokens pueden crearse en el panel web o mediante la API de tokens:

### Crear token de API
```http
POST /api/tokens
Content-Type: application/json

{
  "name": "Runner de Producción"
}
```

Respuesta:
```json
{
  "token": "rgl_live_8f3a9b1c7e2d...",
  "prefix": "rgl_live_8f3a",
  "name": "Runner de Producción",
  "id": "key_9b2e...",
  "created_at": "2026-10-21T10:00:00.000Z"
}
```

{% hint style="danger" %}
Guarde su token al crearlo. La cadena completa se muestra una sola vez. Raigal Cloud almacena un hash SHA-256 y no puede recuperar tokens perdidos.
{% endhint %}

### Gestión del dashboard y repositorios

La gestión de repositorios, seguimiento de ramas y revocación de tokens está disponible a través de la interfaz web en [app.raigal.dev/settings](https://app.raigal.dev/settings) utilizando su sesión autenticada de navegador.
