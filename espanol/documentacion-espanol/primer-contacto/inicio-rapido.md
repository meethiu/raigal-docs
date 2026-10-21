---
description: "Autentíquese, inicialice su configuración, ejecute un primer análisis y aplique correcciones automáticas."
icon: bolt
---

# Inicio rápido

Analice un repositorio y revise los resultados en cuatro pasos.

{% stepper %}
{% step %}
### 1. Autenticarse con su organización

Inicie sesión para vincular su terminal local con su organización de Raigal:

```bash
raigal login
```

O exporte el token de API de su organización:

```bash
export RAIGAL_TOKEN="rgl_live_su_token_aqui"
```
{% endstep %}

{% step %}
### 2. Inicializar la configuración

Cree su configuración y elija su pipeline de CI:

```bash
raigal init
```

Este asistente interactivo detecta su entorno y genera:
- `.raigal/config.yml`: umbrales de puntuación, activación de motores y rutas excluidas (`exclude`).
- `.raigal/rules.yml`: reglas de arquitectura y ajustes de severidad.
- Flujo de CI (GitHub Actions, GitLab CI o Bitbucket Pipelines).
{% endstep %}

{% step %}
### 3. Ejecutar el análisis

Ejecute el comando de escaneo en la raíz de su repositorio:

```bash
raigal scan .
```

Raigal ejecuta en paralelo los motores de formateo, linting, calidad de código, patrones de IA y seguridad, mostrando la puntuación obtenida (0-100):

```text
   94 / 100  Healthy       0 errors  ·  1 warning  ·  1 fixable
   12 files  ·  6 engines  ·  48ms

 Top findings
   #  Finding                 Rule                       Status
   1  console.log leftover    ai-slop/console-leftover   warn [auto]
```
{% endstep %}

{% step %}
### 4. Aplicar correcciones

Aplique correcciones automáticas seguras para problemas de formato, comentarios triviales e importaciones sin uso:

```bash
raigal fix .
```

Para cambios estructurales que requieran edición semántica, ejecute `raigal agent .` para iniciar una sesión de reparación en un worktree aislado con Claude Code, Cursor u OpenCode.
{% endstep %}
{% endstepper %}
