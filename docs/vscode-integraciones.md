# 💻 Integraciones con VS Code

VS Code es el editor más usado para desarrollar con Cloudflare Workers. Esta guía cubre todas las integraciones disponibles: extensiones oficiales, servidores MCP para IA en el editor, Skills, debugging de Workers y configuración recomendada de workspace.

---

## Extensión oficial de Cloudflare

La extensión **Cloudflare Workers** añade soporte directo para el ecosistema de Workers en VS Code.

```bash
# Instalar desde la terminal
code --install-extension cloudflare.cloudflare-workers-bindings
```

O busca `Cloudflare Workers` en el panel de extensiones de VS Code.

**Qué incluye:**
- Autocompletado y validación de `wrangler.jsonc` (bindings, flags, triggers)
- Snippets para Workers, Durable Objects, Queues y Workflows
- Documentación inline de bindings al pasar el ratón

---

## MCP: Cloudflare en tu asistente de VS Code

**MCP (Model Context Protocol)** permite que el asistente de IA de tu editor hable directamente con la API de Cloudflare. Puede listar tus Workers, ver logs, crear recursos, ejecutar consultas en D1, y mucho más — todo desde el chat del editor.

Cloudflare mantiene un servidor MCP oficial en `mcp.cloudflare.com` que no requiere instalación local.

### Configurar MCP con GitHub Copilot

**Requisitos:** GitHub Copilot con el modo agente activado (VS Code 1.99+)

Crea el fichero `.vscode/mcp.json` en tu proyecto (solo para ese proyecto) o configúralo globalmente:

```json
{
  "servers": {
    "cloudflare": {
      "type": "stdio",
      "command": "npx",
      "args": [
        "mcp-remote",
        "https://mcp.cloudflare.com/sse"
      ]
    }
  }
}
```

> La primera vez que uses el MCP de Cloudflare, te pedirá autenticarte con tu cuenta de Cloudflare vía OAuth — no necesitas generar tokens manualmente.

Una vez configurado, abre el chat de Copilot (`Ctrl+Alt+I` / `Cmd+Alt+I`) y cambia al modo agente. Prueba:

```
@workspace Usando Cloudflare, lista mis Workers desplegados en producción

@workspace Muéstrame los logs del último despliegue de "mi-api"

@workspace Crea una nueva base de datos D1 llamada "mi-app-db" y
genera el esquema SQL para una tabla de usuarios con campos básicos

@workspace ¿Cuánto tráfico ha recibido mi Worker "mi-api" en las
últimas 24 horas?
```

### Configurar MCP con Continue.dev

