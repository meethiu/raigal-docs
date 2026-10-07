---
description: "Conecte Raigal MCP con Claude Code, Cursor, Claude Desktop, Windsurf, Cline y Antigravity."
icon: plugs
---

# Servidor MCP de Raigal

El servidor Model Context Protocol (MCP) de Raigal conecta los motores de calidad y detección de patrones directamente con agentes de programación e IDEs mediante `npx @methiu/raigal-mcp`.

## Requisitos y autenticación

Raigal MCP se autentica de dos formas:

- Sesión local: si ya ha ejecutado `raigal login` en su equipo, el servidor detecta las credenciales locales de forma automática sin necesidad de configurar tokens adicionales.
- Variable de entorno: en entornos sin interfaz o pipelines de CI, proporcione su clave de API mediante la variable `RAIGAL_TOKEN=rgl_live_...`.

## Configuración por cliente

{% tabs %}
{% tab title="Claude Code" %}
Ejecute este comando en su terminal:

```bash
claude mcp add raigal -- npx -y @methiu/raigal-mcp
```

Con una clave de API explícita:

```bash
claude mcp add raigal -e RAIGAL_TOKEN=rgl_live_... -- npx -y @methiu/raigal-mcp
```
{% endtab %}

{% tab title="Cursor" %}
1. Abra los ajustes de Cursor (**Cursor Settings**, `Cmd + ,` o `Ctrl + ,`).
2. Vaya a **Features** > **MCP**.
3. Pulse **Add New MCP Server**.
4. Rellene los campos:
   - Name: `raigal`
   - Type: `command`
   - Command: `npx -y @methiu/raigal-mcp`

Si prefiere configurar el archivo `.cursor/mcp.json` directamente:

```json
{
  "mcpServers": {
    "raigal": {
      "command": "npx",
      "args": ["-y", "@methiu/raigal-mcp"],
      "env": {
        "RAIGAL_TOKEN": "rgl_live_YOUR_TOKEN_HERE"
      }
    }
  }
}
```
{% endtab %}

{% tab title="Claude Desktop" %}
Añada el servidor a su archivo de configuración de Claude Desktop:
- macOS: `~/Library/Application Support/Claude/claude_desktop_config.json`
- Windows: `%APPDATA%\Claude\claude_desktop_config.json`

```json
{
  "mcpServers": {
    "raigal": {
      "command": "npx",
      "args": ["-y", "@methiu/raigal-mcp"],
      "env": {
        "RAIGAL_TOKEN": "rgl_live_YOUR_TOKEN_HERE"
      }
    }
  }
}
```
{% endtab %}

{% tab title="Windsurf" %}
Añada la configuración a `~/.codeium/windsurf/mcp_config.json`:

```json
{
  "mcpServers": {
    "raigal": {
      "command": "npx",
      "args": ["-y", "@methiu/raigal-mcp"]
    }
  }
}
```
{% endtab %}

{% tab title="Cline / Roo Code" %}
1. Abra la barra lateral de **Cline / Roo Code** en VS Code.
2. Pulse el icono de **MCP Servers**.
3. Seleccione **Edit Global MCP Settings** y añada:

```json
{
  "mcpServers": {
    "raigal": {
      "command": "npx",
      "args": ["-y", "@methiu/raigal-mcp"]
    }
  }
}
```
{% endtab %}

{% tab title="Antigravity" %}
Añada la configuración a `~/.gemini/config/mcp_config.json` o al archivo `.agents/mcp_config.json` de su repositorio:

```json
{
  "mcpServers": {
    "raigal": {
      "command": "npx",
      "args": ["-y", "@methiu/raigal-mcp"]
    }
  }
}
```
{% endtab %}
{% endtabs %}

## Herramientas disponibles

Una vez conectado, los agentes disponen de cuatro herramientas:

| Herramienta | Descripción |
|---|---|
| `raigal_scan` | Analiza el repositorio en busca de patrones defectuosos (comentarios innecesarios, excepciones silenciadas, casts `as any`), problemas de seguridad, errores de lint y formato, devolviendo una puntuación de 0 a 100. |
| `raigal_fix` | Ejecuta correcciones automáticas y seguras para los problemas detectados. |
| `raigal_why` | Explica el motivo por el que se activó una regla y sugiere recomendaciones de solución. |
| `raigal_baseline` | Genera o actualiza una línea base para que el agente se concentre únicamente en defectos nuevos. |

## Instrucciones para el prompt del agente

Para que los agentes de programación verifiquen su trabajo antes de concluir, añada esta instrucción a `AGENTS.md`, `CLAUDE.md` o al prompt del sistema:

```markdown
Before submitting code changes, run the `raigal_scan` MCP tool to check for AI anti-patterns, swallowed errors, and type regressions. If issues are found, use `raigal_fix` or resolve them before finishing.
```
