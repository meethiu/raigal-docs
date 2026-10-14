---
description: "Sesiones de reparación automatizadas en git worktrees con agentes de IA locales."
icon: robot
---

# Reparación automatizada con agentes

Para hallazgos que requieran refactorización o cambios semánticos, `raigal agent` ejecuta una sesión de reparación iterativa con una herramienta local de IA en un git worktree aislado.

## Cómo funcionan las reparaciones en worktrees

Para proteger la rama activa de modificaciones no verificadas, `raigal agent` aísla todos los cambios:

1. Raigal crea un git worktree temporal en `.raigal/worktrees/agent-<timestamp>-<pid>/`, registrado automáticamente en `.git/info/exclude` para no ensuciar el estado de git.
2. El proveedor del agente (Claude Code, OpenCode o Codex) recibe el diagnóstico, el contexto de los archivos y la puntuación objetivo en su prompt.
3. El agente realiza ediciones y verifica los cambios dentro del worktree.
4. Tras revisar los cambios, puede ejecutar `raigal agent apply <session-id>` para fusionarlos en local o usar `--pr` para abrir un pull request en GitHub.

## Proveedores compatibles

Raigal se conecta con tres herramientas de programación locales:

| Proveedor | Invocación | Comando de configuración |
|---|---|---|
| Claude Code | `claude` | `raigal agent connect claude` |
| OpenCode | `opencode` | `raigal agent connect opencode` |
| Codex | `codex` | `raigal agent connect codex` |
| Detección automática | Selección automática | `raigal agent use auto` |

Compruebe los proveedores disponibles en su máquina:
```bash
raigal agent providers
```

## Flujos de trabajo

### 1. Sesión de reparación interactiva local

Pruebe y valide las modificaciones antes de crear un commit en git:

```bash
# Previsualizar qué abordará la sesión de reparación
raigal agent plan .

# Iniciar la sesión interactiva de reparación
raigal agent --provider opencode .

# Consultar sesiones de reparación activas y recientes
raigal agent sessions .

# Examinar los cambios introducidos y diagnósticos de una sesión
raigal agent show <session-id>

# Aplicar las modificaciones aprobadas a la rama activa
raigal agent apply <session-id>
```

### 2. Creación automática de pull requests (`--pr`)

Para generar una rama y abrir un pull request directamente:

```bash
raigal agent --provider claude --pr --target 95
```

Con la opción `--pr`, Raigal:
1. Aplica arreglos seguros y traslada los problemas restantes al agente.
2. Comprueba que las pruebas pasan y la puntuación alcanza `--target` (por defecto: 90).
3. Confirma los cambios en una rama aislada (`raigal/repair-...`).
4. Publica la rama en el repositorio remoto y abre un pull request en GitHub.
5. Envía la telemetría a Raigal Cloud para crear una tarjeta en la pestaña **Agent Actions**.

### 3. Sincronización del ciclo de vida (`report-pr-closed`)

Cuando se fusiona o cierra un pull request en GitHub, actualice el estado del tablero mediante un flujo de GitHub Actions:

```yaml
# .github/workflows/raigal-pr-closed.yml
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
      - name: Reportar Cierre de PR a Raigal Cloud
        env:
          RAIGAL_TOKEN: ${{ secrets.RAIGAL_TOKEN }}
          RAIGAL_API_URL: ${{ secrets.RAIGAL_API_URL }}
        run: |
          npx -y @methiu/raigal@0.2.2 report-pr-closed --pr ${{ github.event.pull_request.number }} ${{ github.event.pull_request.merged && '--merged' || '' }}
```

{% hint style="success" %}
Al abrir un pull request mediante `raigal agent --pr`, se crea una tarjeta en la pestaña **Agent Actions**. Cuando se fusiona el PR, `report-pr-closed` mueve la tarjeta a **Merged** y registra la diferencia de puntuación.
{% endhint %}
