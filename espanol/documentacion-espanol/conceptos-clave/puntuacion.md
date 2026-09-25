---
description: "Cómo Raigal calcula la puntuación 0-100, penalizaciones ponderadas y normalización por densidad."
icon: gauge
---

# Motor de puntuación

Raigal evalúa el código en una escala de 0 a 100, donde 100 indica que no se han detectado incidencias.

## Rangos de calidad

| Rango de puntuación | Estado | Significado | Código de salida en CI |
|---|---|---|---|
| 90-100 | Healthy | Arquitectura limpia, sin defectos críticos ni patrones defectuosos | `0` (Aprobado) |
| 75-89 | Warning | Formato mejorable, comentarios redundantes o deuda técnica leve | `0` (Aprobado salvo límite estricto) |
| 0-74 | Failing | Problemas estructurales, alta densidad de avisos o alertas de seguridad | `1` (Bloqueo en CI) |

## Fórmula de cálculo por densidad

En lugar de restar puntos fijos por cada aviso, Raigal pondera las penalizaciones según el volumen de código:

$$D = \frac{\sum (\text{peso}_i \times \text{severidad}_i)}{\text{KLOC}}$$

Donde:
- $\text{KLOC}$ representa el total de líneas de código físico dividido entre 1.000.
- Cada regla tiene un multiplicador de peso. Las alertas de seguridad restan más puntos, mientras que las deducciones por comentarios triviales se reducen de forma progresiva.
- Los archivos con menos de 100 líneas aplican un denominador base para no penalizar en exceso commits pequeños.

## Ponderaciones por categoría

- Seguridad y vulnerabilidades (1.5x): filtración de credenciales, inyecciones SQL y llamadas inseguras.
- Patrones de IA y código redundante (1.2x): comprobaciones nulas innecesarias, fallbacks artificiales y comentarios obvios.
- Arquitectura y límites (1.0x): dependencias circulares y referencias entre módulos no permitidas.
- Calidad de código y complejidad (0.9x): complejidad ciclomática alta y ramas de código muerto.
- Formato y estilo (0.6x): sangrado y discrepancias de espaciado.
