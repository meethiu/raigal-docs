---
description: "How to install the Raigal CLI via npm, npx, pnpm, yarn, or bun on macOS, Linux, and Windows."
icon: download
---

# Installation

You can run Raigal directly without installation using `npx`, install it globally across your system, or add it as a project dependency.

## System requirements

- **Node.js**: version 20.12 or higher (Node 22 LTS recommended)
- **Package manager**: `npm`, `pnpm`, `yarn`, or `bun`
- **Operating systems**: macOS, Linux, or Windows (PowerShell and cmd.exe)

## Run directly with `npx`

To run a scan immediately without installing anything permanently:

```bash
npx --yes @methiu/raigal@latest scan .
```

To run the interactive project setup wizard:

```bash
npx --yes @methiu/raigal@latest init
```

## Global installation

Install Raigal globally to make the `raigal` command available in any terminal window:

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

Verify that the CLI is installed correctly:

```bash
raigal --version
```

## Project dependency

To pin Raigal as a development dependency for your repository:

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

Add scripts to your `package.json` for team convenience:

```json
{
  "scripts": {
    "scan": "raigal scan .",
    "gate": "raigal ci .",
    "fix": "raigal fix ."
  }
}
```

Now any team member can run `pnpm scan` or `npm run gate` with consistent project configuration.
