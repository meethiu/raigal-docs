---
description: "Cómo instalar la GitHub App de Raigal, aplicar puertas de calidad en pull requests y configurar protección de ramas."
icon: git-pull-request
---

# Puerta de calidad en pull requests y tablero

Configure Raigal para evaluar pull requests, aplicar protección de ramas y supervisar el estado de revisión en el tablero de remediación en `https://app.raigal.dev`.

## Arquitectura: GitHub App y Actions

Raigal combina la GitHub App oficial (`raigal-cloud`) con los flujos de trabajo de GitHub Actions:

```text
┌───────────────────────────────┐        ┌───────────────────────────────┐
│     Repositorio de GitHub     │        │       Raigal Cloud            │
│                               │        │                               │
│  El desarrollador abre un PR  │───────►│ 1. GitHub App recibe evento   │
│  (Head SHA: 3b2a1c...)        │        │    (pull_request.opened)      │
│                               │        │                               │
│  El runner de Actions inicia  │        │                               │
│  (ejecuta raigal ci)          │        │                               │
│          │                    │        │                               │
│          ▼ (POST /v1/runs)    │        │ 2. Crea check pendiente       │
│   Ingesta de métricas y SHA   │───────►│    "Raigal Quality Gate"      │
│                               │        │                               │
│                               │        ▼                               │
│                               │◄─────── Resuelve el check run          │
│                               │         (Éxito / Fallo + Resumen)      │
│                               │        │                               │
│  Protección de rama aplicada  │        │                               │
│  Tarjeta actualizada en vivo  │        │                               │
└───────────────────────────────┘        └───────────────────────────────┘
```

Funcionamiento de este modelo:
- El código no sale del runner: el código fuente se analiza íntegramente en su máquina o en el runner de GitHub Actions. Raigal Cloud no clona ni almacena su código.
- Comprobación pendiente inmediata: al abrir o actualizar un PR, la app registra un check `"Raigal Quality Gate"` en estado `in_progress` en el commit correspondiente y añade la tarjeta al tablero.
- Protección de ramas nativa: la comprobación `"Raigal Quality Gate"` puede definirse como obligatoria en las reglas de protección de ramas de GitHub.
- Actualización automática al fusionar o cerrar: GitHub envía los eventos directamente a la app, actualizando las tarjetas sin necesidad de pasos adicionales en CI.

## Configuración: GitHub App y Actions

{% stepper %}

