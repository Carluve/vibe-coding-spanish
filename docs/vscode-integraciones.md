# Integraciones con VS Code

Cloudflare ofrece varias integraciones con VS Code para mejorar el desarrollo: extensiones, servidores MCP y Skills para asistentes de IA dentro del editor.

---

## Extensión oficial de Wrangler

La integración más directa es a través de la extensión **Cloudflare Workers** para VS Code, que añade soporte para `wrangler.jsonc` y bindings.

> Busca "Cloudflare Workers" en el marketplace de VS Code o instala via:
```bash
code --install-extension cloudflare.cloudflare-workers-bindings
```

Funcionalidades:
- Autocompletado de bindings en `wrangler.jsonc`
- Validación de configuración
- Snippets para Workers, D1, R2 y KV

---

## GitHub Copilot + Cloudflare MCP

Conecta GitHub Copilot (en VS Code) a la API de Cloudflare mediante un servidor MCP. Copilot puede consultar tu cuenta, listar workers, ver logs y ejecutar operaciones directamente desde el chat del editor.

### Configuración

1. Instala la extensión **GitHub Copilot** en VS Code
2. Añade el servidor MCP de Cloudflare en tu configuración de VS Code:

```json
// .vscode/mcp.json  (por proyecto)
{
  "servers": {
    "cloudflare": {
      "type": "stdio",
      "command": "npx",
      "args": ["mcp-remote", "https://mcp.cloudflare.com/sse"],
      "env": {
        "CLOUDFLARE_API_TOKEN": "${env:CLOUDFLARE_API_TOKEN}"
      }
    }
  }
}
```

O en la configuración global de usuario (`settings.json`):

```json
"github.copilot.mcpServers": {
  "cloudflare": {
    "command": "npx",
    "args": ["mcp-remote", "https://mcp.cloudflare.com/sse"]
  }
}
```

3. Abre el chat de Copilot (`Ctrl+Alt+I`) y pregunta en lenguaje natural:

```
@cloudflare lista mis Workers desplegados
@cloudflare muéstrame los logs del Worker "mi-api" en los últimos 10 minutos
@cloudflare crea un nuevo D1 database llamado "mi-app-db"
```

> Documentación MCP oficial: [developers.cloudflare.com/agents/model-context-protocol](https://developers.cloudflare.com/agents/model-context-protocol/)

---

## Continue.dev + Cloudflare Workers AI

[Continue](https://continue.dev/) es una extensión open source para VS Code que funciona como copilot y soporta modelos locales y remotos — incluido Workers AI via AI Gateway.

### Instalación

```bash
code --install-extension Continue.continue
```

### Configurar Workers AI como proveedor

Edita `~/.continue/config.json`:

```json
{
  "models": [
    {
      "title": "Llama 3.1 (Workers AI)",
      "provider": "openai",
      "model": "@cf/meta/llama-3.1-8b-instruct",
      "apiBase": "https://gateway.ai.cloudflare.com/v1/{account_id}/{gateway_name}/workers-ai/v1",
      "apiKey": "tu-cloudflare-api-token"
    }
  ]
}
```

---

## Cloudflare Skills en editores con IA

Los **Cloudflare Skills** son archivos de conocimiento especializado que mejoran las respuestas de asistentes IA sobre la plataforma de Cloudflare.

### Compatible con

| Editor / Herramienta | Cómo instalar |
|---------------------|---------------|
| Claude Code | `npx skills add https://github.com/cloudflare/skills` |
| OpenCode | `opencode skills install cloudflare` |
| Codex (OpenAI) | `codex skills install cloudflare` |
| Pi | Integración nativa |

### Skills disponibles

| Skill | Cuándo activarlo |
|-------|-----------------|
| `cloudflare` | Preguntas generales de la plataforma |
| `workers-best-practices` | Al escribir o revisar Workers |
| `wrangler` | Al usar la CLI o configurar `wrangler.jsonc` |
| `durable-objects` | Al diseñar sistemas con estado |
| `agents-sdk` | Al construir agentes con el Agents SDK |
| `building-ai-agent` | Tutorial paso a paso de agentes |
| `building-mcp-server` | Al crear servidores MCP |
| `sandbox-sdk` | Al usar entornos aislados |
| `web-perf` | Optimización de rendimiento |

---

## Extensiones útiles para desarrollo con Cloudflare

| Extensión | ID | Para qué |
|-----------|-----|---------|
| REST Client | `humao.rest-client` | Probar endpoints de tu Worker sin Postman |
| Thunder Client | `rangav.vscode-thunder-client` | Cliente HTTP integrado en VS Code |
| SQLite Viewer | `qwtel.sqlite-viewer` | Ver bases de datos D1 en local |
| Prettier | `esbenp.prettier-vscode` | Formateo automático de TypeScript |
| ESLint | `dbaeumer.vscode-eslint` | Linting para Workers en TypeScript |
| GitLens | `eamodio.gitlens` | Historial de cambios y co-autoría |

---

## Workspace recomendado para proyectos Workers

Crea un archivo `.vscode/settings.json` en tu proyecto:

```json
{
  "editor.formatOnSave": true,
  "editor.defaultFormatter": "esbenp.prettier-vscode",
  "typescript.preferences.importModuleSpecifier": "relative",
  "files.associations": {
    "wrangler.jsonc": "jsonc"
  },
  "[typescript]": {
    "editor.defaultFormatter": "esbenp.prettier-vscode"
  }
}
```

Y `.vscode/extensions.json` para recomendar extensiones al equipo:

```json
{
  "recommendations": [
    "cloudflare.cloudflare-workers-bindings",
    "humao.rest-client",
    "esbenp.prettier-vscode",
    "dbaeumer.vscode-eslint"
  ]
}
```

---

## Depuración de Workers en VS Code

Wrangler expone un puerto de inspección compatible con el debugger de VS Code:

```bash
# Inicia con inspector
npx wrangler dev --inspect
```

Añade esta configuración en `.vscode/launch.json`:

```json
{
  "version": "0.2.0",
  "configurations": [
    {
      "name": "Debug Worker",
      "type": "node",
      "request": "attach",
      "port": 9229,
      "sourceMaps": true,
      "resolveSourceMapLocations": ["${workspaceFolder}/**"]
    }
  ]
}
```

Ahora puedes poner breakpoints en `src/index.ts` y depurar tu Worker como cualquier aplicación Node.js.

---

[Volver al índice principal](../README.md)
