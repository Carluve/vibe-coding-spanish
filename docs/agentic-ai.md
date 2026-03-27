# Agentic AI

Infraestructura de IA en el edge: modelos serverless, agentes con estado persistente, AI Gateway y herramientas MCP (Model Context Protocol).

---

## Repositorios oficiales

| Repositorio | Estrellas | Descripción | Lenguaje |
|-------------|-----------|-------------|----------|
| [cloudflare/agents](https://github.com/cloudflare/agents) | ~1.2k | **Agents SDK** — framework para agentes IA con estado en Durable Objects | TypeScript |
| [cloudflare/agents-starter](https://github.com/cloudflare/agents-starter) | — | Kit de inicio: chat, tools, scheduling, vision | TypeScript |
| [cloudflare/workers-mcp](https://github.com/cloudflare/workers-mcp) | ~270+ | Servidor MCP en Workers — conecta LLMs a la API de Cloudflare | TypeScript |
| [cloudflare/skills](https://github.com/cloudflare/skills) | — | Agent Skills para Claude Code, OpenCode, Codex y Pi | Markdown |

---

## Documentación por producto

| Producto | Documentación | Descripción |
|----------|--------------|-------------|
| Workers AI | [developers.cloudflare.com/workers-ai](https://developers.cloudflare.com/workers-ai/) | Modelos ML serverless en GPUs — LLMs, embeddings, imágenes, ASR |
| AI Gateway | [developers.cloudflare.com/ai-gateway](https://developers.cloudflare.com/ai-gateway/) | Proxy para OpenAI, Anthropic, Google — caché, rate limiting y analytics |
| Vectorize | [developers.cloudflare.com/vectorize](https://developers.cloudflare.com/vectorize/) | Base de datos vectorial para búsqueda semántica y RAG |
| Agents SDK | [developers.cloudflare.com/agents](https://developers.cloudflare.com/agents/) | Agentes con estado, scheduling, WebSockets y herramientas |
| MCP Servers | [developers.cloudflare.com/agents/model-context-protocol](https://developers.cloudflare.com/agents/model-context-protocol/) | Servidores MCP remotos con OAuth |
| AI Security | [developers.cloudflare.com/ai-gateway](https://developers.cloudflare.com/ai-gateway/) | Descubrimiento y protección de aplicaciones IA |

---

## Inicio rápido: Agente IA

```bash
# Crea un agente con el starter oficial
npx create-cloudflare@latest --template cloudflare/agents-starter

cd agents-starter
npm install

# Desarrolla en local (Workers AI sin API key)
npm run dev
# Abre http://localhost:5173

# Despliega a producción
npm run deploy
```

---

## Workers AI — Ejemplos

### Inferencia con un LLM

```typescript
export default {
  async fetch(request: Request, env: Env) {
    const { messages } = await request.json();

    const respuesta = await env.AI.run("@cf/meta/llama-3.1-8b-instruct", {
      messages,
    });

    return Response.json(respuesta);
  },
};
```

### Generación de embeddings (para RAG)

```typescript
const embeddings = await env.AI.run("@cf/baai/bge-small-en-v1.5", {
  text: ["primer documento", "segundo documento"],
});

// Almacena en Vectorize
await env.VECTORIZE.upsert([
  { id: "doc-1", values: embeddings.data[0] },
  { id: "doc-2", values: embeddings.data[1] },
]);
```

### Búsqueda semántica

```typescript
const consulta = await env.AI.run("@cf/baai/bge-small-en-v1.5", {
  text: ["¿Cómo configuro un tunnel?"],
});

const resultados = await env.VECTORIZE.query(consulta.data[0], {
  topK: 5,
  returnMetadata: true,
});
```

---

## Agents SDK — Agente con estado persistente

El Agents SDK permite crear agentes que mantienen estado entre conversaciones, se despiertan con schedules y exponen herramientas vía MCP.

```typescript
import { Agent } from "agents";

export class MiAgente extends Agent<Env> {
  // Estado persistente en SQLite (Durable Object)
  async onRequest(request: Request) {
    const historial = await this.sql`
      SELECT * FROM mensajes ORDER BY creado_en DESC LIMIT 10
    `;
    return Response.json(historial);
  }

  // Scheduling automático — se ejecuta sin usuario presente
  async onSchedule(schedule: { cron: string }) {
    await this.sql`
      INSERT INTO tareas_log (ejecutado_en) VALUES (datetime('now'))
    `;
  }
}
```

### Herramientas para el agente

```typescript
import { Agent, tool } from "agents";
import { z } from "zod";

export class AgenteConTools extends Agent<Env> {
  buscarProducto = tool({
    description: "Busca un producto en la base de datos",
    parameters: z.object({
      nombre: z.string(),
    }),
    execute: async ({ nombre }) => {
      const resultado = await this.sql`
        SELECT * FROM productos WHERE nombre LIKE ${"%" + nombre + "%"}
      `;
      return resultado;
    },
  });
}
```

---

## Servidor MCP en Workers

Expón herramientas a cualquier LLM compatible con MCP (Claude, GPT-4, etc.).

```bash
npx create-cloudflare@latest mi-mcp-server
npm install workers-mcp
npx workers-mcp setup
npm run deploy
```

> Documentación: [MCP Servers en Workers](https://developers.cloudflare.com/agents/model-context-protocol/)

---

## Cloudflare Skills para editores

Instala Skills de Cloudflare en tu editor o agente:

```bash
# Claude Code
npx skills add https://github.com/cloudflare/skills

# OpenCode
opencode skills install cloudflare
```

Skills disponibles: `cloudflare`, `workers-best-practices`, `wrangler`, `durable-objects`, `agents-sdk`, `building-ai-agent`, `building-mcp-server`, `sandbox-sdk`, `web-perf`, `find-skills`

---

[Volver al índice principal](../README.md)
