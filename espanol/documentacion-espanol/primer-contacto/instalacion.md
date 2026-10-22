---
description: "Como instalar Raigal CLI mediante npm, npx, pnpm, yarn o bun en macOS, Linux y Windows."
icon: download
---

# Instalacion

Puede ejecutar Raigal directamente sin instalacion permanente mediante `npx`, instalarlo de forma global en su equipo o añadirlo como dependencia de su proyecto.

## Requisitos del sistema

- **Node.js**: version 20.12 o superior (se recomienda Node 22 LTS)
- **Gestor de paquetes**: `npm`, `pnpm`, `yarn` o `bun`
- **Sistemas operativos**: macOS, Linux o Windows (PowerShell y cmd.exe)

## Ejecutar directamente con `npx`

Para realizar un analisis inmediato sin instalar ningun paquete:

```bash
npx --yes @methiu/raigal@latest scan .
```

Para ejecutar el asistente interactivo de configuracion:

```bash
npx --yes @methiu/raigal@latest init
```

## Instalacion global

Instale Raigal de forma global para disponer del comando `raigal` en cualquier terminal:

{% tabs %}
{% tab title="npm" %}
```bash
npm install -g @methiu/raigal
```
{% endtab %}

{% tab title="pnpm" %}
```bash
pnpm add -g @methiu/raigal
```
{% endtab %}

{% tab title="yarn" %}
```bash
yarn global add @methiu/raigal
```
{% endtab %}

{% tab title="bun" %}
```bash
bun add -g @methiu/raigal
```
{% endtab %}
{% endtabs %}

Compruebe que el CLI se instalo correctamente:

```bash
raigal --version
```

## Dependencia del proyecto

Para fijar Raigal como dependencia de desarrollo en su repositorio:

{% tabs %}
{% tab title="npm" %}
```bash
npm install -D @methiu/raigal
```
{% endtab %}

{% tab title="pnpm" %}
```bash
pnpm add -D @methiu/raigal
```
{% endtab %}

{% tab title="yarn" %}
```bash
yarn add -D @methiu/raigal
```
{% endtab %}

{% tab title="bun" %}
```bash
bun add -d @methiu/raigal
```
{% endtab %}
{% endtabs %}

Añada los scripts correspondientes a su `package.json` para facilitar el uso a su equipo:

```json
{
  "scripts": {
    "scan": "raigal scan .",
    "gate": "raigal ci .",
    "fix": "raigal fix ."
  }
}
```

De esta forma, cualquier miembro del equipo puede ejecutar `pnpm scan` o `npm run gate` utilizando la configuracion del proyecto.
