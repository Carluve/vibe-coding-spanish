# Guías en Español

Documentación oficial en español y guías prácticas para desarrolladores hispanohablantes.

---

## Documentación oficial en español

| Recurso | Enlace | Descripción |
|---------|--------|-------------|
| Soporte en Español | [developers.cloudflare.com/support/other-languages/español](https://developers.cloudflare.com/support/other-languages/espa%C3%B1ol-espa%C3%B1a/) | Artículos de soporte traducidos al español |
| Centro de Recursos | [cloudflare.com/es-es/resource-hub](https://www.cloudflare.com/es-es/resource-hub/) | Guías de soluciones, informes y casos de uso |
| Centro de Aprendizaje | [cloudflare.com/es-es/learning](https://www.cloudflare.com/es-es/learning/) | Conceptos de seguridad, rendimiento y redes |
| Centro de Arquitectura | [cloudflare.com/es-la/architecture](https://www.cloudflare.com/es-la/architecture/) | Arquitecturas de referencia y diagramas técnicos |
| Portal de Impacto Social | [cloudflare.com/es-es/impact-portal/getting-started](https://www.cloudflare.com/es-es/impact-portal/getting-started/) | Primeros pasos con Cloudflare |

---

## Guías por nivel

### Nivel Principiante — Primeros Pasos

#### ¿Qué es Cloudflare?

Cloudflare actúa como un **proxy inverso** entre tus usuarios y tu servidor de origen. Cuando alguien visita tu sitio web, la petición pasa primero por la red global de Cloudflare:

- **330+ ciudades** en todo el mundo
- **477 Tb/s** de capacidad de red
- Protección DDoS gratuita en todos los planes
- Caché global de contenido estático
- Optimización de rendimiento automática

Todo esto sin necesidad de cambiar tu proveedor de hosting.

#### Añadir tu dominio en 5 pasos

1. Crea una cuenta en [dash.cloudflare.com/sign-up](https://dash.cloudflare.com/sign-up)
2. Haz clic en **Add a site** e introduce tu dominio
3. Cloudflare escaneará tus registros DNS automáticamente
4. Actualiza los **nameservers** en tu registrador de dominios con los que Cloudflare te asigne
5. Espera la propagación (normalmente minutos, máximo 48h)

> Documentación oficial: [Primeros Pasos](https://developers.cloudflare.com/fundamentals/setup/account-setup/)

#### Entender los modos de proxy

| Icono en Dashboard | Modo | Descripción |
|-------------------|------|-------------|
| Nube naranja | Proxied | El tráfico pasa por Cloudflare (recomendado) |
| Nube gris | DNS only | Cloudflare solo resuelve DNS, sin proxy |

Con la nube naranja activa obtienes: caché, WAF, DDoS y optimizaciones.

---

### Nivel Intermedio — Developer Platform

#### Tu primer Worker

```bash
# Crea el proyecto
npm create cloudflare@latest mi-primer-worker

# Elige "Hello World Worker" (TypeScript)
# Entra al directorio
cd mi-primer-worker

# Ejecuta en local con hot-reload
npx wrangler dev
```

Abre `http://localhost:8787` para ver tu Worker. Edita `src/index.ts` y los cambios se recargan automáticamente.

#### Entender los bindings

Los bindings conectan tu Worker a otros servicios de Cloudflare:

```typescript
interface Env {
  // Binding a D1 (base de datos)
  DB: D1Database;
  // Binding a R2 (almacenamiento)
  MI_BUCKET: R2Bucket;
  // Binding a KV
  MI_KV: KVNamespace;
  // Variable de entorno
  API_KEY: string;
}

export default {
  async fetch(request: Request, env: Env) {
    // Usa env.DB, env.MI_BUCKET, etc.
  },
};
```

#### Deploy a producción

```bash
# Primera vez — autenticarse
npx wrangler login

# Desplegar
npx wrangler deploy

# Ver logs en tiempo real
npx wrangler tail
```

---

### Nivel Avanzado — Agentic AI

#### Arquitectura de un Agente con Cloudflare

```
Usuario → Workers (HTTP/WS) → Durable Object (Agent)
                                      ↓
                               Estado en SQLite
                               Workers AI (LLM)
                               Herramientas (tools)
                               Scheduling (cron)
```

#### Construir un agente conversacional

```typescript
import { Agent, AgentNamespace } from "agents";

interface Env {
  MiAgente: AgentNamespace<MiAgente>;
}

export class MiAgente extends Agent<Env> {
  async onMessage(mensaje: string) {
    // Guardar en historial
    await this.sql`
      INSERT INTO mensajes (role, content, timestamp)
      VALUES ('user', ${mensaje}, datetime('now'))
    `;

    // Obtener historial
    const historial = await this.sql`
      SELECT role, content FROM mensajes ORDER BY timestamp
    `;

    // Llamar al LLM
    const respuesta = await this.env.AI.run(
      "@cf/meta/llama-3.1-8b-instruct",
      { messages: historial }
    );

    // Guardar respuesta
    await this.sql`
      INSERT INTO mensajes (role, content, timestamp)
      VALUES ('assistant', ${respuesta.response}, datetime('now'))
    `;

    return respuesta.response;
  }
}
```

#### Pipeline RAG (Retrieval Augmented Generation)

```typescript
// 1. Indexar documentos
async function indexar(texto: string, env: Env) {
  const embedding = await env.AI.run("@cf/baai/bge-small-en-v1.5", {
    text: [texto],
  });
  await env.VECTORIZE.upsert([{
    id: crypto.randomUUID(),
    values: embedding.data[0],
    metadata: { texto },
  }]);
}

// 2. Buscar y generar respuesta
async function responder(pregunta: string, env: Env) {
  // Buscar documentos relevantes
  const queryEmbedding = await env.AI.run("@cf/baai/bge-small-en-v1.5", {
    text: [pregunta],
  });
  const docs = await env.VECTORIZE.query(queryEmbedding.data[0], {
    topK: 3,
    returnMetadata: true,
  });

  // Generar respuesta con contexto
  const contexto = docs.matches
    .map(m => m.metadata?.texto)
    .join("\n\n");

  const respuesta = await env.AI.run("@cf/meta/llama-3.1-8b-instruct", {
    messages: [
      { role: "system", content: `Contexto:\n${contexto}` },
      { role: "user", content: pregunta },
    ],
  });

  return respuesta.response;
}
```

---

## Recursos adicionales en español

- **Cloudflare Blog** — artículos técnicos (algunos en español): [blog.cloudflare.com](https://blog.cloudflare.com/)
- **Cloudflare TV** — presentaciones y demos: [cloudflare.tv](https://cloudflare.tv/)
- **Foro de la comunidad** — preguntas en español aceptadas: [community.cloudflare.com](https://community.cloudflare.com/)

---

[Volver al índice principal](../README.md)
