---
description: "Modelo de seguridad, almacenamiento de credenciales, variables de entorno y licencias offline."
icon: lock
---

# Modelo de seguridad y credenciales

Raigal almacena credenciales de forma aislada, limpia secretos antes de enviar métricas y admite autenticación sin claves fijas mediante OIDC de GitHub Actions.

## Almacenamiento de credenciales

En equipos locales, las credenciales y licencias se guardan en el directorio del usuario:

- Ruta: `~/.raigal/credentials.json`
- Permisos: `0600` (lectura y escritura exclusivas para el usuario propietario)
- Estructura:
  ```json
  {
    "token": "rgl_sess_...",
    "org": { "id": "org_123", "name": "Acme Corp" },
    "user": { "id": "user_456", "email": "dev@acme.com" },
    "created_at": "2026-09-23T12:00:00.000Z"
  }
  ```

## Configuración de `RAIGAL_TOKEN` y `RAIGAL_API_URL`

En pipelines de CI, contenedores Docker o scripts de automatización, configure la autenticación con variables de entorno:

- `RAIGAL_TOKEN`: token de API de su organización (`rgl_live_...`). Tiene prioridad sobre `~/.raigal/credentials.json`.
- `RAIGAL_API_URL`: dirección del backend (por defecto `https://app.raigal.dev`).

{% tabs %}
{% tab title="GitHub Actions" %}
```yaml
- name: Escaneo Raigal
  env:
    RAIGAL_TOKEN: ${{ secrets.RAIGAL_TOKEN }}
    RAIGAL_API_URL: ${{ secrets.RAIGAL_API_URL || 'https://app.raigal.dev' }}
  run: npx @methiu/raigal scan .
```
{% endtab %}

{% tab title="Terminal (macOS/Linux)" %}
```bash
export RAIGAL_TOKEN="rgl_live_su_token_aqui"
export RAIGAL_API_URL="https://app.raigal.dev"
npx @methiu/raigal whoami
```
{% endtab %}

{% tab title="Windows PowerShell" %}
```powershell
$env:RAIGAL_TOKEN="rgl_live_su_token_aqui"
$env:RAIGAL_API_URL="https://app.raigal.dev"
npx @methiu/raigal whoami
```
{% endtab %}
{% endtabs %}

## Limpieza de secretos y gestión de datos

Antes de enviar telemetría a la nube:
1. El CLI elimina cabeceras de autorización, tokens Bearer, claves privadas y patrones de credenciales habituales.
2. Solo se transmiten recuentos diagnósticos, identificadores de reglas, rutas relativas y diferencias de puntuación. El código fuente nunca se sube a Raigal Cloud.
3. El servidor almacena los registros en tablas de PostgreSQL (`telemetry_runs`, `pull_requests`) mediante operaciones idempotentes.

## Autenticación OIDC en GitHub Actions

Para ejecutar flujos en GitHub Actions sin almacenar secretos fijos:

1. Añada `permissions: id-token: write` al trabajo del flujo.
2. El runner de GitHub genera un token OIDC firmado.
3. Raigal envía dicho token en la cabecera `X-GitHub-OIDC` a `/api/v1/telemetry/runs`.
4. Raigal Cloud valida la firma criptográfica con `https://token.actions.githubusercontent.com` y comprueba que el propietario del repositorio pertenece a su organización.

## Licencias fuera de línea (Ed25519)

Raigal no realiza consultas de red en cada archivo analizado:

1. Al iniciar sesión con `raigal login` o definir `RAIGAL_TOKEN`, el servidor firma una licencia temporal mediante una clave privada Ed25519.
2. El CLI valida la firma sin conexión utilizando la clave pública incluida en el binario (`src/cloud/keys.ts`).
3. La licencia se guarda en `~/.raigal/entitlement.json` durante un máximo de 72 horas, lo que permite trabajar sin conexión a internet.
