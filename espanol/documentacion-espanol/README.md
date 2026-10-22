---
description: >-
  Control de calidad de codigo y deteccion de patrones de IA para equipos de
  ingenieria.
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

# Vision general de Raigal

Raigal detecta patrones defectuosos introducidos por asistentes de programacion asistidos por IA, como comentarios triviales, excepciones silenciadas (`swallowed-exception`), aserciones de tipo inseguras (`as any`) y llamadas dinamicas de riesgo (`eval`). Reune formateo, linting, analisis de complejidad, reglas de seguridad y deteccion de patrones en un unico comando que devuelve una puntuacion de 0 a 100.

Junto con [Raigal Cloud](https://app.raigal.dev) y la [GitHub App oficial de Raigal](https://github.com/apps/raigal-cloud), evalua pull requests, gestiona comprobaciones de estado para la proteccion de ramas, organiza las incidencias en un tablero de remediacion en 4 etapas y ejecuta reparaciones con agentes de programacion.

{% hint style="success" %}
Los archivos de codigo fuente completos nunca se suben ni se almacenan en Raigal Cloud. El analisis de AST, la ejecucion de linters y el calculo de complejidad ocurren localmente en su maquina o runner de CI. Durante la evaluacion de pull requests en CI, se envia una vista previa del parche (limitada a 20 KB) con enmascaramiento de credenciales (`maskSecrets`) para inspeccionar cambios en el tablero.
{% endhint %}

## Que detecta Raigal

Los asistentes de programacion como Claude Code, Cursor, Copilot u OpenCode aceleran el desarrollo, pero suelen introducir defectos concretos:

* Comentarios triviales que solo repiten el nombre de funciones o variables.
* Aserciones de tipo que eluden el compilador (`as any`).
* Bloques `catch` vacios o excepciones silenciadas sin registrar.
* Consultas SQL concatenadas, inyeccion de comandos o llamadas a `eval()`.
* Bloques de codigo duplicados y funciones con complejidad ciclomatica excesiva.

Raigal evalua el codigo en local mediante hooks de agente, en CI con cada commit y en pull requests antes de la fusion.

## Componentes

<table data-view="cards">
  <thead>
    <tr>
      <th>Componente</th>
      <th>Descripcion</th>
      <th>Enlace</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><strong>Raigal Cloud</strong></td>
      <td>Panel web en <code>app.raigal.dev</code> con tablero de remediacion en 4 etapas e historial de escaneos.</td>
      <td><a href="plataforma-cloud/tablero.md">Tablero Cloud</a></td>
    </tr>
    <tr>
      <td><strong>GitHub App</strong></td>
      <td>Comprobaciones en pull requests y cumplimiento de proteccion de ramas.</td>
      <td><a href="automatizacion-y-ci-cd/pull-requests.md">Configurar GitHub App</a></td>
    </tr>
    <tr>
      <td><strong>Motor de puntuacion (0-100)</strong></td>
      <td>Puntuacion ponderada normalizada por recuento de archivos fuente, topes dinamicos por regla y compresion logaritmica.</td>
      <td><a href="conceptos-clave/puntuacion.md">Motor de Puntuacion</a></td>
    </tr>
    <tr>
      <td><strong>CLI y hooks de agente</strong></td>
      <td>Verificacion pre-commit, correcciones automaticas seguras (<code>raigal fix</code>) e intercepcion en tiempo real para agentes.</td>
      <td><a href="referencia-cli/comandos.md">Referencia del CLI</a></td>
    </tr>
    <tr>
      <td><strong>Reparacion con agentes</strong></td>
      <td>Entornos git aislados donde agentes locales corrigen problemas y generan pull requests.</td>
      <td><a href="automatizacion-y-ci-cd/reparacion-agentes.md">Reparacion con Agentes</a></td>
    </tr>
    <tr>
      <td><strong>OIDC en GitHub Actions</strong></td>
      <td>Autenticacion sin contraseñas fijas para runners de CI mediante tokens OIDC de GitHub.</td>
      <td><a href="conceptos-clave/seguridad.md">Modelo de Seguridad</a></td>
    </tr>
    <tr>
      <td><strong>Raigal MCP</strong></td>
      <td>Servidor Model Context Protocol que conecta las herramientas de Raigal con Claude Code, Cursor, Windsurf, Cline y Antigravity.</td>
      <td><a href="automatizacion-y-ci-cd/servidor-mcp.md">Servidor MCP</a></td>
    </tr>
  </tbody>
</table>

## Primeros pasos

{% stepper %}
{% step %}
### 1. Crear cuenta en Raigal Cloud

Cree el espacio de trabajo de su organizacion en [app.raigal.dev](https://app.raigal.dev) con una prueba de evaluacion de 14 dias e invite a su equipo. [Guia de Configuracion de la Organizacion](primer-contacto/configuracion-cliente.md)
{% endstep %}

{% step %}
### 2. Instalar la GitHub App de Raigal

Conecte sus repositorios con la [GitHub App oficial de Raigal](https://github.com/apps/raigal-cloud) para habilitar comprobaciones automaticas en pull requests. [Puerta de Calidad en Pull Requests](automatizacion-y-ci-cd/pull-requests.md)
{% endstep %}

{% step %}
### 3. Configurar el flujo de CI

Añada un flujo de GitHub Actions para evaluar los pull requests y enviar las puntuaciones al tablero. [Pipelines de CI/CD y GitHub Actions](automatizacion-y-ci-cd/ci-cd.md)
{% endstep %}

{% step %}
### 4. Configurar herramientas locales y hooks

Instale el CLI (`raigal scan`, `raigal fix`) y active los hooks de intercepcion para Claude Code o Cursor. [Guia de Inicio Rapido](primer-contacto/inicio-rapido.md)
{% endstep %}
{% endstepper %}
