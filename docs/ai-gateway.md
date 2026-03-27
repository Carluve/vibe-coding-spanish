# AI Gateway — Proxy de Inferencia

**AI Gateway** es una capa de proxy entre tus aplicaciones y los proveedores de IA. Todo el tráfico de inferencia pasa por Cloudflare, lo que te da observabilidad, caché, rate limiting y control de costes — sin cambiar casi nada en tu código.

---

## ¿Por qué usar AI Gateway?

| Problema sin AI Gateway | Solución con AI Gateway |
|------------------------|------------------------|
| Sin visibilidad de qué peticiones se hacen | Logs de cada request con prompt, respuesta, coste y latencia |
| Pagas por cada petición aunque sea repetida | Caché semántica — respuestas idénticas no consumen tokens |
| Un proveedor caído para toda la app | Fallbacks automáticos entre proveedores |
| Difícil limitar el uso por usuario | Rate limiting configurable |
| Las claves API viajan desde el cliente | La clave solo la conoce tu backend / AI Gateway |

---

## Documentación oficial

| Recurso | Enlace |
|---------|--------|
| AI Gateway | [developers.cloudflare.com/ai-gateway](https://developers.cloudflare.com/ai-gateway/) |
| Proveedores soportados | [/ai-gateway/providers](https://developers.cloudflare.com/ai-gateway/providers/) |
| Caché | [/ai-gateway/configuration/caching](https://developers.cloudflare.com/ai-gateway/configuration/caching/) |
| Rate Limiting | [/ai-gateway/configuration/rate-limiting](https://developers.cloudflare.com/ai-gateway/configuration/rate-limiting/) |
| Logs y Analytics | [/ai-gateway/observability](https://developers.cloudflare.com/ai-gateway/observability/) |

---

## Configuración inicial

1. Ve al [Dashboard de Cloudflare](https://dash.cloudflare.com/) → **AI** → **AI Gateway**
2. Haz clic en **Create Gateway**
3. Asigna un nombre (ej: `mi-app-gateway`)
4. Copia la URL base que se genera:
   ```
   https://gateway.ai.cloudflare.com/v1/{account_id}/{gateway_name}
   ```

---

## Proveedores soportados

AI Gateway es compatible con la mayoría de proveedores de IA:

| Proveedor | URL del Gateway |
|-----------|----------------|
| OpenAI | `.../openai` |
| Anthropic | `.../anthropic` |
| Google Vertex AI | `.../google-vertex-ai` |
| Google AI Studio | `.../google-ai-studio` |
| Workers AI | `.../workers-ai/v1` |
| Groq | `.../groq` |
| Mistral | `.../mistral` |
| Cohere | `.../cohere` |
| Perplexity | `.../perplexity-ai` |
| Replicate | `.../replicate` |
| HuggingFace | `.../huggingface` |
| Azure OpenAI | `.../azure-openai` |
| AWS Bedrock | `.../aws-bedrock` |

---

## Casos de uso

### 1. Pasar inferencia de OpenAI por AI Gateway

Cambia solo la `baseURL` — el resto del código no cambia:

```typescript
// Antes (directo a OpenAI)
import OpenAI from "openai";
const client = new OpenAI({ apiKey: process.env.OPENAI_API_KEY });

// Después (a través de AI Gateway)
import OpenAI from "openai";
const client = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
  baseURL: "https://gateway.ai.cloudflare.com/v1/{account_id}/{gateway_name}/openai",
});

// El resto del código es idéntico
const respuesta = await client.chat.completions.create({
  model: "gpt-4o",
  messages: [{ role: "user", content: "Hola" }],
});
```

### 2. Pasar inferencia de Anthropic por AI Gateway

```typescript
import Anthropic from "@anthropic-ai/sdk";

const client = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
  baseURL: "https://gateway.ai.cloudflare.com/v1/{account_id}/{gateway_name}/anthropic",
});

const mensaje = await client.messages.create({
  model: "claude-sonnet-4-6",
  max_tokens: 1024,
  messages: [{ role: "user", content: "¿Qué es Cloudflare Workers?" }],
});
```

### 3. Usar Workers AI via AI Gateway

Accede a modelos de Workers AI con la API compatible con OpenAI — útil para herramientas como OpenCode, Continue.dev o cualquier cliente OpenAI:

```typescript
import OpenAI from "openai";

const client = new OpenAI({
  apiKey: process.env.CLOUDFLARE_API_TOKEN,
  baseURL: "https://gateway.ai.cloudflare.com/v1/{account_id}/{gateway_name}/workers-ai/v1",
});

const respuesta = await client.chat.completions.create({
  model: "@cf/meta/llama-3.1-8b-instruct",
  messages: [{ role: "user", content: "Explica qué son los Durable Objects" }],
});

console.log(respuesta.choices[0].message.content);
```

### 4. Fallback automático entre proveedores

Si el proveedor principal falla, AI Gateway puede redirigir automáticamente al siguiente:

```typescript
// Usando el endpoint universal de AI Gateway
const respuesta = await fetch(
  "https://gateway.ai.cloudflare.com/v1/{account_id}/{gateway_name}",
  {
    method: "POST",
    headers: {
      "Content-Type": "application/json",
      "cf-aig-authorization": `Bearer ${process.env.CF_AIG_TOKEN}`,
    },
    body: JSON.stringify([
      // Intenta primero con Anthropic
      {
        provider: "anthropic",
        endpoint: "messages",
        headers: { "x-api-key": process.env.ANTHROPIC_API_KEY },
        query: {
          model: "claude-sonnet-4-6",
          max_tokens: 1024,
          messages: [{ role: "user", content: "Hola" }],
        },
      },
      // Si falla, usa Workers AI como fallback
      {
        provider: "workers-ai",
        endpoint: "@cf/meta/llama-3.1-8b-instruct",
        headers: { Authorization: `Bearer ${process.env.CLOUDFLARE_API_TOKEN}` },
        query: {
          messages: [{ role: "user", content: "Hola" }],
        },
      },
    ]),
  }
);
```

### 5. AI Gateway desde un Worker

Todo integrado en el edge — el Worker usa AI Gateway para llamar a proveedores externos:

```typescript
export default {
  async fetch(request: Request, env: Env): Promise<Response> {
    const { pregunta } = await request.json();

    // El Worker llama a Anthropic a través de AI Gateway
    const respuesta = await fetch(
      `https://gateway.ai.cloudflare.com/v1/${env.CF_ACCOUNT_ID}/${env.GATEWAY_NAME}/anthropic/v1/messages`,
      {
        method: "POST",
        headers: {
          "Content-Type": "application/json",
          "x-api-key": env.ANTHROPIC_API_KEY,
          "anthropic-version": "2023-06-01",
        },
        body: JSON.stringify({
          model: "claude-sonnet-4-6",
          max_tokens: 1024,
          messages: [{ role: "user", content: pregunta }],
        }),
      }
    );

    const datos = await respuesta.json();
    return Response.json({ respuesta: datos.content[0].text });
  },
};
```

---

## Caché semántica

Activa la caché en el dashboard de AI Gateway para evitar peticiones duplicadas:

- **Exact match cache** — respuestas idénticas reutilizadas directamente
- **Semantic cache** — peticiones similares reutilizan respuestas (basado en embeddings)

```typescript
// Puedes controlar el caché por petición
const respuesta = await client.chat.completions.create(
  { model: "gpt-4o", messages: [...] },
  {
    headers: {
      "cf-aig-cache-ttl": "3600",        // Cachea 1 hora
      "cf-aig-skip-cache": "false",      // Usa caché si existe
    },
  }
);
```

---

## Observabilidad

En el dashboard de AI Gateway puedes ver para cada petición:
- Modelo usado y proveedor
- Tokens consumidos (input / output)
- Coste estimado en USD
- Latencia en ms
- Si fue un cache hit o miss
- El prompt y la respuesta completa (configurable)

---

## Rate Limiting

Configura límites desde el dashboard:

- Por número de peticiones (req/min, req/día)
- Por tokens consumidos
- Por usuario (usando el header `cf-aig-user-id`)

```typescript
// Identificar usuario para rate limiting por usuario
const respuesta = await fetch(gateway_url, {
  headers: {
    "cf-aig-user-id": usuario_id,  // Tu ID de usuario interno
    // ...resto de headers
  },
});
```

---

[Volver al índice principal](../README.md)
