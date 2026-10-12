---
description: "Cómo calcula Raigal la puntuación de calidad 0-100, deducciones ponderadas y compresión logarítmica."
icon: gauge
---

# Motor de puntuación

Raigal evalúa el código en una escala de 0 a 100, donde 100 indica ausencia total de incidencias. En lugar de restar puntos fijos arbitrarios o utilizar métricas de líneas de código, el motor normaliza las incidencias según el número de archivos fuente, aplica topes de saturación por regla y comprime las deducciones de forma logarítmica.

## Cálculo de deducciones

Cada aviso genera una deducción base calculada a partir de su severidad, la categoría de motor y el nivel de impacto de la regla:

$$\text{deducción} = \text{penalizaciónPorSeveridad} \times \text{pesoDelMotor} \times \text{multiplicadorDeImpacto}$$

### Penalizaciones por severidad

| Severidad | Penalización | Descripción |
|---|---|---|
| `error` | 3.0 | Fallos de compilación, vulnerabilidades de seguridad, errores de tipo o violaciones graves |
| `warning` | 1.0 | Conversiones inseguras, complejidad excesiva o advertencias de mantenimiento |
| `info` / `advisory` | 0.25 | Sugerencias de estilo y notas informativas |

### Ponderación de motores

Cada motor de diagnóstico aplica un multiplicador de categoría acorde a su nivel de riesgo:

| Motor | Peso | Alcance |
|---|---|---|
| `security` | 1.5 | Secretos expuestos, vulnerabilidades de inyección y llamadas peligrosas |
| `ai-slop` | 1.0 | Comprobaciones nulas redundantes, polyfills inventados, comentarios obvios y patrones de IA |
| `architecture` | 1.0 | Límites entre capas y dependencias circulares entre módulos |
| `code-quality` | 0.8 | Exportaciones sin uso (Knip), código muerto y límites de complejidad |
| `lint` | 0.6 | Análisis estático con Oxlint, Ruff y diagnósticos de compilador |
| `format` | 0.3 | Discrepancias de formato, espacios e importaciones desordenadas |

### Niveles de impacto y límites por regla

Cada regla pertenece a un nivel de impacto que define su multiplicador y su tope de deducción por defecto:

| Nivel | Multiplicador | Tope por defecto | Tipos de hallazgos habituales |
|---|---|---|---|
| `strict` | $1.0\times$ | Sin tope | Errores de tipado, fallos sintácticos y credenciales filtradas |
| `standard` | $1.0\times$ | Sin tope | Diagnósticos estándar de linter y comprobaciones de defectos fiables |
| `maintainability` | $0.75\times$ | 24 | Complejidad ciclomática alta y funciones excesivamente largas |
| `mechanical` | $0.5\times$ | 16 | Código muerto menor y variables locales no referenciadas |
| `style` | $0.5\times$ | 8 | Variaciones de formato y discrepancias de nombres |
| `advisory` | $0.25\times$ | 8 | Comentarios redundantes y avisos informativos |

### Límite dinámico por regla

Para evitar que una única regla repetitiva hunda la puntuación de todo el proyecto, las deducciones máximas de cada regla escalan de forma proporcional al tamaño del repositorio:

$$\text{escalaDeTope} = \max\left(1, \frac{\text{recuentoArchivos}}{10}\right)$$

La deducción acumulada de una regla individual queda acotada por:

$$\text{topeEfectivo} = \min(\text{maxPerRule}, \text{topeRegla}) \times \text{escalaDeTope}$$

Por defecto, `maxPerRule` es 40. En un proyecto con 50 archivos fuente, `escalaDeTope` es 5.0, por lo que una regla consultiva con tope base de 8 no podrá restar más de 40 puntos en todo el repositorio.

## Densidad de deducción y escalado

Las deducciones se suman entre todas las reglas y se normalizan frente al volumen del proyecto:

$$\text{densidadDeDeducción} = \frac{\sum \text{deduccionesAcotadas}}{\text{recuentoArchivos} + \text{constanteDeSuavizado}}$$

- `recuentoArchivos`: Total de archivos fuente analizados. Si el análisis se ejecuta sobre diagnósticos prefiltrados sin lista de archivos, Raigal toma como referencia los archivos con avisos observados.
- `constanteDeSuavizado`: Por defecto es 5 (con reserva interna de 10 en cálculos directos). Este factor evita penalizaciones desproporcionadas en repositorios pequeños o cambios de un solo archivo.
- Factor de escala: La densidad se multiplica por un factor de calibración de 5 para obtener las deducciones escaladas finales:

$$\text{deduccionesEscaladas} = \text{densidadDeDeducción} \times 5$$

## Compresión logarítmica de la puntuación

La puntuación final comprime las deducciones escaladas en un rango estable de 0 a 100:

$$\text{puntuación} = \max\left(0, \text{redondear}\left(100 - \frac{100 \times \ln(1 + \text{deduccionesEscaladas})}{\ln(1 + 100 + \text{deduccionesEscaladas})}\right)\right)$$

Esta curva matemática proporciona ventajas concretas:
- Un repositorio con 0 incidencias obtiene exactamente 100 puntos.
- La penalización marginal es mayor en las primeras incidencias, premiando a los proyectos limpios por mantener su estándar.
- Los repositorios heredados con cientos de incidencias se aproximan de forma asintótica a 0, garantizando que la puntuación nunca sea negativa sin recurrir a recortes artificiales.

## Umbrales de puntuación y rangos de estado

Raigal utiliza exclusivamente puntuaciones numéricas de 0 a 100. No se emplean calificaciones por letras (A–E).

### Etiquetas de estado en el CLI

Al ejecutar `raigal scan` o `raigal ci`, la interfaz de línea de comandos asigna etiquetas según los umbrales configurados (`good: 75`, `ok: 50`):

| Rango de puntuación | Etiqueta en CLI | Código de salida en CI |
|---|---|---|
| $75 - 100$ | `Healthy` | `0` (Aprobado) |
| $50 - 74$ | `Needs Work` | `0` (o `1` si se activa `--fail-on` o `failBelow`) |
| $< 50$ | `Critical` | `1` (Falla el control de calidad por defecto) |

En entornos de integración continua, `raigal ci` devuelve código de salida 1 si la puntuación es inferior a `ci.failBelow` (por defecto: 70) o si se detecta algún aviso con severidad `error`.

### Rangos de salud en el panel web

La interfaz web de Raigal Cloud (`app.raigal.dev`) agrupa los repositorios en cinco bandas de salud:

| Rango de puntuación | Banda en panel web | Color identificativo |
|---|---|---|
| $90 - 100$ | `Excellent` | Verde lima |
| $80 - 89$ | `Good` | Verde esmeralda |
| $70 - 79$ | `Fair` | Ámbar / amarillo |
| $60 - 69$ | `Poor` | Naranja |
| $< 60$ | `Critical` | Rojo |
