# 🤖 Agentic AI

Cloudflare ofrece una plataforma completa para construir aplicaciones de IA: modelos serverless en GPUs (Workers AI), agentes con estado persistente (Agents SDK), búsqueda semántica (Vectorize), proxy inteligente (AI Gateway) y servidores de herramientas (MCP). Todo corriendo en el edge global.

---

## Arquitectura general

```
                    ┌─────────────────────────────────┐
                    │         Tu aplicación            │
                    └───────────────┬─────────────────┘
                                    │
                    ┌───────────────▼─────────────────┐
                    │          AI Gateway              │  ← Logs, caché, rate limiting
                    └───────────────┬─────────────────┘
                                    │
              ┌─────────────────────┼──────────────────────┐
              │                     │                       │
   ┌──────────▼──────┐  ┌──────────▼──────┐  ┌────────────▼────────┐
   │   Workers AI    │  │    Anthropic     │  │     OpenAI / etc    │
   │  (Llama, etc.)  │  │    (Claude)      │  │                     │
   └─────────────────┘  └─────────────────┘  └─────────────────────┘

   ┌─────────────────────────────────────────────────────────────────┐
   │                      Agents SDK                                  │
   │  ┌─────────────────┐  ┌──────────────┐  ┌────────────────────┐ │
   │  │  Durable Object │  │  Scheduling  │  │    WebSockets      │ │
   │  │  (estado + SQL) │  │  (cron/sleep)│  │  (streaming chat)  │ │
   │  └─────────────────┘  └──────────────┘  └────────────────────┘ │
   └─────────────────────────────────────────────────────────────────┘

   ┌─────────────┐    ┌─────────────┐    ┌─────────────────────────┐
   │  Vectorize  │    │  MCP Server │    │     AI Security         │
   │  (RAG/embeddings) │  │ (herramientas)│  │  (descubrimiento + DLP)│
   └─────────────┘    └─────────────┘    └─────────────────────────┘
```

---

## Repositorios oficiales

