---
description: "How to install the Raigal CLI via npm, npx, or local dependencies on macOS, Linux, and Windows."
icon: download
---

# Installation

You can run Raigal directly with `npx` or install it globally or as a project dependency.

## Requirements

- Node.js 20.12 or higher (Node 22 LTS recommended)
- `npm`, `pnpm`, `yarn`, or `bun`
- macOS, Linux, or Windows (PowerShell and cmd.exe)

## Run with `npx`

To run a scan without installing the package globally:

```bash
npx @methiu/raigal scan .
```

To pin a specific release in CI or local scripts:

```bash
npx -y @methiu/raigal@0.2.2 scan .
```

## Global installation

To keep the `raigal` command available in your terminal:

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

Verify that the CLI is installed:

```bash
raigal --version
```

## Project dependency

To pin Raigal as a development dependency for your project:

```bash
pnpm add -D @methiu/raigal
```

Add scripts to `package.json`:

```json
{
  "scripts": {
    "scan": "raigal scan .",
    "gate": "raigal ci ."
  }
}
```
