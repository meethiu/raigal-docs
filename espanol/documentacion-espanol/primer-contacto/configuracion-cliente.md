---
description: "Guía de configuración para equipos y administradores de Raigal Cloud."
icon: building
---

# Configuración inicial de la organización

Esta guía explica cómo configurar Raigal para su equipo utilizando el panel web en [app.raigal.dev](https://app.raigal.dev) y la [GitHub App de Raigal](https://github.com/apps/raigal-cloud).

## Pasos de configuración

| Paso | Acción | Dónde | Tiempo |
|---|---|---|---|
| 1. Crear organización | Iniciar sesión con GitHub, GitLab o correo corporativo | [app.raigal.dev](https://app.raigal.dev) | 1 min |
| 2. Instalar GitHub App | Añadir la app a los repositorios seleccionados | [github.com/apps/raigal-cloud](https://github.com/apps/raigal-cloud) | 1 min |
| 3. Copiar token de API | Copiar su clave `rgl_live_...` | Configuración > Tokens de API | 30 seg |
| 4. Añadir flujo de CI | Crear `.github/workflows/raigal.yml` | Sus repositorios | 2 min |
| 5. Invitar al equipo | Enviar invitaciones o configurar SSO | Configuración > Miembros | 1 min |

## 1. Crear su organización

1. Entre en [app.raigal.dev](https://app.raigal.dev).
2. Inicie sesión con GitHub, GitLab o su correo de trabajo.
3. Abra el selector de organizaciones en la esquina superior derecha y elija **Crear Organización** (o seleccione un espacio existente).
4. Indique el nombre de su organización.

{% hint style="info" %}
Si gestiona varias cuentas o clientes, puede alternar entre organizaciones desde el menú superior derecho.
{% endhint %}

## 2. Instalar la GitHub App de Raigal

La GitHub App gestiona las comprobaciones de pull requests: añade estados pendientes al abrir PRs, actualiza los resultados cuando finaliza el CI y refleja fusiones y cierres de ramas.

1. En el panel de Raigal Cloud, vaya a **Configuración** y elija **Sincronizar Cuentas de GitHub** (o abra [github.com/apps/raigal-cloud](https://github.com/apps/raigal-cloud)).
2. Seleccione **Install App**.
3. Elija su organización o cuenta de GitHub.
4. Seleccione todos los repositorios o marque los que desee proteger.
5. Confirme con **Install & Authorize**.

GitHub conectará esos repositorios con su organización en Raigal Cloud.

## 3. Obtener credenciales de la organización

Para que GitHub Actions o las sesiones de terminal envíen métricas a su panel, copie su clave de API:

1. Abra [app.raigal.dev/settings](https://app.raigal.dev).
2. En la tarjeta **Token Activo de la Organización**, copie el valor `rgl_live_...`.
3. Si necesita rotar la clave, pulse **Generar Nuevo Token** en **Claves de API**.

{% hint style="tip" %}
Si prefiere evitar claves estáticas en GitHub, incluya `permissions: id-token: write` en su flujo de trabajo. Raigal Cloud valida la identidad del runner mediante OIDC de GitHub.
{% endhint %}

## 4. Añadir el flujo de GitHub Actions

Añada el archivo `.github/workflows/raigal.yml` a cada repositorio:

```yaml
name: Puerta de Calidad Raigal

on:
  pull_request:
    branches: [main, develop]
  push:
    branches: [main]

jobs:
  quality-gate:
    name: Control de Calidad y Slop
    runs-on: ubuntu-latest
    permissions:
      contents: read
      id-token: write
      pull-requests: write
      security-events: write

    steps:
      - name: Descargar Código
        uses: actions/checkout@v4
        with:
          fetch-depth: 0

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
            npx -y @methiu/raigal@latest ci --changes --base origin/${{ github.base_ref }} --human
          else
            npx -y @methiu/raigal@latest ci --changes --base origin/${{ github.base_ref || 'main' }}
          fi
```

En su repositorio de GitHub:
1. Vaya a **Settings** > **Secrets and variables** > **Actions**.
2. Añada un secreto llamado `RAIGAL_TOKEN` con el valor `rgl_live_...`.

## 5. Invitar al equipo

1. En **Configuración**, abra la pestaña **Miembros**.
2. Pulse **Invitar Miembros** e introduzca las direcciones de correo.
3. Asigne los roles: **Admin** para responsables de plataforma o **Member** para desarrolladores.
4. Cada usuario recibirá un correo con el enlace para unirse a la organización en [app.raigal.dev](https://app.raigal.dev).

## Configuración en el equipo local

Los desarrolladores pueden autenticar sus terminales para enviar métricas y usar herramientas de reparación:

```bash
# Iniciar sesión en el navegador
raigal login

# Comprobar la organización activa y repositorios permitidos
raigal whoami

# Comprobar herramientas locales y entornos
raigal doctor

# Capturar línea base antes de escanear repositorios heredados
raigal hook baseline

# Instalar hooks de intercepción para agentes activos
raigal hook install --claude
```

## Gestión de repositorios conectados

En **Configuración** > **Repositorios Conectados**, los administradores pueden:
- Desplegar un repositorio para revisar ejecuciones recientes, hallazgos frecuentes y puntuaciones por rama.
- Pulsar **Tablero** para abrir el panel de remediación en 4 etapas.
- Pulsar **Eliminar** para desvincular un repositorio inactivo y borrar sus registros históricos de escaneos y pull requests.