[Continue](https://continue.dev/) es una alternativa open source a GitHub Copilot, especialmente interesante porque soporta Workers AI como backend.

```bash
code --install-extension Continue.continue
```

Edita `~/.continue/config.json`:

```json
{
  "models": [
    {
      "title": "Claude (via AI Gateway)",
      "provider": "anthropic",
      "model": "claude-sonnet-4-5",
      "apiKey": "TU_ANTHROPIC_API_KEY",
      "apiBase": "https://gateway.ai.cloudflare.com/v1/{account_id}/{gateway_name}/anthropic"
    },
    {
      "title": "Llama 3.3 70B (Workers AI)",
      "provider": "openai",
      "model": "@cf/meta/llama-3.3-70b-instruct-fp8-fast",
      "apiKey": "TU_CLOUDFLARE_API_TOKEN",
      "apiBase": "https://gateway.ai.cloudflare.com/v1/{account_id}/{gateway_name}/workers-ai/v1"
    }
  ],
  "mcpServers": [
    {
      "name": "cloudflare",
      "command": "npx",
      "args": ["mcp-remote", "https://mcp.cloudflare.com/sse"]
    }
  ],
  "tabAutocompleteModel": {
    "title": "Llama 3.1 8B (rápido)",
    "provider": "openai",
    "model": "@cf/meta/llama-3.1-8b-instruct",
    "apiKey": "TU_CLOUDFLARE_API_TOKEN",
    "apiBase": "https://gateway.ai.cloudflare.com/v1/{account_id}/{gateway_name}/workers-ai/v1"
  }
}
```

Con esta configuración, Continue usa Workers AI para el autocompletado inline (rápido y económico) y Claude para las preguntas más complejas en el chat.

---

## Cloudflare Skills en editores

Los **Skills** son ficheros de conocimiento especializado sobre Cloudflare que el asistente de IA lee antes de responder. Con ellos, el agente sabe exactamente cómo funciona Wrangler, los bindings, el Agents SDK, los Durable Objects...

### Instalación

```bash
# Claude Code
npx skills add https://github.com/cloudflare/skills

# OpenCode
opencode skills install cloudflare
```

### Qué incluyen los Skills

| Skill | Contenido |
|-------|-----------|
| `cloudflare` | Visión general de la plataforma y conceptos clave |
| `workers-best-practices` | Patrones de código, manejo de errores, rendimiento |
| `wrangler` | CLI completa: comandos, flags, `wrangler.jsonc` |
| `durable-objects` | Diseño de sistemas con estado, SQLite embebido |
| `agents-sdk` | API completa del Agents SDK, herramientas, scheduling |
| `building-ai-agent` | Tutorial paso a paso, casos de uso, arquitectura |
| `building-mcp-server` | Crear servidores MCP en Workers, autenticación OAuth |
| `sandbox-sdk` | Entornos de ejecución aislados para código no confiable |
| `web-perf` | Cache Rules, Early Hints, optimización de assets |
| `find-skills` | Muestra qué skills hay disponibles y cuándo usarlos |

> Repositorio oficial: [github.com/cloudflare/skills](https://github.com/cloudflare/skills)

---

## Extensiones recomendadas para desarrollar con Workers

### Esenciales

| Extensión | ID | Por qué la necesitas |
|-----------|----|--------------------|
| Cloudflare Workers | `cloudflare.cloudflare-workers-bindings` | Soporte nativo para Wrangler y bindings |
| REST Client | `humao.rest-client` | Prueba tus endpoints directamente desde `.http` files |
| Prettier | `esbenp.prettier-vscode` | Formatea TypeScript automáticamente al guardar |
| ESLint | `dbaeumer.vscode-eslint` | Detecta problemas en tu código antes de desplegar |

### Muy útiles

| Extensión | ID | Para qué |
|-----------|----|----|
| Thunder Client | `rangav.vscode-thunder-client` | Cliente HTTP completo tipo Postman integrado en VS Code |
| SQLite Viewer | `qwtel.sqlite-viewer` | Ver bases de datos D1 en local (fichero `.sqlite`) |
| GitLens | `eamodio.gitlens` | Historial completo de cambios y coautoría |
| Error Lens | `usernamehw.errorlens` | Muestra errores de TypeScript directamente en la línea |
| Better Comments | `aaron-bond.better-comments` | Comentarios con colores por tipo (TODO, FIXME, etc.) |

### Para IA en el editor

| Extensión | ID | Descripción |
|-----------|----|----|
| GitHub Copilot | `GitHub.copilot` | Autocompletado y chat con Claude/GPT (requiere suscripción) |
| Continue | `Continue.continue` | Open source, soporta Workers AI, compatible con MCP |

---

## Workspace recomendado para proyectos Workers

Crea estos ficheros en tu proyecto para que cualquier persona del equipo tenga la misma configuración desde el primer día:

### `.vscode/settings.json`

```json
{
  // Formateo automático al guardar
  "editor.formatOnSave": true,
  "editor.defaultFormatter": "esbenp.prettier-vscode",
  "[typescript]": {
    "editor.defaultFormatter": "esbenp.prettier-vscode"
  },

  // TypeScript
  "typescript.preferences.importModuleSpecifier": "relative",
  "typescript.suggest.autoImports": true,

  // Ficheros de configuración de Cloudflare
  "files.associations": {
    "wrangler.jsonc": "jsonc",
    "*.toml": "toml"
  },

  // Excluir carpetas de búsquedas
  "search.exclude": {
    "**/node_modules": true,
    "**/.wrangler": true,
    "**/dist": true
  },

  // ESLint
  "editor.codeActionsOnSave": {
    "source.fixAll.eslint": "explicit"
  }
}
```

### `.vscode/extensions.json`

```json
{
  "recommendations": [
    "cloudflare.cloudflare-workers-bindings",
    "esbenp.prettier-vscode",
    "dbaeumer.vscode-eslint",
    "humao.rest-client",
    "qwtel.sqlite-viewer",
    "usernamehw.errorlens"
  ]
}
```

Cuando alguien clone el repo y abra VS Code, verá una notificación para instalar las extensiones recomendadas.

### `.vscode/mcp.json` (para todo el equipo)

```json
{
  "servers": {
    "cloudflare": {
      "type": "stdio",
      "command": "npx",
      "args": ["mcp-remote", "https://mcp.cloudflare.com/sse"]
    }
  }
}
```

---

## Probar endpoints con REST Client

Crea un fichero `requests.http` en la raíz del proyecto para documentar y probar tu API directamente desde VS Code (extensión REST Client):

```http
### Variables
@base = http://localhost:8787
@token = mi-token-de-prueba

### Listar productos
GET {{base}}/api/productos?pagina=1
Authorization: Bearer {{token}}

### Crear producto
POST {{base}}/api/productos
Authorization: Bearer {{token}}
Content-Type: application/json

{
  "nombre": "Camiseta Cloudflare",
  "precio": 29.99,
  "stock": 100
}

### Obtener producto por ID
GET {{base}}/api/productos/1
Authorization: Bearer {{token}}

### Producción
@prod = https://mi-api.mi-usuario.workers.dev

GET {{prod}}/api/productos
Authorization: Bearer {{token}}
```

Haz clic en `Send Request` encima de cada bloque para ejecutarlo. Los resultados aparecen en un panel lateral.

---

## Depuración de Workers en VS Code

Puedes depurar tu Worker con breakpoints reales gracias al inspector de Wrangler.

### Paso 1: Arrancar con el inspector

```bash
npx wrangler dev --inspect
# → Inspector disponible en ws://127.0.0.1:9229
```

### Paso 2: Configurar el debugger

Crea `.vscode/launch.json`:

```json
{
  "version": "0.2.0",
  "configurations": [
    {
      "name": "Debug Cloudflare Worker",
      "type": "node",
      "request": "attach",
      "port": 9229,
      "address": "127.0.0.1",
      "localRoot": "${workspaceFolder}/src",
      "remoteRoot": "/",
      "sourceMaps": true,
      "resolveSourceMapLocations": [
        "${workspaceFolder}/src/**",
        "!**/node_modules/**"
      ],
      "skipFiles": ["<node_internals>/**", "**/node_modules/**"]
    }
  ]
}
```

### Paso 3: Depurar

1. Arranca `npx wrangler dev --inspect`
2. Pon un breakpoint en `src/index.ts` (clic en el margen izquierdo)
3. Pulsa `F5` o ve a **Run → Start Debugging**
4. Haz una petición a `http://localhost:8787` (desde el navegador o REST Client)
5. VS Code parará en el breakpoint — puedes inspeccionar variables, el `env`, la `request`...

---

## Tareas de VS Code para comandos frecuentes

Crea `.vscode/tasks.json` para ejecutar comandos de Wrangler con `Ctrl+Shift+B`:

```json
{
  "version": "2.0.0",
  "tasks": [
    {
      "label": "Workers: Desarrollar en local",
      "type": "shell",
      "command": "npx wrangler dev",
      "group": { "kind": "build", "isDefault": true },
      "presentation": { "panel": "dedicated", "clear": true },
      "problemMatcher": []
    },
    {
      "label": "Workers: Desplegar a producción",
      "type": "shell",
      "command": "npx wrangler deploy",
      "presentation": { "panel": "shared" },
      "problemMatcher": []
    },
    {
      "label": "Workers: Ver logs en tiempo real",
      "type": "shell",
      "command": "npx wrangler tail",
      "presentation": { "panel": "dedicated" },
      "problemMatcher": [],
      "isBackground": true
    },
    {
      "label": "D1: Ejecutar migraciones (local)",
      "type": "shell",
      "command": "npx wrangler d1 execute mi-db --local --file=migrations/$(ls migrations/ | tail -1)",
      "presentation": { "panel": "shared" },
      "problemMatcher": []
    }
  ]
}
```

---

[← Volver al índice](../README.md)