{% step %}
### 1. Instalar la GitHub App oficial
1. Abra la [GitHub App oficial de Raigal](https://github.com/apps/raigal-cloud).
2. Seleccione **Install**.
3. Elija su organización o cuenta personal de GitHub.
4. Seleccione todos los repositorios o elija los que desee proteger.
5. Confirme con **Install & Authorize**.
{% endstep %}

{% step %}
### 2. Obtener el token de la organización
1. Inicie sesión en [app.raigal.dev](https://app.raigal.dev).
2. Abra **Settings**.
3. En **API Tokens**, pulse **Create New Token**.
4. Asigne un nombre al token y copie la clave generada (`rgl_live_...`).
{% endstep %}

{% step %}
### 3. Guardar el secreto en el repositorio
1. En su repositorio de GitHub, vaya a **Settings** > **Secrets and variables** > **Actions**.
2. Pulse **New repository secret**.
3. Nombre: `RAIGAL_TOKEN`.
4. Valor: pegue la clave `rgl_live_...`.
5. Guarde con **Add secret**.

{% hint style="tip" %}
Si no desea gestionar claves estáticas, configure `permissions: id-token: write` en su flujo. Raigal Cloud autenticará al runner mediante OIDC de GitHub.
{% endhint %}
{% endstep %}

{% step %}
### 4. Añadir el flujo de GitHub Actions
Cree el archivo `.github/workflows/raigal.yml` en su repositorio:

```yaml
name: Puerta de Calidad Raigal

on:
  pull_request:
    types: [opened, synchronize, reopened]
    branches: [main, develop]
  push:
    branches: [main]

jobs:
  gate:
    name: Control de Calidad y Slop
    runs-on: ubuntu-latest
    permissions:
      contents: read
      id-token: write
      pull-requests: write

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
        run: git fetch origin ${{ github.base_ref }} --depth=1

      - name: Ejecutar Puerta de Calidad Raigal
        env:
          RAIGAL_TOKEN: ${{ secrets.RAIGAL_TOKEN }}
          RAIGAL_API_URL: ${{ secrets.RAIGAL_API_URL || 'https://app.raigal.dev' }}
          IS_FORK: ${{ github.event.pull_request.head.repo.full_name != github.repository }}
        run: |
          if [ "$IS_FORK" = "true" ]; then
            npx -y @methiu/raigal@0.2.2 ci --changes --base origin/${{ github.base_ref }} --human
          else
            npx -y @methiu/raigal@0.2.2 ci --changes --base origin/${{ github.base_ref || 'main' }} --report-pr
          fi
```

{% hint style="success" %}
Al recibir los webhooks de cierre directamente a través de la GitHub App, no se requieren flujos adicionales para reportar cuando un PR se cierra o se fusiona.
{% endhint %}
{% endstep %}

{% step %}
### 5. Exigir la comprobación en la protección de ramas
1. En GitHub, abra **Settings** > **Branches** (o **Rules > Rulesets**).
2. En las reglas de protección, edite la regla para su rama principal.
3. Marque **Require status checks to pass before merging**.
4. Busque y seleccione **Raigal Quality Gate**.
5. Guarde los cambios.
{% endstep %}

{% endstepper %}

## Modo CI independiente (sin GitHub App)

Si las políticas de seguridad de su empresa impiden instalar GitHub Apps, puede usar Raigal únicamente con GitHub Actions:

{% stepper %}

{% step %}
### 1. Configurar secreto
En GitHub **Settings** > **Secrets and variables** > **Actions**, añada `RAIGAL_TOKEN`.
{% endstep %}

{% step %}
### 2. Añadir flujo de CI
Añada `.github/workflows/raigal.yml` con el contenido anterior. En modo autónomo, el control se reflejará como un estado de trabajo normal en GitHub Actions.
{% endstep %}

{% step %}
### 3. Sincronizar cierre de PRs (opcional)
Para actualizar el tablero al cerrar o fusionar PRs sin la app, añada este flujo complementario:

```yaml
name: Sincronización Ciclo de Vida PR

on:
  pull_request:
    types: [closed]

jobs:
  sync-pr-closed:
    name: Sync Closed PR Status
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 22

      - name: Notificar Cierre a Raigal Cloud
        env:
          RAIGAL_TOKEN: ${{ secrets.RAIGAL_TOKEN }}
          RAIGAL_API_URL: ${{ secrets.RAIGAL_API_URL || 'https://app.raigal.dev' }}
        run: |
          npx -y @methiu/raigal@0.2.2 report-pr-closed \
            --pr ${{ github.event.pull_request.number }} \
            ${{ github.event.pull_request.merged && '--merged' || '' }}
```
{% endstep %}

{% endstepper %}

## Seguimiento de PRs en el tablero

En [Raigal Cloud](https://app.raigal.dev), abra el tablero de su repositorio:

1. Columna de pull requests:
   - Aprobado: muestra la puntuación (ej: `94/100`), autor, rama de origen y rama de destino.
   - Bloqueado: muestra el recuento de infracciones bloqueantes y categorías con alertas.
2. Panel lateral de inspección:
   - Pulse cualquier tarjeta para desplegar el panel de detalles.
   - Revise los hallazgos bloqueantes con identificador de regla, ruta y línea.
   - Compruebe las estadísticas de cambios y la vista previa del diff unificado.
   - Copie el comando `git checkout` para revisar la rama localmente o pulse **Open on GitHub** para abrir el PR.

## Resolución de incidencias

| Síntoma | Causa | Solución |
| :--- | :--- | :--- |
| La comprobación permanece pendiente | El flujo de CI no terminó o superó el tiempo límite | Compruebe que el flujo finalizó correctamente para el commit `head_sha`. Raigal Cloud descarta comprobaciones huérfanas tras 20 minutos. |
| No aparece el check run en el PR | La GitHub App no está instalada en el repositorio | Compruebe que la [GitHub App raigal-cloud](https://github.com/apps/raigal-cloud) está conectada al repositorio. |
| Error HTTP 401 o token inválido en CI | Clave `RAIGAL_TOKEN` ausente o no válida | Verifique que `RAIGAL_TOKEN` está configurado en los secretos del repositorio y coincide con un token activo en [Configuración](https://app.raigal.dev/settings). |
| Error HTTP 403 `owner_not_allowed` en CI | El propietario del repositorio no figura en la lista permitida | Vaya a **Configuración > Propietarios Permitidos** en Raigal Cloud y pulse **Sincronizar GitHub**. |
| La vista previa del diff muestra 0 archivos cambiados | Checkout superficial (`fetch-depth: 1`) | Añada `fetch-depth: 0` al paso `actions/checkout@v4` para disponer del historial git completo respecto a la rama base. |
| La tarjeta del PR no se actualiza al fusionar | Modo autónomo sin flujo de sincronización | Si no utiliza la GitHub App, añada el flujo de sincronización con `report-pr-closed`. |