| Repositorio | ⭐ | Descripción |
|-------------|-----|-------------|
| [cloudflare/agents](https://github.com/cloudflare/agents) | ~1.2k | Agents SDK — framework completo para agentes con estado |
| [cloudflare/agents-starter](https://github.com/cloudflare/agents-starter) | — | Template de inicio: chat, tools, scheduling, vision |
| [cloudflare/workers-mcp](https://github.com/cloudflare/workers-mcp) | ~270+ | Servidor MCP en Workers con soporte OAuth |
| [cloudflare/skills](https://github.com/cloudflare/skills) | — | Skills para Claude Code, OpenCode y otros editores |

---

## Documentación por producto

| Producto | Docs | Descripción |
|----------|------|-------------|
| Workers AI | [developers.cloudflare.com/workers-ai](https://developers.cloudflare.com/workers-ai/) | Inferencia serverless en GPUs — LLMs, embeddings, imágenes, ASR |
| AI Gateway | [developers.cloudflare.com/ai-gateway](https://developers.cloudflare.com/ai-gateway/) | Proxy, caché, logs y rate limiting para cualquier proveedor |
| Vectorize | [developers.cloudflare.com/vectorize](https://developers.cloudflare.com/vectorize/) | Base de datos vectorial para búsqueda semántica y RAG |
| Agents SDK | [developers.cloudflare.com/agents](https://developers.cloudflare.com/agents/) | Agentes con estado, scheduling, WebSockets, herramientas |
| MCP | [developers.cloudflare.com/agents/model-context-protocol](https://developers.cloudflare.com/agents/model-context-protocol/) | Servidores MCP remotos con OAuth en Workers |

---

## Workers AI — Inferencia serverless

Workers AI te permite ejecutar modelos de ML directamente en tus Workers, sin gestionar GPUs ni infraestructura. El modelo más cercano al usuario responde.

### Modelos disponibles (selección)

| Categoría | Modelo | Descripción |
|-----------|--------|-------------|
| LLM (chat) | `@cf/meta/llama-3.3-70b-instruct-fp8-fast` | El más potente y equilibrado |
| LLM (rápido) | `@cf/meta/llama-3.1-8b-instruct` | Respuestas más rápidas, menor coste |
| LLM (código) | `@cf/qwen/qwen2.5-coder-32b-instruct` | Especializado en generación de código |
| Embeddings | `@cf/baai/bge-small-en-v1.5` | Vectores para búsqueda semántica |
| Imágenes | `@cf/black-forest-labs/flux-1-schnell` | Generación de imágenes |
| Transcripción | `@cf/openai/whisper` | Audio a texto (ASR) |
| Traducción | `@cf/meta/m2m100-1.2b` | Traducción entre idiomas |

Catálogo completo: [developers.cloudflare.com/workers-ai/models](https://developers.cloudflare.com/workers-ai/models/)

### Inferencia básica en un Worker

```typescript
interface Env {
  AI: Ai; // Binding de Workers AI
}

export default {
  async fetch(request: Request, env: Env): Promise<Response> {
    const { pregunta } = await request.json<{ pregunta: string }>();

    const respuesta = await env.AI.run("@cf/meta/llama-3.3-70b-instruct-fp8-fast", {
      messages: [
        {
          role: "system",
          content: "Eres un asistente técnico especializado en Cloudflare. Responde siempre en español.",
        },
        {
          role: "user",
          content: pregunta,
        },
      ],
      // Parámetros opcionales
      max_tokens: 1024,
      temperature: 0.7,
    });

    return Response.json({ respuesta: respuesta.response });
  },
};
```

```jsonc
// wrangler.jsonc — añade el binding de AI
{
  "ai": { "binding": "AI" }
}
```

### Streaming de respuestas

Para respuestas largas, el streaming mejora mucho la experiencia de usuario:

```typescript
const stream = await env.AI.run(
  "@cf/meta/llama-3.3-70b-instruct-fp8-fast",
  {
    messages: [{ role: "user", content: pregunta }],
    stream: true,
  }
);

// stream es un ReadableStream compatible con SSE
return new Response(stream, {
  headers: {
    "Content-Type": "text/event-stream",
    "Cache-Control": "no-cache",
  },
});
```

### Generación de embeddings

```typescript
const resultado = await env.AI.run("@cf/baai/bge-small-en-v1.5", {
  text: ["primer documento sobre Workers", "segundo documento sobre D1"],
});

// resultado.data es un array de arrays de números (vectores de 384 dimensiones)
console.log(resultado.data[0].length); // 384
```

---

## Vectorize — Base de datos vectorial

Vectorize almacena embeddings para búsqueda semántica. Es la pieza central de una arquitectura RAG (Retrieval Augmented Generation).

### Crear un índice

```bash
npx wrangler vectorize create mi-indice \
  --dimensions=384 \           # Debe coincidir con el modelo de embeddings
  --metric=cosine              # cosine, euclidean o dot-product
```

```jsonc
// wrangler.jsonc
{
  "vectorize": [
    {
      "binding": "VECTORIZE",
      "index_name": "mi-indice"
    }
  ]
}
```

### Pipeline RAG completo

```typescript
interface Env {
  AI: Ai;
  VECTORIZE: VectorizeIndex;
}

// Paso 1: Indexar documentos (ejecutar una vez, o al actualizar contenido)
async function indexarDocumento(texto: string, metadata: object, env: Env) {
  // Generar embedding
  const { data } = await env.AI.run("@cf/baai/bge-small-en-v1.5", {
    text: [texto],
  });

  // Guardar en Vectorize
  await env.VECTORIZE.upsert([
    {
      id: crypto.randomUUID(),
      values: data[0],
      metadata: { texto, ...metadata },
    },
  ]);
}

// Paso 2: Responder preguntas con contexto
async function responderConRAG(pregunta: string, env: Env): Promise<string> {
  // Convertir la pregunta a vector
  const { data } = await env.AI.run("@cf/baai/bge-small-en-v1.5", {
    text: [pregunta],
  });

  // Buscar los 5 documentos más relevantes
  const resultados = await env.VECTORIZE.query(data[0], {
    topK: 5,
    returnMetadata: "all",
  });

  // Construir contexto con los documentos encontrados
  const contexto = resultados.matches
    .filter((m) => m.score > 0.7) // Solo si son suficientemente relevantes
    .map((m) => m.metadata?.texto as string)
    .join("\n\n---\n\n");

  if (!contexto) {
    return "No encontré información relevante para responder esa pregunta.";
  }

  // Generar respuesta con el contexto
  const respuesta = await env.AI.run("@cf/meta/llama-3.3-70b-instruct-fp8-fast", {
    messages: [
      {
        role: "system",
        content: `Responde basándote EXCLUSIVAMENTE en el siguiente contexto.
Si la respuesta no está en el contexto, dilo claramente.
Responde siempre en español.

CONTEXTO:
${contexto}`,
      },
      { role: "user", content: pregunta },
    ],
  });

  return respuesta.response;
}
```

---

## Agents SDK — Agentes con estado persistente

El Agents SDK es lo más diferencial de Cloudflare en el ecosistema de IA. Un **agente** es un Durable Object que mantiene estado (en SQLite), puede recibir mensajes, ejecutar herramientas, programar tareas futuras y comunicarse vía WebSocket.

### Inicio rápido

```bash
npx create-cloudflare@latest mi-agente --template cloudflare/agents-starter
cd mi-agente
npm install
npm run dev
# → http://localhost:5173
```

### Anatomía de un agente

```typescript
import { Agent } from "agents";
import { createAnthropic } from "@ai-sdk/anthropic";
import { generateText, tool } from "ai";
import { z } from "zod";

interface Env {
  MiAgente: AgentNamespace<MiAgente>;
  ANTHROPIC_API_KEY: string;
  DB: D1Database;
}

export class MiAgente extends Agent<Env> {
  // ─── Estado ───────────────────────────────────────────────────────────────
  // El agente tiene SQLite embebido. Puedes crear tablas y consultarlas.
  // El estado persiste entre peticiones y reinicios.

  async inicializarEstado() {
    await this.sql`
      CREATE TABLE IF NOT EXISTS conversaciones (
        id        INTEGER PRIMARY KEY AUTOINCREMENT,
        role      TEXT NOT NULL CHECK(role IN ('user', 'assistant', 'system')),
        content   TEXT NOT NULL,
        timestamp TEXT NOT NULL DEFAULT (datetime('now'))
      )
    `;
  }

  // ─── Peticiones HTTP ───────────────────────────────────────────────────────
  async onRequest(request: Request): Promise<Response> {
    const url = new URL(request.url);

    if (url.pathname === "/historial") {
      const mensajes = await this.sql`
        SELECT role, content, timestamp
        FROM conversaciones
        ORDER BY timestamp DESC
        LIMIT 50
      `;
      return Response.json(mensajes);
    }

    return new Response("No encontrado", { status: 404 });
  }

  // ─── Mensajes de chat ──────────────────────────────────────────────────────
  async onMessage(mensaje: string): Promise<void> {
    // Guardar mensaje del usuario
    await this.sql`
      INSERT INTO conversaciones (role, content) VALUES ('user', ${mensaje})
    `;

    // Obtener historial para contexto
    const historial = await this.sql<{ role: string; content: string }>`
      SELECT role, content FROM conversaciones
      ORDER BY timestamp DESC LIMIT 20
    `;

    // Llamar al LLM con herramientas
    const anthropic = createAnthropic({ apiKey: this.env.ANTHROPIC_API_KEY });

    const { text } = await generateText({
      model: anthropic("claude-sonnet-4-5"),
      system: "Eres un asistente útil. Responde siempre en español.",
      messages: historial.results.reverse().map((m) => ({
        role: m.role as "user" | "assistant",
        content: m.content,
      })),
      tools: {
        obtenerHora: tool({
          description: "Obtiene la hora y fecha actuales",
          parameters: z.object({}),
          execute: async () => new Date().toLocaleString("es-ES"),
        }),
        buscarEnDB: tool({
          description: "Busca información en la base de datos de la empresa",
          parameters: z.object({
            consulta: z.string().describe("Término de búsqueda"),
          }),
          execute: async ({ consulta }) => {
            const resultados = await this.env.DB.prepare(
              "SELECT * FROM productos WHERE nombre LIKE ? LIMIT 5"
            )
              .bind(`%${consulta}%`)
              .all();
            return resultados.results;
          },
        }),
      },
      maxSteps: 5, // Máximo de llamadas a herramientas por respuesta
    });

    // Guardar respuesta del asistente
    await this.sql`
      INSERT INTO conversaciones (role, content) VALUES ('assistant', ${text})
    `;

    // Emitir la respuesta por WebSocket a los clientes conectados
    this.broadcast(JSON.stringify({ role: "assistant", content: text }));
  }

  // ─── Scheduling ────────────────────────────────────────────────────────────
  // El agente puede programar sus propias tareas futuras
  async onSchedule(schedule: Schedule): Promise<void> {
    // Se ejecuta cuando el agente lo programó con this.schedule(...)
    const tarea = schedule.payload as { tipo: string };

    if (tarea.tipo === "informe_diario") {
      await this.generarInformeDiario();
    }
  }

  private async generarInformeDiario() {
    const stats = await this.sql`
      SELECT COUNT(*) as total FROM conversaciones
      WHERE timestamp >= datetime('now', '-1 day')
    `;
    console.log("Conversaciones en las últimas 24h:", stats.results[0].total);

    // Programar el siguiente informe para mañana
    await this.schedule(
      new Date(Date.now() + 24 * 60 * 60 * 1000),
      "informe_diario"
    );
  }
}

// Worker principal que enruta las peticiones al agente correcto
export default {
  async fetch(request: Request, env: Env): Promise<Response> {
    // Cada usuario tiene su propio agente con su propio estado
    const usuarioId = request.headers.get("X-User-ID") ?? "anonimo";

    // Obtener (o crear) el Durable Object del agente para este usuario
    const agente = env.MiAgente.get(env.MiAgente.idFromName(usuarioId));

    return agente.fetch(request);
  },
};
```

```jsonc
// wrangler.jsonc
{
  "durable_objects": {
    "bindings": [
      { "name": "MiAgente", "class_name": "MiAgente" }
    ]
  },
  "migrations": [
    { "tag": "v1", "new_classes": ["MiAgente"] }
  ]
}
```

---

## MCP — Model Context Protocol

MCP es el protocolo estándar para que los LLMs usen herramientas externas. Cloudflare te permite crear servidores MCP en Workers y exponerlos vía HTTPS con autenticación OAuth.

### Crear un servidor MCP

```bash
npx create-cloudflare@latest mi-mcp --template cloudflare/workers-mcp
cd mi-mcp
npm install
```

```typescript
import { WorkerEntrypoint } from "cloudflare:workers";
import { ProxyToSelf } from "workers-mcp";

export default class MiMCPServer extends WorkerEntrypoint<Env> {
  // Cada método público se convierte automáticamente en una herramienta MCP

  /**
   * Busca productos en la base de datos
   * @param nombre Nombre o parte del nombre del producto
   * @returns Lista de productos encontrados
   */
  async buscarProductos(nombre: string) {
    const { results } = await this.env.DB.prepare(
      "SELECT id, nombre, precio, stock FROM productos WHERE nombre LIKE ? AND activo = 1"
    )
      .bind(`%${nombre}%`)
      .all();

    return results;
  }

  /**
   * Consulta el estado de un pedido
   * @param pedidoId ID del pedido
   */
  async consultarPedido(pedidoId: number) {
    const pedido = await this.env.DB.prepare(
      "SELECT * FROM pedidos WHERE id = ?"
    )
      .bind(pedidoId)
      .first();

    if (!pedido) throw new Error(`Pedido ${pedidoId} no encontrado`);
    return pedido;
  }

  async fetch(request: Request): Promise<Response> {
    return new ProxyToSelf(this).fetch(request);
  }
}
```

Una vez desplegado, cualquier LLM compatible con MCP (Claude, GPT-4o, Gemini...) puede usar tus herramientas:

```bash
npx wrangler deploy
# → https://mi-mcp.tu-usuario.workers.dev

# Configurar en Claude Desktop (~/.claude/claude_desktop_config.json):
# {
#   "mcpServers": {
#     "mi-empresa": {
#       "command": "npx",
#       "args": ["mcp-remote", "https://mi-mcp.tu-usuario.workers.dev"]
#     }
#   }
# }
```

---

## Cloudflare Skills para Claude Code y OpenCode

```bash
# Claude Code
npx skills add https://github.com/cloudflare/skills

# OpenCode
opencode skills install cloudflare

# Verificar skills instalados
# En Claude Code: escribe /skills en el chat
```

---

[← Volver al índice](../README.md)
