---
description: >-
  Control de calidad de código y detección de patrones de IA para equipos de
  ingeniería.
icon: shield-check
layout:
  width: default
  title:
    visible: true
  description:
    visible: true
  tableOfContents:
    visible: true
  outline:
    visible: true
  pagination:
    visible: true
  metadata:
    visible: true
  tags:
    visible: true
  actions:
    visible: true
  anchors:
    visible: true
---

# Visión general de Raigal

Raigal detecta patrones defectuosos introducidos por asistentes de programación de IA, como comentarios innecesarios, comprobaciones nulas redundantes, errores silenciados y llamadas inseguras. Reúne formateo, linting, análisis de complejidad, reglas de seguridad y detección de patrones en un único comando que devuelve una puntuación de 0 a 100.

Junto con [Raigal Cloud](https://app.raigal.dev) y la [GitHub App de Raigal](https://github.com/apps/raigal-cloud), evalúa pull requests, gestiona comprobaciones de estado para la protección de ramas, organiza las incidencias en un tablero de remediación y ejecuta reparaciones con agentes de programación.

{% hint style="success" %}
Raigal ejecuta el análisis de AST, las auditorías de seguridad y la detección de patrones dentro de su máquina local o runner de CI. El código fuente nunca se envía a los servidores de Raigal.
{% endhint %}

## Qué detecta Raigal

Los asistentes de programación como Claude Code, Cursor, Copilot u OpenCode aceleran el desarrollo, pero suelen introducir defectos concretos:

* Comentarios que solo repiten el nombre de funciones o variables.
* Comprobaciones defensivas sobre valores que el sistema de tipos ya garantiza.
* Bloques `catch` vacíos o errores ignorados sin registrar.
* Consultas SQL sin escapar, inyección de comandos y llamadas a `eval()`.
* Dependencias circulares o referencias que violan los límites entre capas del proyecto.

Raigal evalúa el código en local mediante hooks de agente, en CI con cada commit y en pull requests antes de la fusión.

## Componentes

<table data-view="cards"><thead><tr><th>Componente</th><th>Descripción</th><th>Enlace</th></tr></thead><tbody><tr><td><strong>Raigal Cloud</strong></td><td>Panel web en <code>app.raigal.dev</code> con tablero de remediación en 4 etapas e historial de escaneos.</td><td><a href="plataforma-cloud/tablero.md">Tablero Cloud</a></td></tr><tr><td><strong>GitHub App</strong></td><td>Comprobaciones en pull requests y cumplimiento de protección de ramas.</td><td><a href="automatizacion-y-ci-cd/pull-requests.md">Configurar GitHub App</a></td></tr><tr><td><strong>Motor de puntuación (0-100)</strong></td><td>Puntuación ponderada que normaliza los hallazgos según las líneas de código.</td><td><a href="conceptos-clave/puntuacion.md">Motor de Puntuación</a></td></tr><tr><td><strong>CLI y hooks de agente</strong></td><td>Verificación pre-commit, correcciones automáticas seguras (<code>raigal fix</code>) e intercepción en tiempo real para agentes.</td><td><a href="referencia-cli/comandos.md">Referencia del CLI</a></td></tr><tr><td><strong>Reparación con agentes</strong></td><td>Entornos git aislados donde agentes locales corrigen problemas y generan pull requests.</td><td><a href="automatizacion-y-ci-cd/reparacion-agentes.md">Reparación con Agentes</a></td></tr><tr><td><strong>OIDC en GitHub Actions</strong></td><td>Autenticación sin contraseñas fijas para runners de CI mediante tokens OIDC de GitHub.</td><td><a href="conceptos-clave/seguridad.md">Modelo de Seguridad</a></td></tr></tbody></table>

## Primeros pasos

{% stepper %}
{% step %}
### 1. Crear cuenta en Raigal Cloud

Cree el espacio de trabajo de su organización en [app.raigal.dev](https://app.raigal.dev) e invite a su equipo. [Guía de Configuración de la Organización](primer-contacto/configuracion-cliente.md)
{% endstep %}

{% step %}
### 2. Instalar la GitHub App de Raigal

Conecte sus repositorios con la [GitHub App de Raigal](https://github.com/apps/raigal-cloud) para habilitar comprobaciones automáticas en pull requests. [Puerta de Calidad en Pull Requests](automatizacion-y-ci-cd/pull-requests.md)
{% endstep %}

{% step %}
### 3. Configurar el flujo de CI

Añada un flujo de GitHub Actions para evaluar los pull requests y enviar las puntuaciones al tablero. [Pipelines de CI/CD y GitHub Actions](automatizacion-y-ci-cd/ci-cd.md)
{% endstep %}

{% step %}
### 4. Configurar herramientas locales y hooks

Instale el CLI (`raigal scan`, `raigal fix`) y active los hooks de intercepción para Claude Code o Cursor. [Guía de Inicio Rápido](primer-contacto/inicio-rapido.md)
{% endstep %}
{% endstepper %}
