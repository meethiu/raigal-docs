---
description: "Conecte Raigal MCP a Claude Code, Cursor, Claude Desktop, Windsurf, Cline y Antigravity."
icon: plugs
---

# Servidor Raigal MCP

El servidor Model Context Protocol (MCP) de Raigal conecta los motores de calidad y detección de patrones directamente con sus agentes de programación e IDEs mediante `npx -y -p @methiu/raigal raigal-mcp` (o `npx -y @methiu/raigal mcp`).

## Requisitos y autenticación

Raigal MCP se autentica de dos formas:

- Sesión local: si ya ejecutó `raigal login` en su equipo, el servidor MCP detecta sus credenciales locales automáticamente sin configuración adicional.
- Variable de entorno: en entornos headless o CI, proporcione su token configurando `RAIGAL_TOKEN=rgl_live_...`.

## Configuración de clientes

{% tabs %}
{% tab title="Claude Code" %}
Ejecute el comando de configuración en su terminal:

```bash
claude mcp add raigal -- npx -y -p @methiu/raigal raigal-mcp
```

Con token de API explícito:

```bash
claude mcp add raigal -e RAIGAL_TOKEN=rgl_live_... -- npx -y -p @methiu/raigal raigal-mcp
```
{% endtab %}

{% tab title="Cursor" %}
1. Abra **Cursor Settings** (`Cmd + ,` o `Ctrl + ,`).
2. Vaya a **Features** > **MCP**.
3. Seleccione **Add New MCP Server**.
4. Configure el servidor:
   - Nombre: `raigal`
   - Tipo: `command`
   - Comando: `npx -y -p @methiu/raigal raigal-mcp`

Para configurar vía `.cursor/mcp.json`:

```json
{
  "mcpServers": {
    "raigal": {
      "command": "npx",
      "args": ["-y", "-p", "@methiu/raigal", "raigal-mcp"],
      "env": {
        "RAIGAL_TOKEN": "rgl_live_SU_TOKEN_AQUI"
      }
    }
  }
}
```
{% endtab %}

{% tab title="Claude Desktop" %}
Agregue el servidor a su archivo de configuración de Claude Desktop:
- macOS: `~/Library/Application Support/Claude/claude_desktop_config.json`
- Windows: `%APPDATA%\Claude\claude_desktop_config.json`

```json
{
  "mcpServers": {
    "raigal": {
      "command": "npx",
      "args": ["-y", "-p", "@methiu/raigal", "raigal-mcp"],
      "env": {
        "RAIGAL_TOKEN": "rgl_live_SU_TOKEN_AQUI"
      }
    }
  }
}
```
{% endtab %}

{% tab title="Windsurf" %}
Agregue el servidor a `~/.codeium/windsurf/mcp_config.json`:

```json
{
  "mcpServers": {
    "raigal": {
      "command": "npx",
      "args": ["-y", "-p", "@methiu/raigal", "raigal-mcp"]
    }
  }
}
```
{% endtab %}

{% tab title="Cline / Roo Code" %}
1. Abra la barra lateral de **Cline / Roo Code** en VS Code.
2. Seleccione el icono de **MCP Servers**.
3. Seleccione **Edit Global MCP Settings** y agregue:

```json
{
  "mcpServers": {
    "raigal": {
      "command": "npx",
      "args": ["-y", "-p", "@methiu/raigal", "raigal-mcp"]
    }
  }
}
```
{% endtab %}

{% tab title="Antigravity" %}
Agregue el servidor a `~/.gemini/config/mcp_config.json` o a `.agents/mcp_config.json` en su repositorio:

```json
{
  "mcpServers": {
    "raigal": {
      "command": "npx",
      "args": ["-y", "-p", "@methiu/raigal", "raigal-mcp"]
    }
  }
}
```
{% endtab %}
{% endtabs %}

## Herramientas disponibles

Una vez conectado, los agentes pueden invocar cuatro herramientas:

| Herramienta | Descripción |
|---|---|
| `raigal_scan` | Evalúa el repositorio en busca de patrones de IA, riesgos de seguridad, linter y formato, retornando un puntaje de 0 a 100. |
| `raigal_fix` | Aplica correcciones automáticas y seguras para los hallazgos detectados. |
| `raigal_why` | Explica por qué se activó una regla concreta y sugiere remediaciones. |
| `raigal_baseline` | Crea o actualiza una línea base para que el agente se centre en las nuevas regresiones. |

## Instrucciones para el prompt del agente

Agregue este bloque a `AGENTS.md`, `CLAUDE.md` o al prompt de su agente:

```markdown
Antes de enviar cambios de código, ejecute la herramienta MCP `raigal_scan` para verificar antipatrones de IA, excepciones silenciadas y regresiones de tipos. Si se detectan problemas, use `raigal_fix` o resuélvalos antes de finalizar.
```
