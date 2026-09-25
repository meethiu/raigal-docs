---
description: "Guía paso a paso de incorporación para equipos de ingeniería, directores de tecnología y administradores que utilizan Raigal Cloud."
icon: building
---

# Configuración y Onboarding de la Organización

Esta guía detalla la configuración de Raigal para su organización de ingeniería utilizando la plataforma alojada en **[app.raigal.dev](https://app.raigal.dev)** y la **[GitHub App Oficial de Raigal](https://github.com/apps/raigal-cloud)**.

---

## Lista de Comprobación de Onboarding

| Paso | Acción | Dónde | Tiempo |
|---|---|---|---|
| **1. Crear Organización** | Iniciar sesión con GitHub, Google o correo corporativo | [app.raigal.dev](https://app.raigal.dev) | 1 min |
| **2. Instalar GitHub App** | Instalar la App oficial en sus repositorios | [github.com/apps/raigal-cloud](https://github.com/apps/raigal-cloud) | 1 min |
| **3. Obtener Token de Org** | Copiar su token de API `rgl_live_...` | Configuración &rarr; Tokens de API | 30 seg |
| **4. Configurar CI/CD** | Añadir `.github/workflows/raigal.yml` | Sus repositorios | 2 min |
| **5. Invitar Ingenieros** | Enviar invitaciones al equipo o configurar SSO | Configuración &rarr; Miembros de la Organización | 1 min |

---

## Paso 1: Registro y Creación de su Organización

1. Acceda a **[app.raigal.dev](https://app.raigal.dev)**.
2. Inicie sesión con GitHub, Google o su correo corporativo.
3. En el selector de organizaciones situado en la esquina superior izquierda, seleccione **Crear Organización** (o elija el espacio de trabajo existente de su equipo).
4. Introduzca el nombre de su organización (p. ej., *Acme Engineering*).

{% hint style="info" %}
Raigal admite el cambio de organización multi-inquilino. Si pertenece a varios equipos o gestiona cuentas de clientes independientes, puede alternar entre ellos cómodamente desde el selector superior izquierdo.
{% endhint %}

---

## Paso 2: Instalar la GitHub App Oficial de Raigal

La GitHub App de Raigal gestiona el ciclo de vida de la puerta de calidad en Pull Requests: publica comprobaciones pendientes inmediatas al abrir PRs, actualiza el estado de la comprobación al finalizar el CI y sincroniza los estados de fusión y cierre de forma automática.

1. En el panel de Raigal Cloud, vaya a **Configuración** &rarr; haga clic en **Sincronizar Cuentas de GitHub** (o visite **[github.com/apps/raigal-cloud](https://github.com/apps/raigal-cloud)** directamente).
2. Haga clic en **Install App** (Instalar App).
3. Seleccione su organización o cuenta personal de GitHub.
4. Elija **All repositories** (Todos los repositorios) o seleccione los repositorios específicos que desea proteger con Raigal.
5. Haga clic en **Install & Authorize** (Instalar y autorizar).

Una vez instalada, GitHub vincula automáticamente sus repositorios con su organización en Raigal Cloud.

---

## Paso 3: Obtener las Credenciales de la Organización

Para que GitHub Actions o las sesiones locales de los desarrolladores puedan enviar métricas de calidad a su panel en la nube, obtenga la clave de API de su organización:

1. Abra **[app.raigal.dev/settings](https://app.raigal.dev)**.
2. En la tarjeta **Token Activo de la Organización**, se muestra su clave en vivo (`rgl_live_...`).
3. Haga clic en **Copiar** para copiar el token.
4. Si necesita generar una nueva clave o rotar credenciales, haga clic en **Generar Nuevo Token** en la sección **Claves de API**.

{% hint style="tip" %}
**GitHub Actions sin Secretos (OIDC):** Si prefiere no gestionar tokens estáticos en GitHub Secrets, active `permissions: id-token: write` en su flujo de trabajo. Raigal Cloud autentica de forma nativa los tokens OIDC del runner de GitHub sin requerir secretos estáticos.
{% endhint %}

---

## Paso 4: Añadir la Puerta de Calidad en GitHub Actions

Añada el archivo `.github/workflows/raigal.yml` a cada repositorio que desee proteger:

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
      id-token: write  # Habilita OIDC de GitHub sin secretos estáticos

    steps:
      - name: Descargar Código
        uses: actions/checkout@v4
        with:
          fetch-depth: 0 # Necesario para calcular diferencias precisas respecto a la rama base

      - name: Configurar Node.js
        uses: actions/setup-node@v4
        with:
          node-version: 22

      - name: Ejecutar Puerta de Calidad Raigal
        env:
          RAIGAL_TOKEN: ${{ secrets.RAIGAL_TOKEN }}
          RAIGAL_API_URL: https://app.raigal.dev
        run: |
          npx @methiu/raigal ci --changes --base origin/${{ github.base_ref || 'main' }} --human
```

En su repositorio de GitHub:
- Vaya a **Settings** &rarr; **Secrets and variables** &rarr; **Actions**.
- Añada un nuevo secreto de repositorio:
  - Nombre: `RAIGAL_TOKEN`
  - Valor: Su token `rgl_live_...` obtenido en el Paso 3.

---

## Paso 5: Invitar a su Equipo de Ingeniería

1. En **Configuración**, haga clic en el nombre de su organización o en la pestaña **Miembros**.
2. Haga clic en **Invitar Miembros** para enviar invitaciones por correo electrónico.
3. Asigne los roles correspondientes (**Admin** para responsables de plataforma, **Member** para ingenieros).
4. Los ingenieros invitados recibirán un correo para unirse al espacio de trabajo de su organización en **[app.raigal.dev](https://app.raigal.dev)**.

---

## Configuración en las Máquinas de los Desarrolladores

Los desarrolladores pueden autenticar sus entornos locales para transmitir métricas de escaneo, acceder a las políticas de la organización y ejecutar reparaciones autónomas:

```bash
# Autenticarse mediante el navegador
npx @methiu/raigal login

# Verificar la organización activa
npx @methiu/raigal whoami
```

Para instalar hooks de intercepción en tiempo real en Claude Code o Cursor:

```bash
npx @methiu/raigal hook install
```

---

## Gestión de Repositorios Conectados

En **Configuración &rarr; Repositorios Conectados**, los administradores pueden:
- **Supervisar la Actividad en Directo:** Expandir cualquier repositorio para revisar escaneos recientes, principales hallazgos y puntuaciones de calidad por rama.
- **Acceder al Tablero Kanban:** Hacer clic en **Tablero** para ir directamente al panel de remediación en 4 etapas de ese repositorio.
- **Eliminar Repositorios en Desuso:** Hacer clic en **Eliminar** (icono de papelera) para borrar permanentemente un repositorio inactivo, eliminando en cascada sus ejecuciones históricas y registros de PR asociados.
