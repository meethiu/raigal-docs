---
description: "Configuración de motores, umbrales de puntuación y rutas en .raigal/config.yml."
icon: gear
---

# Configuración

Raigal lee las preferencias de su proyecto del archivo `.raigal/config.yml` en la raíz del repositorio. El archivo sigue un esquema estricto validado mediante Zod; cualquier propiedad no reconocida o mal escrita produce un error de validación inmediato.

## Ejemplo completo de configuración

A continuación se muestra un archivo `.raigal/config.yml` completo con el esquema válido:

```yaml
version: 1

# Supported engines
engines:
  format: true          # Biome, Ruff, Gofmt, Cargo fmt, etc.
  lint: true            # Oxlint, Ruff, Clippy, Golangci-lint, Roslynator, etc.
  code-quality: true    # Knip unused exports, dead code, complexity (NOT "quality")
  ai-slop: true         # 50+ AI anti-pattern detection rules (NOT "ai_slop")
  architecture: false   # Dependency & boundary rules
  security: true        # Secrets, injections, unsafe deserialization

# Quality and complexity limits
quality:
  maxFunctionLoc: 80
  maxFileLoc: 400
  maxNesting: 5
  maxParams: 6

# Engine-specific lint settings
lint:
  typecheck: false      # Bundled tsc compiler diagnostics
  expoDoctor: false     # Expo React Native project hygiene checks
  csharp:
    projectEvaluation: false
    jb: true            # JetBrains InspectCode
    roslynator: true
    jbSeverityFloor: "WARNING"
  cpp:
    cppcheck: true
    clangTidy: true
    cppcheckEnable: "warning,performance,portability"

# CI gate configuration
ci:
  failBelow: 70         # Default score threshold (NOT scoring.fail_under)
  format: "json"

# Score calculation tuning
scoring:
  thresholds:
    good: 75
    ok: 50
  smoothing: 5
  maxPerRule: 40

# Rule severity overrides
rules:
  ai-slop/trivial-comment: "off"
  ai-slop/unsafe-type-assertion: "error"

# Path exclusions (NOT "ignore")
exclude:
  - "node_modules"
  - ".git"
  - "dist"
  - "build"
  - "coverage"
include: []
```

## Ubicación de archivos y prioridad

Raigal busca el archivo de configuración recorriendo directorios hacia arriba hasta la raíz del repositorio:
1. `.raigal/config.yml` o `.raigal/config.yaml`
2. `.raigal/config.json`
3. Archivos en raíz: `.raigal.yml`, `.raigal.yaml` o `.raigal.json`
4. Directorios heredados de compatibilidad (`.aislop/config.yml`, `.aislop.yml`)

## Referencia de opciones

### `engines`

Habilita o deshabilita motores de análisis individuales:

- `format` (booleano, por defecto: `true`): espacios en blanco, sangrado, ajuste de líneas y orden de importaciones.
- `lint` (booleano, por defecto: `true`): reglas de análisis estático con Oxlint, Ruff, Clippy y herramientas del lenguaje.
- `code-quality` (booleano, por defecto: `true`): exportaciones sin uso (Knip), código muerto y límites de complejidad. Tenga en cuenta que la clave es `code-quality`, no `quality`.
- `ai-slop` (booleano, por defecto: `true`): más de 50 reglas que detectan patrones defectuosos de IA, comentarios triviales y wrappers redundantes. La clave es `ai-slop`, no `ai_slop`.
- `architecture` (booleano, por defecto: `false`): límites arquitectónicos e importaciones cruzadas entre capas.
- `security` (booleano, por defecto: `true`): secretos expuestos, inyecciones de código y llamadas a funciones peligrosas.

### `quality`

Define límites estructurales evaluados por el motor `code-quality`:

- `maxFunctionLoc` (numérico, por defecto: `80`): máximo de líneas físicas permitidas por función.
- `maxFileLoc` (numérico, por defecto: `400`): máximo de líneas físicas permitidas por archivo.
- `maxNesting` (numérico, por defecto: `5`): profundidad máxima de bloques de control anidados.
- `maxParams` (numérico, por defecto: `6`): número máximo de parámetros permitidos por función.

### `lint`

Configura herramientas de compilación y diagnóstico específicas del lenguaje:

- `typecheck` (booleano, por defecto: `false`): ejecuta diagnósticos del compilador de TypeScript (`tsc`).
- `expoDoctor` (booleano, por defecto: `false`): ejecuta comprobaciones de higiene en proyectos Expo React Native.
- `csharp`: opciones para C#, incluyendo `roslynator` y JetBrains InspectCode (`jb`).
- `cpp`: opciones para C/C++ (`cppcheck`, `clangTidy`).

### `ci`

Configura el comportamiento del control de calidad en integración continua:

- `failBelow` (numérico, por defecto: `70`): puntuación mínima para aprobar la comprobación. Si la puntuación es inferior a este valor o se detecta algún aviso de nivel `error`, `raigal ci` devuelve código de salida 1. La clave es `ci.failBelow`, no `scoring.fail_under`.
- `format` (`"json"`, por defecto: `"json"`): formato de salida en ejecuciones de CI.

### `scoring`

Permite calibrar los parámetros del motor de puntuación:

- `thresholds`: objeto que define las fronteras de estado en el CLI.
  - `good` (numérico, por defecto: `75`): puntuación a partir de la cual el estado se etiqueta como `Healthy`.
  - `ok` (numérico, por defecto: `50`): puntuación a partir de la cual se etiqueta como `Needs Work`. Por debajo se considera `Critical`.
- `smoothing` (numérico, por defecto: `5`): constante de suavizado sumada al recuento de archivos para evitar caídas bruscas en repositorios pequeños.
- `maxPerRule` (numérico, por defecto: `40`): límite base de deducción para cualquier regla antes de aplicar el multiplicador de escala del proyecto.
- `weights`: ponderaciones personalizadas por categoría (por defecto: `security: 1.5`, `ai-slop: 1.0`, `architecture: 1.0`, `code-quality: 0.8`, `lint: 0.6`, `format: 0.3`).

### `rules`

Mapa de identificadores de reglas a modificaciones de severidad:

```yaml
rules:
  ai-slop/trivial-comment: "off"
  ai-slop/unsafe-type-assertion: "error"
```

Los valores válidos de severidad son `"error"`, `"warning"` y `"off"`.

### `exclude` e `include`

Define el ámbito de archivos analizados en el sistema:

- `exclude` (lista de cadenas): rutas y directorios que se omiten. Por defecto: `["node_modules", ".git", "dist", "build", "coverage"]`. La clave es `exclude`, no `ignore`.
- `include` (lista de cadenas): rutas explícitas a analizar. Cuando está vacía (`[]`), se analizan todos los archivos compatibles no excluidos.

### `overrides`

Aplica límites de calidad o severidades específicas a patrones de archivos determinados:

```yaml
overrides:
  - files: ["legacy/**/*.ts"]
    quality:
      maxFileLoc: 800
    rules:
      ai-slop/trivial-comment: "off"
```

### `telemetry`

Controla el envío de métricas anónimas a Raigal Cloud:

```yaml
telemetry:
  enabled: true
```

Los eventos de telemetría solo contienen recuentos de métricas numéricas, reglas activadas y tiempos de ejecución. El contenido del código fuente y de los archivos nunca se transmite.
