---
description: "Configuración de reglas del proyecto, umbrales y motores mediante .raigal/config.yml."
icon: gear
---

# Configuración

Raigal lee su configuración del archivo `.raigal/config.yml` en la raíz del repositorio.

## Configuración mínima

```yaml
version: 1

scoring:
  threshold: 85
  fail_under: 80

engines:
  format: true
  lint: true
  quality: true
  ai_slop: true
  security: true
  architecture: true

ignore:
  - "dist/**"
  - "node_modules/**"
  - "**/*.min.js"
```

## Referencia de opciones

### `scoring`

- `threshold` (numérico, por defecto: 80): puntuación de referencia deseada para el repositorio.
- `fail_under` (numérico, por defecto: 80): puntuación mínima exigida en CI (`raigal ci`). Cualquier valor inferior devuelve código de salida `1`.

### `engines`

Activa o desactiva módulos de diagnóstico individuales:
- `ai_slop`: patrones repetitivos, comprobaciones defensivas innecesarias y comentarios obvios.
- `security`: secretos expuestos, claves privadas y llamadas a funciones vulnerables.
- `lint`: análisis estático con Oxlint y Ruff.
- `format`: comprobación de estilo con Biome, Gofmt y Ruff.
- `quality`: detección de código sin uso (Knip) y límites de complejidad ciclomática.
- `architecture`: límites de arquitectura e importaciones entre capas.

### `telemetry`

Para desactivar el envío de telemetría a la nube:

```yaml
telemetry:
  enabled: false
```

Los eventos de telemetría solo incluyen métricas numéricas y recuentos de reglas. El contenido del código y de los archivos nunca se transmite.
