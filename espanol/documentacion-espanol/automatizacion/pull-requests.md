---
description: "Cómo instalar la GitHub App oficial de Raigal Cloud, aplicar puertas de calidad en Pull Requests, configurar protección de ramas y supervisar PRs en el tablero Kanban."
icon: git-pull-request
---

# Puerta de Calidad en Pull Requests y Tablero Kanban

Integre Raigal en el ciclo de vida de sus Pull Requests para aplicar automáticamente umbrales de calidad, detectar "AI slop", bloquear riesgos antes de fusionar en la rama principal y visualizar evaluaciones de PRs en tiempo real directamente en el tablero Kanban de Raigal Cloud (`https://app.raigal.dev`).

---

## Arquitectura: Modelo Híbrido de GitHub App + Actions

Raigal utiliza una **Arquitectura Híbrida** de nivel empresarial que combina la GitHub App oficial (`raigal-cloud`) con sus flujos de trabajo en GitHub Actions:

```
┌────────────────────────────────────────────────────────────────────────┐
│                          Ecosistema de GitHub                          │
│                                                                        │
│   1. El desarrollador abre un PR o sube un commit                      │
│          │                                                             │
│          ▼                                                             │
│   Webhook de GitHub ───────────────────────┐                           │
│          │                                 │                           │
│          ▼ (desencadena CI)                │ (evento pull_request)     │
│   Runner de GitHub Actions                 │                           │
│     • Ejecuta linters AST y escaneo slop   ▼                           │
│     • Calcula diff y puntuación    Plano de Control Raigal Cloud       │
│     • Cero código sale del runner   (https://app.raigal.dev)           │
│          │                                 │                           │
│          ▼ (POST /v1/runs)                 │ 2. Crea Check Pendiente   │
│   Ingesta de métricas y Head SHA           │    "Raigal Quality Gate"  │
│          │                                 ▼                           │
│          └─────────────────────────► Resuelve el Check Run             │
│                                      (Éxito / Fallo + Resumen)         │
│                                            │                           │
│                                            ▼                           │
│   Protección de Ramas Aplicada ◄───────────┘                           │
│   Tarjeta Actualizada en Vivo en el Tablero Kanban                     │
└────────────────────────────────────────────────────────────────────────┘
```

### ¿Por qué este modelo híbrido?
- **Cero Exfiltración de Código**: Su código fuente propietario permanece 100% dentro de su runner de GitHub Actions. Raigal Cloud nunca clona, almacena ni accede a su código fuente.
- **Respuesta Pendiente Inmediata**: En el mismo instante en que se abre o sincroniza un PR, la GitHub App oficial registra inmediatamente una comprobación `"Raigal Quality Gate"` en estado `in_progress` en el SHA del commit y crea la tarjeta en su tablero Kanban.
- **Protección Nativa de Ramas**: La comprobación `"Raigal Quality Gate"` aparece directamente en las reglas de protección de ramas de su repositorio como un estado obligatorio para autorizar la fusión.
- **Detección Automática de Fusión y Cierre**: Cuando un PR se fusiona o se cierra, GitHub envía el evento directamente a la App de Raigal, actualizando la tarjeta del tablero automáticamente sin requerir scripts adicionales en CI.

---

## Configuración: GitHub App Oficial + Actions (Recomendado)

Siga estos pasos para conectar su repositorio en menos de 2 minutos:

{% stepper %}

