# Vibe Coding con OpenCode

**Vibe Coding** es una forma de desarrollar software asistido por IA donde describes lo que quieres construir en lenguaje natural y el agente escribe, ejecuta y corrige el código por ti. [OpenCode](https://opencode.ai/) es un cliente de terminal open source compatible con múltiples LLMs.

---

## ¿Qué es OpenCode?

OpenCode es una CLI de coding con IA que corre directamente en tu terminal. A diferencia de Claude Code (que usa solo Claude), OpenCode soporta múltiples proveedores:

- Anthropic (Claude)
- OpenAI (GPT-4o)
- Google (Gemini)
- Cualquier modelo compatible con OpenAI API — incluido **Workers AI via AI Gateway**

> Repositorio: [sst/opencode](https://github.com/sst/opencode)

---

## Instalación

```bash
# Con npm
npm install -g opencode-ai

# Con curl (Linux/macOS)
curl -fsSL https://opencode.ai/install | bash
```

---

## Configuración con Cloudflare

### Usar Workers AI como proveedor de inferencia

Puedes apuntar OpenCode a tu **AI Gateway** para usar Workers AI (Llama, Mistral, etc.) como backend:

```bash
# ~/.config/opencode/config.json
{
  "provider": "openai",
  "model": "openai/@cf/meta/llama-3.1-8b-instruct",
  "baseURL": "https://gateway.ai.cloudflare.com/v1/{account_id}/{gateway_name}/workers-ai/v1",
  "apiKey": "tu-cloudflare-api-token"
}
```

> Guía completa: [AI Gateway como proxy para OpenCode](docs/ai-gateway.md)

### Instalar Cloudflare Skills en OpenCode

Los Skills amplían las capacidades de OpenCode con conocimiento especializado de Cloudflare:

```bash
# Instalar todos los skills de Cloudflare
opencode skills install cloudflare

# O desde el repositorio directamente
npx skills add https://github.com/cloudflare/skills
```

Skills disponibles:
| Skill | Descripción |
|-------|-------------|
| `cloudflare` | Conocimiento general de la plataforma |
| `workers-best-practices` | Patrones y buenas prácticas en Workers |
| `wrangler` | Uso avanzado de la CLI Wrangler |
| `durable-objects` | Diseño de sistemas con estado persistente |
| `agents-sdk` | Construcción de agentes con el Agents SDK |
| `building-ai-agent` | Guía paso a paso para agentes IA |
| `building-mcp-server` | Crear servidores MCP en Workers |
| `sandbox-sdk` | Uso del Sandbox SDK para entornos aislados |
| `web-perf` | Optimización de rendimiento web |
| `find-skills` | Descubre qué skills están disponibles |

---

## Flujo de trabajo — Vibe Coding con Cloudflare

### 1. Iniciar un proyecto

```bash
# Crea el proyecto con create-cloudflare
npm create cloudflare@latest mi-app

# Abre OpenCode en el directorio
cd mi-app
opencode
```

### 2. Desarrollar por conversación

Describe lo que quieres en lenguaje natural:

```
> Añade un endpoint GET /productos que devuelva los productos de la tabla "productos" en D1, paginados de 10 en 10

> Ahora añade autenticación con un header Authorization: Bearer <token>, el token lo guardo en una variable de entorno API_SECRET

> Crea la migración SQL para la tabla productos con campos: id, nombre, precio, stock, creado_en
```

OpenCode escribe el código, crea los archivos y puede ejecutar comandos como `wrangler d1 execute` directamente.

### 3. Probar en local

```bash
npx wrangler dev
# OpenCode puede ejecutar esto por ti y leer los logs
```

### 4. Desplegar

```bash
# OpenCode puede hacer el deploy cuando le confirmes
npx wrangler deploy
```

---

## Ejemplo real: API con D1 en una sesión

Partiendo de cero, en una sola sesión de vibe coding puedes construir:

```
> Crea una API REST completa con Hono y D1 para gestionar una lista de tareas.
  Necesito: GET /tasks, POST /tasks, PUT /tasks/:id, DELETE /tasks/:id.
  Añade validación de datos con Zod y tests con Vitest.
```

OpenCode generará:
- `src/index.ts` — Worker con Hono y todos los endpoints
- `schema.sql` — Migración de la tabla tasks
- `src/index.spec.ts` — Tests con Vitest
- `wrangler.jsonc` — Configuración con el binding D1

---

## Comparativa: OpenCode vs Claude Code

| Característica | OpenCode | Claude Code |
|----------------|----------|-------------|
| Interfaz | Terminal (TUI) | Terminal |
| Modelos | Multi-proveedor | Solo Claude |
| Workers AI via Gateway | Sí | No directamente |
| Skills de Cloudflare | Sí | Sí |
| MCP Support | Sí | Sí |
| Open Source | Sí | No |
| Precio | Paga tu propio API | Suscripción Claude |

---

## Recursos

- Repositorio: [github.com/sst/opencode](https://github.com/sst/opencode)
- Skills de Cloudflare: [github.com/cloudflare/skills](https://github.com/cloudflare/skills)
- AI Gateway para inferencia: [ver guía](ai-gateway.md)

---

[Volver al índice principal](../README.md)
