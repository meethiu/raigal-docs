---
description: "Cómo instalar Raigal CLI mediante npm, npx o dependencias locales en macOS, Linux y Windows."
icon: download
---

# Instalación

Puede ejecutar Raigal directamente con `npx` o instalarlo de forma global o como dependencia del proyecto.

## Requisitos

- Node.js 20.12 o superior (se recomienda Node 22 LTS)
- `npm`, `pnpm`, `yarn` o `bun`
- macOS, Linux o Windows (PowerShell y cmd.exe)

## Ejecutar con `npx`

Para realizar un análisis sin instalar el paquete de forma permanente:

```bash
npx @methiu/raigal scan .
```

## Instalación global

Para disponer del comando `raigal` en cualquier terminal:

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
{% endtabs %}

Compruebe la instalación:

```bash
raigal version
# Salida: @methiu/raigal v1.0.0 (o versión activa)
```

## Dependencia del proyecto

Para incluir Raigal entre las dependencias de desarrollo de su equipo:

```bash
pnpm add -D @methiu/raigal
```

Añada los scripts correspondientes a `package.json`:

```json
{
  "scripts": {
    "scan": "raigal scan .",
    "gate": "raigal ci ."
  }
}
```