{% step %}
### Instalar la GitHub App Oficial
1. Vaya a la página de la GitHub App oficial de Raigal Cloud:  
   👉 [**https://github.com/apps/raigal-cloud**](https://github.com/apps/raigal-cloud)
2. Haga clic en **Install** (Instalar).
3. Seleccione su organización o cuenta personal de GitHub.
4. Elija **All repositories** (Todos los repositorios) o seleccione los repositorios específicos que desea proteger con Raigal.
5. Haga clic en **Install & Authorize**.
{% endstep %}

{% step %}
### Obtener el Token de la Organización
1. Inicie sesión en su panel en [**https://app.raigal.dev**](https://app.raigal.dev).
2. Abra **Settings** (Configuración) o presione `⌘ K` y elija *Settings*.
3. En la sección **API Tokens**, haga clic en **Create New Token**.
4. Asigne un nombre al token (p. ej., `GitHub Actions CI`) y copie la clave generada (`rgl_live_...`).
{% endstep %}

{% step %}
### Guardar el Secreto en el Repositorio de GitHub
1. En su repositorio de GitHub, vaya a **Settings** > **Secrets and variables** > **Actions**.
2. Haga clic en **New repository secret**.
3. Nombre: `RAIGAL_TOKEN`
4. Valor: Pegue su token `rgl_live_...`.
5. Haga clic en **Add secret**.

{% hint style="tip" %}
**GitHub Actions OIDC sin Secretos:** Si su organización prefiere no almacenar secretos estáticos, puede prescindir de `RAIGAL_TOKEN` configurando `permissions: id-token: write` en su flujo. Raigal Cloud autenticará de forma criptográfica su runner mediante OIDC.
{% endhint %}
{% endstep %}

{% step %}
### Añadir el Flujo de Trabajo en GitHub Actions
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
      id-token: write  # Habilita OIDC de GitHub sin secretos

    steps:
      - name: Descargar Código
        uses: actions/checkout@v4
        with:
          # CRÍTICO: fetch-depth: 0 asegura el historial completo de git
          # para que Raigal pueda calcular diferencias exactas contra la rama base.
          fetch-depth: 0

      - name: Configurar Node.js
        uses: actions/setup-node@v4
        with:
          node-version: 22

      - name: Ejecutar Puerta de Calidad Raigal
        env:
          RAIGAL_TOKEN: ${{ secrets.RAIGAL_TOKEN }}
          RAIGAL_API_URL: ${{ secrets.RAIGAL_API_URL || 'https://app.raigal.dev' }}
        run: |
          npx @methiu/raigal ci --changes --base origin/${{ github.base_ref || 'main' }} --human
```

{% hint style="success" %}
**Cero Pasos Adicionales al Fusionar:** Dado que la GitHub App oficial recibe de forma nativa los webhooks de cierre de PR, no necesita ningún flujo secundario ni script de notificación. Cuando el PR se fusiona o cierra, su tablero Kanban se actualiza automáticamente.
{% endhint %}
{% endstep %}

{% step %}
### Aplicar Protección de Ramas
1. En su repositorio de GitHub, vaya a **Settings** > **Branches**.
2. En las reglas de protección, edite o cree una regla para `main`.
3. Marque la opción **Require status checks to pass before merging** (Requerir comprobaciones de estado antes de fusionar).
4. En el buscador, busque y seleccione **Raigal Quality Gate**.
5. Guarde los cambios.

A partir de este momento, ningún Pull Request podrá fusionarse si no supera los controles de calidad y detección de AI slop de Raigal.
{% endstep %}

{% endstepper %}

---

## Modo Alternativo: CI Independiente (Sin GitHub App)

Si la política de seguridad de su empresa no permite la instalación de GitHub Apps, puede ejecutar Raigal en modo CI autónomo:

{% stepper %}

{% step %}
### Configurar el Secreto del Repositorio
En GitHub **Settings** > **Secrets and variables** > **Actions**, añada `RAIGAL_TOKEN`.
{% endstep %}

{% step %}
### Añadir el Flujo de CI
Añada el archivo `.github/workflows/raigal.yml` con el contenido mostrado anteriormente. En modo autónomo, el control se reflejará como el estado habitual del trabajo de Actions.
{% endstep %}

{% step %}
### Añadir Flujo de Sincronización de Cierre (Opcional)
Sin la GitHub App, puede sincronizar el estado cuando un PR se cierra o fusiona mediante un flujo complementario:

```yaml
name: Sincronización Ciclo de Vida PR

on:
  pull_request:
    types: [closed]

jobs:
  sync-pr-closed:
    name: Sincronizar Estado PR Cerrado
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
          npx @methiu/raigal report-pr-closed \
            --pr ${{ github.event.pull_request.number }} \
            ${{ github.event.pull_request.merged && '--merged' || '' }}
```
{% endstep %}

{% endstepper %}

---

## Supervisión de PRs en el Tablero Kanban

Abra [**Raigal Cloud**](https://app.raigal.dev) para dar seguimiento a los Pull Requests en tiempo real:

1. **Tarjetas Activas**: Cada PR evaluado aparece en la columna **PULL REQUESTS**.
   * **Superó el Control (Verde)**: Muestra la puntuación de calidad (p. ej., `94/100`), autor, rama de origen y rama de destino.
   * **Bloqueado (Rojo)**: Destaca los motivos de bloqueo y el número de infracciones críticas.
2. **Panel Lateral de Inspección Contextual**:
   * Haga clic en cualquier tarjeta de PR para abrir el panel lateral.
   * Revise los **Hallazgos Bloqueantes** con la regla exacta, ruta de archivo y número de línea.
   * Consulte las **Estadísticas de Diferencias** (+líneas añadidas, -eliminadas, archivos modificados).
   * Inspeccione la **Vista Previa de Unified Diff**.
   * Copie con un clic el comando `git checkout` para revisar la rama en local, o use **Open on GitHub** para abrir el PR directamente.

---

## Gestión de Pull Requests desde Forks Públicos

{% hint style="danger" %}
**Seguridad en Forks:** Por diseño de seguridad, GitHub Actions suprime los secretos del repositorio (`${{ secrets.RAIGAL_TOKEN }}`) en ejecuciones de `pull_request` procedentes de forks externos para evitar fugas de credenciales.
{% endhint %}

Si su repositorio recibe contribuciones desde forks públicos, configure **GitHub Actions OIDC**:

```yaml
permissions:
  contents: read
  id-token: write  # Obligatorio para OIDC sin secretos
```

Al incluir `id-token: write`, el CLI de Raigal solicita automáticamente un token OIDC efímero a GitHub y lo transmite mediante el encabezado `X-GitHub-OIDC`. Raigal Cloud valida la firma criptográfica con el proveedor oficial de GitHub (`token.actions.githubusercontent.com`) y confirma que el propietario del repositorio pertenece a los propietarios autorizados de su organización. No se requieren secretos estáticos.

---

## Guía de Resolución de Incidencias

| Síntoma | Causa | Solución |
| :--- | :--- | :--- |
| **La comprobación se queda en pendiente indefinidamente** | El flujo de CI no se ejecutó o superó el tiempo límite | Compruebe que el flujo de GitHub Actions se ejecutó con éxito para el commit `head_sha`. Raigal Cloud limpia automáticamente comprobaciones huérfanas tras 20 minutos. |
| **No aparece el Check Run en el PR** | La GitHub App no está instalada en el repositorio | Asegúrese de haber instalado la [GitHub App oficial de raigal-cloud](https://github.com/apps/raigal-cloud) en el repositorio específico. |
| **HTTP 401 / Token inválido en CI** | `RAIGAL_TOKEN` ausente o revocado | Verifique que `RAIGAL_TOKEN` esté correctamente definido en los secretos de GitHub y coincida con un token activo en [Configuración](https://app.raigal.dev/settings). |
| **HTTP 403 `owner_not_allowed` en CI** | El propietario del repositorio no está sincronizado en la lista de permitidos | Vaya a **Configuración > Propietarios Permitidos** en Raigal Cloud (`https://app.raigal.dev`) y haga clic en **Sincronizar GitHub**. |
| **La vista previa del diff muestra 0 archivos cambiados** | Checkout superficial (`fetch-depth: 1`) | Añada `fetch-depth: 0` al paso `actions/checkout@v4` para disponer del historial git completo y calcular la diferencia contra la rama base. |
| **La tarjeta del PR no se actualiza al fusionar** | Modo autónomo sin flujo de sincronización | Si no utiliza la GitHub App, añada el flujo `report-pr-closed` para sincronizar los eventos de cierre y fusión. |
