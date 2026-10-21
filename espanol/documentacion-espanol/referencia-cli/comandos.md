---
description: "Referencia de comandos y opciones de la interfaz de línea de comandos de Raigal."
icon: terminal
---

# Comandos y opciones del CLI

Raigal incluye comandos para análisis locales, correcciones automáticas, sesiones de reparación con agentes y validación en CI.

## Índice de comandos

| Comando | Acción |
|---|---|
| `raigal [dir]` | Menú interactivo en terminal o escaneo predeterminado |
| `raigal scan [dir]` | Ejecuta el escaneo de calidad y devuelve la puntuación |
| `raigal fix [dir]` | Aplica correcciones automáticas deterministas o crea prompts para agentes |
| `raigal ci [dir]` | Puerta de calidad con códigos de salida por umbral en CI/CD |
| `raigal agent [dir]` | Sesiones de reparación en worktrees aislados (Claude, Cursor, OpenCode) |
| `raigal hook` | Gestiona hooks en tiempo real para agentes de IA |
| `raigal report-pr-closed` | Sincroniza el ciclo de vida (cierre/fusión) de Pull Requests con Raigal Cloud |
| `raigal login` | Inicio de sesión interactivo vía navegador OAuth o token API |
| `raigal logout` | Elimina credenciales locales y concesiones offline en caché |
| `raigal whoami` | Muestra credenciales activas, usuario, organización y repositorios permitidos |
| `raigal doctor` | Comprueba binarios del sistema, herramientas y motores |
| `raigal rules` | Muestra el catálogo de reglas y pesos de puntuación |
| `raigal init` | Crea el archivo de configuración y flujo opcional para CI |

## Referencia de comandos

### `raigal scan`

Evalúa el directorio indicado y muestra el desglose de calidad.

```bash
raigal scan [opciones] [ruta]
```

Opciones:
- `--changes`: analiza únicamente los archivos modificados respecto a git `HEAD`.
- `--staged`: analiza únicamente los archivos en el stage de git (ideal para pre-commit).
- `--base <ref>`: compara el diff contra una referencia específica (ej: `origin/main`).
- `-d, --verbose`: muestra el desglose detallado de archivos por cada regla.
- `--json`: emite el árbol completo de diagnósticos en formato JSON.
- `--sarif`: genera diagnósticos en estándar OASIS SARIF 2.1.0.
- `--exclude <patrones>`: lista de rutas separadas por comas o repetidas a excluir.
- `--include <patrones>`: lista de rutas separadas por comas o repetidas a incluir.
- `--fail-on <nivel>`: finaliza con código de salida 1 según el nivel: `none`, `error` o `warning`.

### `raigal ci`

Ejecuta el control de calidad en entornos de CI y devuelve código de salida 1 si no se superan los requisitos.

```bash
raigal ci [opciones] [ruta]
```

- Devuelve código de salida 1 si se detecta cualquier aviso de nivel `error` o si la puntuación global es inferior a `ci.failBelow` (por defecto: 70).
- `--changes`: valida únicamente los archivos modificados respecto a `--base` (o `HEAD`).
- `--staged`: valida únicamente archivos en stage.
- `--base <ref>`: referencia base para el diff de `--changes` (ej: `origin/main`).
- `--human`: genera resúmenes legibles en la salida de consola en lugar de JSON.
- `--sarif`: emite informe SARIF 2.1.0 por stdout para anotaciones en GitHub Code Scanning (ej: `raigal ci --sarif > resultados.sarif`).
- `--format <formato>`: formato de salida: `json` (por defecto) o `sarif`.

*(Nota: El reporte al panel de Raigal Cloud se ejecuta automáticamente en entornos CI autenticados mediante OIDC o `RAIGAL_TOKEN`).*

### `raigal fix`

Aplica correcciones automáticas para formateo, importaciones no utilizadas y comentarios narrativos.

```bash
raigal fix [opciones] [ruta]
```

- `--safe`: aplica únicamente cambios deterministas que preservan la semántica (por defecto).
- `-f, --force`: ejecuta correcciones más amplias, como auditorías de dependencias y limpieza de archivos muertos.
- `--dry-run`: previsualiza las modificaciones sin escribir en disco.
- `-p, --prompt`: convierte los problemas no corregibles en un prompt estructurado para su agente de programación.
- `--changes` / `--staged`: limita las correcciones a archivos modificados o en stage.
- `--base <ref>`: referencia base para `--changes`.

### `raigal agent`

