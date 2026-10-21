---
description: "Puntuación de calidad calibrada de 0 a 100, normalización por densidad y bandas de estado."
icon: gauge
---

# Motor de puntuación

Raigal asigna a cada repositorio una puntuación de calidad calibrada entre 0 y 100. La métrica evalúa la densidad de incidencias en lugar del recuento bruto de avisos, evitando que proyectos con bases de código grandes suspendan únicamente por su volumen de archivos.

## Fórmula de puntuación

Raigal evalúa las incidencias detectadas por sus seis motores de diagnóstico mediante un escalado logarítmico normalizado por densidad:

$$\text{Densidad de Deducción} = \frac{\sum \text{Deducciones Acotadas}}{\text{Archivos Fuente Efectivos} + S}$$

$$\text{Puntuación} = \max\left(0, \operatorname{redondear}\left(100 - \frac{100 \cdot \ln(1 + 5 \cdot \text{Densidad de Deducción})}{\ln(101 + 5 \cdot \text{Densidad de Deducción})}\right)\right)$$

Donde:
* **Constante de suavizado ($S$)**: Por defecto es `5` en la configuración (con reserva interna de `10`). Evita que un aviso aislado en un proyecto pequeño de dos archivos hunda la puntuación.
* **Escala de densidad**: Multiplicador de `5.0` aplicado sobre la densidad de deducción.
* **Archivos fuente efectivos**: Recuento de archivos de código y pruebas no excluidos en el espacio de trabajo (o archivos modificados durante escaneos incrementales de pull requests).

## Multiplicadores de severidad y ponderación de motores

Cada aviso de diagnóstico genera una penalización base según su severidad y el motor que lo detectó:

$$\text{Deducción Base} = \text{Multiplicador de Severidad} \times \text{Peso del Motor} \times \text{Multiplicador de Nivel}$$

### Multiplicadores de severidad
* `error`: $3.0$ (además activa código de salida 1 de inmediato en CI)
* `warning`: $1.0$
* `info`: $0.25$

### Ponderación de motores
* `security`: $1.5$ (secretos en código, inyecciones, APIs inseguras, dependencias vulnerables)
* `architecture`: $1.0$ (cruce indebido de capas, importaciones circulares)
* `ai-slop`: $1.0$ (excepciones silenciadas, comentarios triviales, aserciones de tipo laxas)
* `code-quality`: $0.8$ (código muerto, complejidad ciclomática excesiva, exceso de parámetros)
* `lint`: $0.6$ (Oxlint, Ruff, Clippy, Roslynator)
* `format`: $0.3$ (espacios en blanco, indentación, orden de importaciones)

## Topes dinámicos por regla

Para evitar que problemas de estilo muy repetidos (como 500 comas finales omitidas) oculten vulnerabilidades de seguridad o hundan proyectos grandes, las reglas pertenecen a niveles de impacto con topes base dinámicos:

| Nivel de impacto | Multiplicador | Tope base | Hallazgos habituales |
|---|---|---|---|
| `strict` | $1.0\times$ | Sin tope (`maxPerRule`) | Errores de sintaxis, secretos filtrados, promesas no controladas |
| `standard` | $1.0\times$ | Sin tope (`maxPerRule`) | Diagnósticos estándar de linter, advertencias de compilador |
| `maintainability` | $0.75\times$ | 24 | Complejidad ciclomática alta, funciones excesivamente largas |
| `mechanical` | $0.5\times$ | 16 | Importaciones no utilizadas, variables locales sin referencia |
| `style` | $0.5\times$ | 8 | Variaciones de formato, discrepancias de espaciado |
| `advisory` | $0.25\times$ | 8 | Avisos informativos, documentación faltante |

### Escalado según el tamaño del repositorio
El tope efectivo de cada regla escala proporcionalmente con el número de archivos:
$$\text{Escala de Tope} = \max\left(1, \frac{\text{Archivos Fuente Efectivos}}{10}\right)$$
$$\text{Tope Efectivo} = \min(\text{maxPerRule}, \text{Tope del Nivel}) \times \text{Escala de Tope}$$

