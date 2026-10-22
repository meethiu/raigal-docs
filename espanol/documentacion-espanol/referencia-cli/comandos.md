---
description: "Referencia de comandos y opciones de la interfaz de linea de comandos de Raigal."
icon: terminal
---

# Comandos y opciones del CLI

Raigal incluye comandos para analisis locales, correcciones automaticas, sesiones de reparacion con agentes y validacion en CI.

## Indice de comandos

| Comando | Accion |
|---|---|
| `raigal [dir]` | Menu interactivo en terminal o escaneo predeterminado |
| `raigal scan [dir]` | Ejecuta el escaneo de calidad y devuelve la puntuacion |
| `raigal fix [dir]` | Aplica correcciones automaticas deterministas o crea prompts para agentes |
| `raigal ci [dir]` | Puerta de calidad con codigos de salida por umbral en CI/CD |
| `raigal agent [dir]` | Sesiones de reparacion en worktrees aislados (Claude, Cursor, OpenCode) |
| `raigal hook` | Gestiona hooks en tiempo real para agentes de IA |
| `raigal report-pr-closed` | Sincroniza el ciclo de vida (cierre/fusion) de Pull Requests con Raigal Cloud |
| `raigal telemetry` | Inspecciona el payload enviado al panel cloud y el estado de la cola de salida |
| `raigal login` | Inicio de sesion interactivo via navegador OAuth o token API |
| `raigal logout` | Elimina credenciales locales y concesiones offline en cache |
| `raigal whoami` | Muestra credenciales activas, usuario, organizacion, estado de prueba/licencia y repositorios |
| `raigal doctor` | Comprueba binarios del sistema, herramientas y motores |
| `raigal rules` | Muestra el catalogo de reglas y pesos de puntuacion |
| `raigal init` | Crea el archivo de configuracion y flujo opcional para CI |
| `raigal mcp` | Ejecuta el servidor Model Context Protocol (MCP) mediante stdio |

## Referencia de comandos

### `raigal scan`

Evalua el directorio indicado y muestra el desglose de calidad.

```bash
raigal scan [opciones] [ruta]
```

Opciones:
- `--changes`: analiza unicamente los archivos modificados respecto a git `HEAD`.
- `--staged`: analiza unicamente los archivos en el stage de git (ideal para pre-commit).
- `--base <ref>`: compara el diff contra una referencia especifica (ej: `origin/main`).
- `-d, --verbose`: muestra el desglose detallado de archivos por cada regla.
- `--json`: emite el arbol completo de diagnosticos en formato JSON.
- `--sarif`: genera diagnosticos en estandar OASIS SARIF 2.1.0.
- `--exclude <patrones>`: lista de rutas separadas por comas o repetidas a excluir.
- `--include <patrones>`: lista de rutas separadas por comas o repetidas a incluir.
- `--fail-on <nivel>`: finaliza con codigo de salida 1 segun el nivel: `none`, `error` o `warning`.

### `raigal ci`

Ejecuta el control de calidad en entornos de CI y devuelve codigo de salida 1 si no se superan los requisitos.

```bash
raigal ci [opciones] [ruta]
```

- Devuelve codigo de salida 1 si se detecta cualquier aviso de nivel `error` o si la puntuacion global es inferior a `ci.failBelow` (por defecto: 70).
- `--changes`: valida unicamente los archivos modificados respecto a `--base` (o `HEAD`).
- `--staged`: valida unicamente archivos en stage.
- `--base <ref>`: referencia base para el diff de `--changes` (ej: `origin/main`).
- `--human`: genera resumenes legibles en la salida de consola en lugar de JSON.
- `--sarif`: emite informe SARIF 2.1.0 por stdout para anotaciones en GitHub Code Scanning (ej: `raigal ci --sarif > resultados.sarif`).
- `--format <formato>`: formato de salida: `json` (por defecto) o `sarif`.

*(Nota: El reporte al panel de Raigal Cloud se ejecuta automaticamente en entornos CI autenticados mediante OIDC o `RAIGAL_TOKEN`).*

### `raigal fix`

Aplica correcciones automaticas para formateo, importaciones no utilizadas y comentarios narrativos.

```bash
raigal fix [opciones] [ruta]
```

- `--safe`: aplica unicamente cambios deterministas que preservan la semantica (por defecto).
- `-f, --force`: ejecuta correcciones mas amplias, como auditorias de dependencias y limpieza de archivos muertos.
- `--dry-run`: previsualiza las modificaciones sin escribir en disco.
- `-p, --prompt`: convierte los problemas no corregibles en un prompt estructurado para su agente de programacion.
- `--changes` / `--staged`: limita las correcciones a archivos modificados o en stage.
- `--base <ref>`: referencia base para `--changes`.

