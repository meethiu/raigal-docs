---
description: "Ejecute un primer análisis, revise los hallazgos y aplique correcciones automáticas."
icon: bolt
---

# Inicio rápido

Analice un repositorio y revise los resultados en tres pasos.

{% stepper %}
{% step %}
### 1. Inicializar la configuración

Genere los archivos de configuración iniciales:

```bash
raigal init .
```

Esto crea:
- `.raigal/config.yml`: umbrales de puntuación, activación de motores y rutas ignoradas.
- `.raigal/rules.yml`: ajustes de severidad para reglas concretas.
{% endstep %}

{% step %}
### 2. Ejecutar el análisis

Ejecute el comando de escaneo en la raíz de su repositorio:

```bash
raigal scan .
```

Raigal ejecuta en paralelo los motores de formateo, linting, calidad y patrones de IA, y muestra la puntuación obtenida (0-100):

```text
╭───────────────────────────────────────────────────╮
│  Puntuación de Calidad Raigal: 94 / 100 [Healthy] │
╰───────────────────────────────────────────────────╯
Desglose por categorías:
  • AI Slop:       98/100 (1 hallazgo)
  • Lint & Format: 92/100 (3 hallazgos)
  • Seguridad:     100/100 (Limpio)
```
{% endstep %}

{% step %}
### 3. Aplicar correcciones

Aplique correcciones automáticas para problemas de formato, comentarios innecesarios e importaciones sin uso:

```bash
raigal fix .
```

Para cambios que requieran edición semántica, ejecute `raigal agent .` para iniciar una sesión de reparación en un worktree aislado con Claude Code, Cursor u OpenCode.
{% endstep %}
{% endstepper %}