## Ejemplos prácticos

### Ejemplo 1: Microservicio (10 archivos, 1 advertencia)
* **Hallazgo**: 1 `ai-slop/swallowed-exception` (`warning`, severidad $1.0$, motor $1.0$, nivel *strict* $1.0$).
* **Cálculo**:
  $$\text{Deducción} = 1.0, \quad \text{Densidad} = \frac{1.0}{10 + 5} = \frac{1.0}{15} \approx 0.0667$$
  $$\text{Densidad Escalada} = 0.0667 \times 5.0 = 0.3333$$
  $$\text{Puntuación} = 100 - \frac{100 \cdot \ln(1.3333)}{\ln(101.3333)} \approx 100 - 6.23 = \mathbf{94} \quad (\text{Healthy})$$

### Ejemplo 2: Monolito (200 archivos, 500 avisos de formato)
* **Hallazgos**: 500 comas finales omitidas (`format/trailing-comma`, `info` $0.25$, motor $0.3$, nivel *style* $0.5$, tope $8$).
* **Cálculo**:
  $$\text{Deducciones Brutas} = 500 \times 0.25 \times 0.3 \times 0.5 = 18.75$$
  $$\text{Escala de Tope} = \frac{200}{10} = 20, \quad \text{Tope Efectivo} = 8 \times 20 = 160$$
  $$\text{Deducciones Acotadas} = \min(18.75, 160) = 18.75$$
  $$\text{Densidad} = \frac{18.75}{200 + 5} = \frac{18.75}{205} \approx 0.0915, \quad \text{Densidad Escalada} = 0.4573$$
  $$\text{Puntuación} = 100 - \frac{100 \cdot \ln(1.4573)}{\ln(101.4573)} \approx 100 - 8.15 = \mathbf{92} \quad (\text{Healthy})$$

## Umbrales de puntuación y bandas de estado

Raigal utiliza exclusivamente puntuaciones numéricas de 0 a 100. No se emplean calificaciones por letras (A - F).

### Etiquetas de estado en el CLI
Al ejecutar `raigal scan` o `raigal ci`, las etiquetas de la terminal corresponden a los límites de puntuación configurados:

| Rango de puntuación | Etiqueta en CLI | Código de salida en CI |
|---|---|---|
| $75 - 100$ | `Healthy` | `0` (Aprobado) |
| $50 - 74$ | `Needs Work` | `0` (o `1` si se incumple `ci.failBelow`) |
| $< 50$ | `Critical` | `1` (Falla el control de calidad) |

En entornos de integración continua, `raigal ci` devuelve código de salida 1 si la puntuación es inferior a `ci.failBelow` (por defecto: 70) o si se detecta algún aviso con severidad `error`.

### Bandas de salud en el panel web
La interfaz de Raigal Cloud (`app.raigal.dev`) agrupa los repositorios en cuatro bandas de salud calibradas:

| Rango de puntuación | Banda en panel web | Color identificativo |
|---|---|---|
| $90 - 100$ | `Excellent` | Verde lima |
| $75 - 89$ | `Good / Healthy` | Verde esmeralda |
| $50 - 74$ | `Needs Work` | Ámbar / amarillo |
| $< 50$ | `Critical` | Rojo |

### Veredictos de calidad (3 estados)
Las puertas de CI y revisiones de PR evalúan las solicitudes en tres estados distintos:

| Veredicto | Criterios | Resultado del Check Run |
|---|---|---|
| `PASSED` | Puntuación >= 70 (o `ci.failBelow`), 0 errores, 0 advertencias | `success` (Aprobado sin advertencias) |
| `PASSED (WITH WARNINGS)` | Puntuación >= 70, 0 errores, advertencias presentes | `neutral` (Paso consultivo, no bloquea el merge) |
| `FAILED / BLOCKED` | Puntuación < 70, código de salida != 0, o errores presentes | `failure` (Bloquea el merge) |