Ejecuta sesiones de reparación con herramientas de IA locales en un git worktree aislado.

```bash
raigal agent [opciones] [ruta]
```

Opciones principales:
- `--provider <proveedor>`: proveedor a invocar (`auto`, `codex`, `claude`, `opencode`).
- `--target <n>`: umbral de puntuación objetivo a alcanzar (por defecto: 90).
- `--max-turns <n>`: número máximo de turnos del agente por intento de reparación (por defecto: 5).
- `--max-cost <usd>`: tope presupuestario en dólares que detiene la sesión si el consumo de tokens supera el límite.
- `--limit <n>`: número máximo de hallazgos entregados al agente (por defecto: 8).
- `--in-place`: edita directamente en el árbol de trabajo actual en lugar de aislar en worktree.
- `--apply`: aplica los cambios aceptados a la rama original al finalizar.
- `-y, --yes`: omite confirmaciones interactivas para `--apply`.
- `--dry-run`: muestra el proveedor seleccionado y el plan de ejecución sin realizar cambios.
- `--background`: inicia la ejecución del agente en segundo plano y retorna de inmediato.
- `--no-fix`: omite los arreglos automáticos seguros antes de invocar al agente.
- `--commit`: genera un commit con los cambios verificados en una rama de reparación.
- `--pr`: publica la rama y abre un pull request en GitHub.
- `--branch <nombre>`: nombre de la rama para `--commit` o `--pr`.
- `--base <rama>`: rama base de destino para `--pr`.
- `--title <titulo>`: título del pull request para `--pr`.
- `--commit-message <mensaje>`: mensaje de commit.
- `--ready`: abre un pull request listo para revisión en lugar de un borrador.
- `--cleanup`: elimina el worktree generado aunque quede algún cambio pendiente.

Subcomandos:
- `raigal agent plan [dir]`: previsualiza proveedor, worktree, hallazgos y acciones sin editar archivos.
- `raigal agent connect [proveedor]`: conecta con un proveedor local mediante su propia autenticación CLI.
- `raigal agent providers`: lista proveedores detectados localmente y su estado.
- `raigal agent use [proveedor]`: define o muestra el proveedor por defecto para este repositorio.
- `raigal agent sessions [dir]`: lista sesiones de reparación activas y anteriores.
- `raigal agent show <session-id>`: inspecciona diff y metadatos de una sesión en particular.
- `raigal agent apply <session-id>`: fusiona los cambios de una sesión en su rama de trabajo.
- `raigal agent monitor [dir]`: supervisa cambios en git y realiza auditorías en segundo plano (use `--auto-fix` para permitir ediciones).

### `raigal hook`

Gestiona hooks de intercepción para Claude Code, Cursor, Gemini CLI y otras herramientas.

```bash
raigal hook <subcomando>
```

Subcomandos:
- `raigal hook install [agentes...]`: instala hooks de intercepción (ej: `raigal hook install --claude`).
- `raigal hook status`: inspecciona hooks registrados y la integridad de los centinelas SHA-256.
- `raigal hook baseline`: guarda una instantánea de los hallazgos actuales en `.raigal/baseline.json`.
- `raigal hook uninstall [agentes...]`: elimina hooks de forma segura preservando la configuración del editor.

### `raigal report-pr-closed`

Notifica el cierre o fusión de un pull request a Raigal Cloud para actualizar el tablero de remediación.

```bash
raigal report-pr-closed [--pr <numero>] [--merged]
```

- `--pr <numero>`: número del pull request (obtenido automáticamente de `GITHUB_EVENT_PATH` en GitHub Actions).
- `--merged`: indica si el pull request se fusionó en la rama base.

### `raigal login` y `raigal logout`

Gestionan las credenciales de Raigal Cloud y la autenticación mediante licencias fuera de línea.

```bash
# Inicio de sesión interactivo vía navegador OAuth
raigal login

# Inicio de sesión automatizado vía token recibido en stdin
echo "rgl_live_..." | raigal login --with-token

# Limpiar credenciales locales y caché offline
raigal logout
```

### `raigal whoami`

Muestra el usuario autenticado, organización, plan y repositorios de GitHub con acceso autorizado.

```bash
raigal whoami
```

### `raigal doctor`

Verifica el entorno del sistema, formateadores, linters y binarios de agentes de programación.

```bash
raigal doctor
```

### `raigal rules`

Muestra el catálogo de reglas registradas, niveles de severidad, disponibilidad de corrección automática y niveles de impacto.

```bash
raigal rules
```