### `raigal agent`

Ejecuta sesiones de reparacion con herramientas de IA locales en un git worktree aislado.

```bash
raigal agent [opciones] [ruta]
```

Opciones principales:
- `--provider <proveedor>`: proveedor a invocar (`auto`, `codex`, `claude`, `opencode`).
- `--target <n>`: umbral de puntuacion objetivo a alcanzar (por defecto: 90).
- `--max-turns <n>`: numero maximo de turnos del agente por intento de reparacion (por defecto: 5).
- `--max-cost <usd>`: tope presupuestario en dolares que detiene la sesion si el consumo de tokens supera el limite.
- `--limit <n>`: numero maximo de hallazgos entregados al agente (por defecto: 8).
- `--in-place`: edita directamente en el arbol de trabajo actual en lugar de aislar en worktree.
- `--apply`: aplica los cambios aceptados a la rama original al finalizar.
- `-y, --yes`: omite confirmaciones interactivas para `--apply`.
- `--dry-run`: muestra el proveedor seleccionado y el plan de ejecucion sin realizar cambios.
- `--background`: inicia la ejecucion del agente en segundo plano y retorna de inmediato.
- `--no-fix`: omite los arreglos automaticos seguros antes de invocar al agente.
- `--commit`: genera un commit con los cambios verificados en una rama de reparacion.
- `--pr`: publica la rama y abre un pull request en GitHub.
- `--branch <nombre>`: nombre de la rama para `--commit` o `--pr`.
- `--base <rama>`: rama base de destino para `--pr`.
- `--title <titulo>`: titulo del pull request para `--pr`.
- `--commit-message <mensaje>`: mensaje de commit.
- `--ready`: abre un pull request listo para revision en lugar de un borrador.
- `--cleanup`: elimina el worktree generado aunque quede algun cambio pendiente.

Subcomandos:
- `raigal agent plan [dir]`: previsualiza proveedor, worktree, hallazgos y acciones sin editar archivos.
- `raigal agent connect [proveedor]`: conecta con un proveedor local mediante su propia autenticacion CLI.
- `raigal agent providers`: lista proveedores detectados localmente y su estado.
- `raigal agent use [proveedor]`: define o muestra el proveedor por defecto para este repositorio.
- `raigal agent sessions [dir]`: lista sesiones de reparacion activas y anteriores.
- `raigal agent show <session-id>`: inspecciona diff y metadatos de una sesion en particular.
- `raigal agent apply <session-id>`: fusiona los cambios de una sesion en su rama de trabajo.
- `raigal agent monitor [dir]`: supervisa cambios en git y realiza auditorias en segundo plano (use `--auto-fix` para permitir ediciones).

### `raigal hook`

Gestiona hooks de intercepcion para Claude Code, Cursor, Gemini CLI y otras herramientas.

```bash
raigal hook <subcomando>
```

Subcomandos:
- `raigal hook install [agentes...]`: instala hooks de intercepcion (ej: `raigal hook install --claude`).
- `raigal hook status`: inspecciona hooks registrados y la integridad de los centinelas SHA-256.
- `raigal hook baseline`: guarda una instantanea de los hallazgos actuales en `.raigal/baseline.json`.
- `raigal hook uninstall [agentes...]`: elimina hooks de forma segura preservando la configuracion del editor.

### `raigal report-pr-closed`

Notifica el cierre o fusion de un pull request a Raigal Cloud para actualizar el tablero de remediacion.

```bash
raigal report-pr-closed [--pr <numero>] [--merged]
```

- `--pr <numero>`: numero del pull request (obtenido automaticamente de `GITHUB_EVENT_PATH` en GitHub Actions).
- `--merged`: indica si el pull request se fusiono en la rama base.

### `raigal login` y `raigal logout`

Gestionan las credenciales de Raigal Cloud y la autenticacion mediante licencias fuera de linea.

```bash
# Inicio de sesion interactivo via navegador OAuth
raigal login

# Inicio de sesion automatizado via token recibido en stdin
echo "rgl_live_..." | raigal login --with-token

# Limpiar credenciales locales y cache offline
raigal logout
```

### `raigal whoami`

Muestra el usuario autenticado, organizacion, estado de prueba o licencia y repositorios de GitHub con acceso autorizado.

```bash
raigal whoami
```

### `raigal doctor`

Verifica el entorno del sistema, formateadores, linters y binarios de agentes de programacion.

```bash
raigal doctor
```

### `raigal rules`

Muestra el catalogo de reglas registradas, niveles de severidad, disponibilidad de correccion automatica y niveles de impacto.

```bash
raigal rules
```
