---
description: "Raigal es la plataforma de calidad de código y control contra el 'AI slop' diseñada para equipos de ingeniería y agentes de desarrollo."
icon: shield-check
layout:
  width: default
  tableOfContents:
    visible: true
  pagination:
    visible: true
---

# Bienvenido a Raigal

Raigal es una plataforma de calidad de código y motor CLI de nivel empresarial diseñada para prevenir los patrones frágiles, redundantes e inseguros que introducen los asistentes de programación basados en Inteligencia Artificial. Unifica formateo, linting, control de complejidad, auditorías de seguridad y detección especializada de "AI slop" tras un único comando, devolviendo una puntuación accionable de 0 a 100.

Junto con la **[Plataforma Raigal Cloud](https://app.raigal.dev)** y la **[GitHub App Oficial de Raigal](https://github.com/apps/raigal-cloud)**, las organizaciones de ingeniería obtienen control de calidad automatizado en Pull Requests, cumplimiento de reglas de protección de ramas, tableros Kanban de remediación en tiempo real y reparación autónoma con agentes de IA.

{% hint style="success" %}
**Cero Exfiltración de Código:** Raigal ejecuta todo el análisis de AST, las auditorías de seguridad y la detección de slop 100% dentro del entorno del desarrollador o del ejecutor de CI. El código fuente de su empresa nunca sale de su infraestructura ni se envía a los servidores de Raigal.
{% endhint %}

## ¿Por qué los equipos modernos utilizan Raigal?

Los asistentes de programación basados en IA generativa (Claude Code, Cursor, Copilot, Codex, OpenCode) aceleran el desarrollo, pero frecuentemente introducen:
- **Comentarios triviales y relleno:** Explicaciones redundantes de código obvio o duplicación de firmas.
- **Alucinaciones defensivas:** Polyfills innecesarios, verificaciones nulas redundantes y aserciones de tipos obsoletas.
- **Antipatrones de fallo silencioso:** Bloques `catch` vacíos o errores ignorados con recuperaciones silenciosas.
- **Vulnerabilidades de seguridad:** Uso involuntario de `eval()`, consultas SQL sin escapar o inyecciones de comandos en la shell.
- **Deriva arquitectónica:** Violación de las capas de dependencia y acumulación de deuda técnica oculta.

Raigal proporciona una puerta de calidad automatizada en cada fase del ciclo de desarrollo: en local mediante hooks de agente, en los pipelines de CI con cada commit, y en el control de revisión de Pull Requests.

## El Ecosistema de la Plataforma Raigal

<table data-view="cards">
  <thead>
    <tr>
      <th>Componente</th>
      <th>Descripción</th>
      <th>Enlace</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><strong>Plataforma Raigal Cloud</strong></td>
      <td>Panel SaaS centralizado en <code>app.raigal.dev</code> con tablero Kanban de remediación en 4 etapas y telemetría de equipo.</td>
      <td><a href="plataforma-cloud/tablero.md">Tablero Cloud</a></td>
    </tr>
    <tr>
      <td><strong>GitHub App Oficial</strong></td>
      <td>Puerta de calidad para PRs, comprobaciones de estado nativas y cumplimiento de branch protection.</td>
      <td><a href="automatizacion/pull-requests.md">Configurar GitHub App</a></td>
    </tr>
    <tr>
      <td><strong>Puntuación Determinista (0–100)</strong></td>
      <td>Motor de puntuación ponderado por densidad que mide la salud real del código sin ruido.</td>
      <td><a href="conceptos-clave/puntuacion.md">Motor de Puntuación</a></td>
    </tr>
    <tr>
      <td><strong>CLI Local y Hooks de Agente</strong></td>
      <td>Verificación pre-commit, autocorrección (<code>raigal fix</code>) e intercepción en tiempo real para agentes.</td>
      <td><a href="referencia-cli/comandos.md">Referencia del CLI</a></td>
    </tr>
    <tr>
      <td><strong>Reparaciones Autónomas con Agentes</strong></td>
      <td>Worktrees aislados de git que invocan agentes LLM para corregir hallazgos y crear PRs listos para fusionar.</td>
      <td><a href="automatizacion/reparacion-agentes.md">Reparación con Agentes</a></td>
    </tr>
    <tr>
      <td><strong>OIDC Empresarial sin Secretos</strong></td>
      <td>Federación OIDC con GitHub Actions que elimina la necesidad de tokens estáticos de larga duración en CI.</td>
      <td><a href="conceptos-clave/seguridad.md">Modelo de Seguridad</a></td>
    </tr>
  </tbody>
</table>

## Primeros Pasos en 4 Pasos

{% stepper %}
{% step %}
### 1. Inicie sesión en Raigal Cloud
Cree el espacio de trabajo de su organización en **[app.raigal.dev](https://app.raigal.dev)** e invite a los miembros de su equipo.
[Guía de Configuración de la Organización](primer-contacto/configuracion-cliente.md)
{% endstep %}

{% step %}
### 2. Instale la GitHub App de Raigal
Conecte los repositorios de su organización con la **[GitHub App Oficial de Raigal](https://github.com/apps/raigal-cloud)** para habilitar comprobaciones automáticas de estado en cada Pull Request.
[Puerta de Calidad en Pull Requests](automatizacion/pull-requests.md)
{% endstep %}

{% step %}
### 3. Configure la Puerta de Calidad en CI
Añada un sencillo flujo de GitHub Actions para evaluar los Pull Requests y supervisar las puntuaciones de calidad en su tablero Kanban.
[Pipelines de CI/CD y GitHub Actions](automatizacion/ci-cd.md)
{% endstep %}

{% step %}
### 4. Habilite Escaneos y Hooks Locales
Equipe a los desarrolladores con el CLI (`raigal scan`, `raigal fix`) e instale hooks de intercepción para Claude Code y Cursor.
[Guía de Inicio Rápido](primer-contacto/inicio-rapido.md)
{% endstep %}
{% endstepper %}
