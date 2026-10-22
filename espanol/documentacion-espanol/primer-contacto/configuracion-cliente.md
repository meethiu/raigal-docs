---
description: "Guía de configuración para equipos, administradores y clientes en fase piloto que usan Raigal Cloud."
icon: building
---

# Configuración inicial de la organización

Esta guía explica cómo configurar Raigal para su equipo utilizando el panel web en [app.raigal.dev](https://app.raigal.dev) y la [GitHub App oficial de Raigal](https://github.com/apps/raigal-cloud).

Raigal está diseñado para equipos de ingeniería que adoptan asistentes de programación asistidos por IA (Claude Code, Cursor, Copilot, OpenCode). Las nuevas organizaciones comienzan con una prueba de evaluación completa de 14 días con acceso total a los seis motores de análisis, comprobaciones automáticas en pull requests y el panel de remediación.

## Pasos de configuración

| Paso | Acción | Dónde | Tiempo |
|---|---|---|---|
| 1. Crear organización | Iniciar sesión con GitHub, Google o correo corporativo | [app.raigal.dev](https://app.raigal.dev) | 1 min |
| 2. Instalar GitHub App | Conectar la app a los repositorios seleccionados | [github.com/apps/raigal-cloud](https://github.com/apps/raigal-cloud) | 1 min |
| 3. Copiar token de API | Copiar su clave `rgl_live_...` | Configuración > Tokens de API | 30 seg |
| 4. Añadir flujo de CI | Crear `.github/workflows/raigal.yml` | Repositorio de destino | 2 min |
| 5. Verificar primer escaneo | Abrir un pull request o enviar un commit a main | GitHub | 1 min |
| 6. Invitar al equipo | Enviar invitaciones a desarrolladores y líderes técnicos | Configuración > Miembros | 1 min |

## 1. Crear el espacio de trabajo de su organización

1. Ingrese en [app.raigal.dev](https://app.raigal.dev).
2. Inicie sesión con GitHub, Google o su correo corporativo.
3. Abra el selector de organizaciones en la esquina superior derecha y elija **Crear Organización** (o seleccione un espacio de trabajo existente).
4. Indique el nombre de su empresa u organización (ejemplo: `Acme Corp`).
5. Su espacio de trabajo se creará de inmediato con un periodo de prueba de evaluación activo de 14 días.

{% hint style="info" %}
Si gestiona múltiples equipos, cuentas de clientes o entornos de prueba, puede alternar entre organizaciones en cualquier momento desde el selector superior derecho.
{% endhint %}

## 2. Instalar la GitHub App oficial de Raigal

La [GitHub App de Raigal](https://github.com/apps/raigal-cloud) conecta sus repositorios con el plano de control en la nube. Registra comprobaciones pendientes al abrir o actualizar pull requests, resuelve los check runs al finalizar el análisis de CI y mantiene actualizado el tablero de remediación al fusionar o cerrar ramas.

1. En el panel de Raigal Cloud, vaya a **Settings** y elija **Sync GitHub Accounts** (o abra directamente [github.com/apps/raigal-cloud](https://github.com/apps/raigal-cloud)).
2. Pulse **Install App**.
3. Seleccione su organización o cuenta personal de GitHub.
4. Elija **All repositories** o seleccione específicamente los repositorios que desea proteger con Raigal.
5. Confirme con **Install & Authorize**.

Al instalarse, GitHub vincula automáticamente su cuenta en **Settings > Allowed Owners** en Raigal Cloud, autorizando las ejecuciones de CI de esos proyectos.

## 3. Obtener el token de API de la organización

Para permitir que los runners de GitHub Actions o las terminales locales transmitan métricas de escaneo a su panel:

1. Abra [app.raigal.dev/settings](https://app.raigal.dev/settings).
2. En la sección **API Tokens**, localice la tarjeta **Active Organization Token** y copie su clave (`rgl_live_...`).
3. Si desea crear una clave exclusiva para runners de CI o automatizaciones, pulse **Create New Token**, asigne un nombre representativo (ej: `github-actions-ci`) y copie el token generado.

{% hint style="danger" %}
Los tokens de API se muestran completos una sola vez al crearse. Guarde su clave en un lugar seguro. Raigal Cloud almacena únicamente hashes SHA-256 y no puede recuperar tokens perdidos.
{% endhint %}

{% hint style="tip" %}
En GitHub Actions también puede utilizar autenticación sin claves fijas configurando `permissions: id-token: write` en su flujo. Raigal Cloud valida la identidad del runner mediante OIDC criptográfico de GitHub. No obstante, definir el secreto `RAIGAL_TOKEN` en GitHub Secrets se recomienda para máxima consistencia multiplataforma.
{% endhint %}

## 4. Añadir el flujo de GitHub Actions

Añada el archivo `.github/workflows/raigal.yml` a su repositorio:

```yaml
name: Puerta de Calidad Raigal

on:
  push:
    branches: [main, master, develop]
  pull_request:
    branches: [main, master, develop]

permissions:
  contents: read
  id-token: write        # Habilita autenticación OIDC criptográfica sin token
  security-events: write # Requerido si se exportan informes SARIF a GitHub Code Scanning

jobs:
  quality-gate:
    name: Code Quality & Slop Gate
    runs-on: ubuntu-latest

    steps:
      - name: Descargar Codigo
        uses: actions/checkout@v4
        with:
          fetch-depth: 0 # Historial completo necesario para calcular diffs con la rama base

      - name: Configurar Node.js
        uses: actions/setup-node@v4
        with:
          node-version: 22

      - name: Ejecutar Puerta de Calidad Raigal
        env:
          RAIGAL_TOKEN: ${{ secrets.RAIGAL_TOKEN }}
          # RAIGAL_ORG_ID: ${{ vars.RAIGAL_ORG_ID }} # Requerido solo si el propietario pertenece a varias organizaciones
        run: |
          if [ "${{ github.event_name }}" = "pull_request" ]; then
            git fetch origin ${{ github.base_ref }} --depth=1
            npx --yes @methiu/raigal@latest ci --changes --base origin/${{ github.base_ref }}
          else
            npx --yes @methiu/raigal@latest ci .
          fi

      - name: Generar Informe de Seguridad SARIF (Opcional)
        if: always()
        run: npx --yes @methiu/raigal@latest scan . --sarif > raigal-results.sarif

      - name: Subir SARIF a GitHub Code Scanning (Opcional)
        if: always() && github.event.pull_request.head.repo.full_name == github.repository
        uses: github/codeql-action/upload-sarif@v3
        continue-on-error: true
        with:
          sarif_file: raigal-results.sarif
          category: raigal
```

### Guardar el secreto en GitHub:
1. En su repositorio de GitHub, abra **Settings** > **Secrets and variables** > **Actions**.
2. Pulse **New repository secret**.
3. Nombre: `RAIGAL_TOKEN`.
4. Valor: pegue su clave `rgl_live_...`.
5. Confirme con **Add secret**.

## 5. Verificar el primer escaneo

Para comprobar que la integración funciona de extremo a extremo:

1. **Escaneo en Pull Request**: Abra un pull request en su repositorio.
   - La GitHub App de Raigal crea de inmediato un check run en estado `in_progress` llamado **Raigal Quality Gate**.
   - El runner de GitHub Actions ejecuta `raigal ci --changes --base origin/<base>`.
   - La comprobación finaliza con un resumen detallado con la puntuación numérica (0-100), conteo de incidencias y desglose de reglas.
   - Aparece la tarjeta correspondiente en su panel de Raigal Cloud en la pestaña **Pull Requests**.
2. **Escaneo en rama principal**: Envíe un commit a `main` (o fusione su pull request).
   - El flujo ejecuta un escaneo completo del repositorio (`raigal ci .`).
   - La puntuación de salud base y los problemas activos se actualizan en las pestañas **Errors** y **Warnings** en [app.raigal.dev](https://app.raigal.dev).

## 6. Invitar al equipo

1. En Raigal Cloud, vaya a **Settings** > **Members**.
2. Pulse **Invite Member** e introduzca el correo de su colega.
3. Asigne el rol:
   - **Admin**: Acceso total para gestionar repositorios, tokens de API, facturación y miembros.
   - **Member**: Acceso para ver repositorios, inspeccionar hallazgos, revisar PRs y usar herramientas locales.
4. El usuario invitado recibirá un correo con el enlace de activación para unirse al espacio de trabajo.

## Configuración en el equipo local de desarrollo

Los desarrolladores pueden ejecutar escaneos y correcciones automáticas en su máquina antes de publicar código:

```bash
# Iniciar sesion interactiva en el terminal via navegador
npx @methiu/raigal login

# Comprobar la organizacion activa y repositorios autorizados
npx @methiu/raigal whoami

# Comprobar herramientas locales y motores
npx @methiu/raigal doctor

# Ejecutar escaneo local completo
npx @methiu/raigal scan .

# Aplicar correcciones automaticas seguras
npx @methiu/raigal fix .

# Instalar hooks de intercepcion para agentes
npx @methiu/raigal hook install --claude
```

## Resolucion de incidencias comunes de configuracion

| Sintoma | Causa probable | Solucion concreta |
|---|---|---|
| `ambiguous_organization` (HTTP 400) | La cuenta propietaria del repositorio esta vinculada a multiples organizaciones en Raigal | En el flujo de CI, defina `RAIGAL_ORG_ID: ${{ vars.RAIGAL_ORG_ID }}` con el ID de su organizacion (`org_...`) visible en [Settings](https://app.raigal.dev/settings). |
| `owner_not_allowed` (HTTP 403) | El propietario del repositorio no esta vinculado a su organizacion de Raigal | En Raigal Cloud, vaya a **Settings > Allowed Owners** y pulse **Sync GitHub**, o asegurese de que la GitHub App este instalada en la cuenta propietaria del repositorio. |
| Check run atascado en `in_progress` | El flujo de CI fallo antes de ejecutar `raigal ci` o el archivo YAML no existe | Compruebe que `.github/workflows/raigal.yml` se ejecuto para el `head_sha` del PR. Raigal Cloud descarta comprobaciones huerfanas tras 20 minutos. |
| Error HTTP 401 Unauthorized en CI | Clave `RAIGAL_TOKEN` ausente o no valida | Verifique que `RAIGAL_TOKEN` este guardado en los secretos del repositorio y coincida con un token activo en [Settings > API Tokens](https://app.raigal.dev/settings). |
| Vista previa de diff muestra 0 archivos cambiados | Clonado superficial en el runner de GitHub Actions | Asegurese de que `actions/checkout@v4` incluya `fetch-depth: 0` para disponer del historial git completo respecto a la rama base. |
