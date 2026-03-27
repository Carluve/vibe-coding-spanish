# 🔀 AI Gateway

AI Gateway es la capa de proxy inteligente de Cloudflare para tráfico de IA. Se sitúa entre tu aplicación y cualquier proveedor (OpenAI, Anthropic, Google, Workers AI...) y te da control total: logs de cada petición, caché para no pagar dos veces por la misma respuesta, rate limiting, y fallbacks automáticos si un proveedor cae.

```
Tu app → AI Gateway (Cloudflare) → OpenAI / Anthropic / Workers AI / ...
```

---

## ¿Por qué necesitas AI Gateway?

Sin AI Gateway, cada petición de IA en tu app es una caja negra: no sabes exactamente qué se preguntó, cuánto costó, si falló silenciosamente, ni si se está abusando del servicio.

| Sin AI Gateway | Con AI Gateway |
|---------------|---------------|
| Sin visibilidad de peticiones | Logs completos: prompt, respuesta, tokens, coste, latencia |
| Pagas por respuestas repetidas | Caché — respuestas idénticas no consumen tokens |
| Un proveedor caído = app rota | Fallbacks automáticos a proveedor secundario |
| Sin control de uso | Rate limiting por usuario, IP o global |
| API keys en el cliente | Las claves solo las conoce tu backend |
| Sin métricas de coste | Dashboard con coste acumulado por modelo |

---

## Documentación oficial

| Recurso | Enlace |
|---------|--------|
| Referencia completa | [developers.cloudflare.com/ai-gateway](https://developers.cloudflare.com/ai-gateway/) |
| Proveedores soportados | [/ai-gateway/providers](https://developers.cloudflare.com/ai-gateway/providers/) |
| Caché semántica | [/ai-gateway/configuration/caching](https://developers.cloudflare.com/ai-gateway/configuration/caching/) |
| Rate Limiting | [/ai-gateway/configuration/rate-limiting](https://developers.cloudflare.com/ai-gateway/configuration/rate-limiting/) |
| Observabilidad y logs | [/ai-gateway/observability](https://developers.cloudflare.com/ai-gateway/observability/) |
| Autenticación | [/ai-gateway/configuration/authentication](https://developers.cloudflare.com/ai-gateway/configuration/authentication/) |

---

## Proveedores soportados

| Proveedor | Sufijo URL |
|-----------|-----------|
| OpenAI | `/openai` |
| Anthropic | `/anthropic` |
| Google AI Studio | `/google-ai-studio` |
| Google Vertex AI | `/google-vertex-ai` |
| Cloudflare Workers AI | `/workers-ai/v1` |
| Groq | `/groq` |
| Mistral AI | `/mistral` |
| Cohere | `/cohere` |
| Perplexity | `/perplexity-ai` |
| Replicate | `/replicate` |
| HuggingFace | `/huggingface` |
| Azure OpenAI | `/azure-openai` |
| AWS Bedrock | `/aws-bedrock` |
| OpenRouter | `/openrouter` |
| Together AI | `/together-ai` |

---

## Configuración inicial (5 minutos)

1. Abre el [Dashboard de Cloudflare](https://dash.cloudflare.com/)
2. En el menú lateral: **AI → AI Gateway**
3. Haz clic en **Create Gateway**
4. Ponle un nombre descriptivo, por ejemplo `mi-app-produccion`
5. Guarda la URL que se genera — la usarás como `baseURL` en tu código:

```
https://gateway.ai.cloudflare.com/v1/{account_id}/{gateway_name}
```

Eso es todo. Ahora solo tienes que cambiar la `baseURL` en tu código.

---

## Uso: OpenAI a través de AI Gateway

El cambio es mínimo — solo la `baseURL`. El resto del código queda exactamente igual.

```typescript
import OpenAI from "openai";

const cliente = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
  // Antes: "https://api.openai.com/v1"
  // Después: apuntas al gateway
  baseURL: `https://gateway.ai.cloudflare.com/v1/${process.env.CF_ACCOUNT_ID}/${process.env.CF_GATEWAY_NAME}/openai`,
});

// El resto de tu código no cambia
const respuesta = await cliente.chat.completions.create({
  model: "gpt-4o",
  messages: [
    { role: "system", content: "Eres un asistente técnico de Cloudflare." },
    { role: "user", content: "¿Cómo configuro un Cloudflare Tunnel?" },
  ],
  temperature: 0.7,
});

console.log(respuesta.choices[0].message.content);
// ✅ La petición queda registrada en tu gateway con tokens, coste y latencia
```

---

## Uso: Anthropic (Claude) a través de AI Gateway

```typescript
import Anthropic from "@anthropic-ai/sdk";

const cliente = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
  baseURL: `https://gateway.ai.cloudflare.com/v1/${process.env.CF_ACCOUNT_ID}/${process.env.CF_GATEWAY_NAME}/anthropic`,
});

const mensaje = await cliente.messages.create({
  model: "claude-sonnet-4-5",
  max_tokens: 1024,
  system: "Responde siempre en español.",
  messages: [
    { role: "user", content: "Explica qué son los Durable Objects de Cloudflare." }
  ],
});

console.log(mensaje.content[0].text);
```

---

## Uso: Workers AI a través de AI Gateway

Esta es la combinación más interesante: modelos abiertos (Llama, Mistral, Qwen...) corriendo en la infraestructura de Cloudflare, con la interfaz compatible OpenAI, accesibles desde cualquier herramienta o editor.

```typescript
import OpenAI from "openai";

// Workers AI expone una API compatible con OpenAI cuando se accede via AI Gateway
const cliente = new OpenAI({
  apiKey: process.env.CLOUDFLARE_API_TOKEN,
  baseURL: `https://gateway.ai.cloudflare.com/v1/${process.env.CF_ACCOUNT_ID}/${process.env.CF_GATEWAY_NAME}/workers-ai/v1`,
});

const respuesta = await cliente.chat.completions.create({
  // Modelos disponibles en Workers AI: https://developers.cloudflare.com/workers-ai/models/
  model: "@cf/meta/llama-3.1-8b-instruct",
  messages: [
    { role: "user", content: "¿Cuál es la diferencia entre KV y D1 en Cloudflare?" }
  ],
});

console.log(respuesta.choices[0].message.content);
```

> 💡 Esta configuración es la que usan herramientas como **OpenCode**, **Continue.dev** o **Cursor** para apuntar a Workers AI como proveedor de inferencia local/privado.

---

## Fallbacks automáticos entre proveedores

El endpoint universal de AI Gateway permite definir una cadena de proveedores. Si el primero falla (timeout, error 429, error 500...), pasa automáticamente al siguiente.

```typescript
const respuesta = await fetch(
  `https://gateway.ai.cloudflare.com/v1/${accountId}/${gatewayName}`,
  {
    method: "POST",
    headers: {
      "Content-Type": "application/json",
      // Token de AI Gateway (distinto al token de cada proveedor)
      "cf-aig-authorization": `Bearer ${process.env.CF_AIG_TOKEN}`,
    },
    body: JSON.stringify([
      // 1er intento: Claude (Anthropic)
      {
        provider: "anthropic",
        endpoint: "messages",
        headers: {
          "x-api-key": process.env.ANTHROPIC_API_KEY,
          "anthropic-version": "2023-06-01",
        },
        query: {
          model: "claude-sonnet-4-5",
          max_tokens: 1024,
          messages: [{ role: "user", content: "Hola" }],
        },
      },
      // 2o intento si falla: Workers AI (gratuito dentro de límites)
      {
        provider: "workers-ai",
        endpoint: "@cf/meta/llama-3.1-8b-instruct",
        headers: {
          Authorization: `Bearer ${process.env.CLOUDFLARE_API_TOKEN}`,
        },
        query: {
          messages: [{ role: "user", content: "Hola" }],
        },
      },
    ]),
  }
);

const datos = await respuesta.json();
```

---

## AI Gateway dentro de un Worker

Combinar Workers + AI Gateway te da la arquitectura ideal: tu lógica en el edge, sin API keys expuestas al cliente, con observabilidad completa.

```typescript
// src/index.ts
interface Env {
  ANTHROPIC_API_KEY: string;
  CF_ACCOUNT_ID: string;
  CF_GATEWAY_NAME: string;
}

export default {
  async fetch(request: Request, env: Env): Promise<Response> {
    if (request.method !== "POST") {
      return new Response("Método no permitido", { status: 405 });
    }

    const { pregunta } = await request.json<{ pregunta: string }>();

    if (!pregunta?.trim()) {
      return Response.json({ error: "La pregunta no puede estar vacía" }, { status: 400 });
    }

    const gatewayUrl = `https://gateway.ai.cloudflare.com/v1/${env.CF_ACCOUNT_ID}/${env.CF_GATEWAY_NAME}/anthropic/v1/messages`;

    const respuesta = await fetch(gatewayUrl, {
      method: "POST",
      headers: {
        "Content-Type": "application/json",
        "x-api-key": env.ANTHROPIC_API_KEY,
        "anthropic-version": "2023-06-01",
      },
      body: JSON.stringify({
        model: "claude-sonnet-4-5",
        max_tokens: 1024,
        messages: [{ role: "user", content: pregunta }],
      }),
    });

    if (!respuesta.ok) {
      const error = await respuesta.text();
      console.error("Error en AI Gateway:", error);
      return Response.json({ error: "Error al procesar la petición" }, { status: 502 });
    }

    const datos = await respuesta.json();
    return Response.json({
      respuesta: datos.content[0].text,
      tokens: datos.usage,
    });
  },
};
```

---

## Caché — No pagues dos veces por la misma pregunta

### Caché exacta

Cuando dos peticiones tienen exactamente el mismo prompt, la segunda no llega al proveedor — se sirve desde la caché de Cloudflare.

```typescript
// Controlar el TTL de caché por petición (en segundos)
const respuesta = await cliente.chat.completions.create(
  {
    model: "gpt-4o",
    messages: [{ role: "user", content: "¿Qué es Cloudflare Workers?" }],
  },
  {
    headers: {
      "cf-aig-cache-ttl": "86400",    // Cachear 24 horas
      "cf-aig-skip-cache": "false",   // Usar caché si existe
    },
  }
);
```

### Caché semántica (beta)

La caché semántica usa embeddings para detectar preguntas similares aunque no sean idénticas:

- "¿Qué es un Worker?" y "Explícame qué son los Cloudflare Workers" pueden devolver la misma respuesta cacheada.

Se activa desde la configuración del gateway en el dashboard (no requiere cambios en el código).

### Cuándo NO usar caché

```typescript
// Para respuestas que deben ser siempre frescas (datos en tiempo real, generación creativa)
headers: {
  "cf-aig-skip-cache": "true",
}
```

---

## Rate Limiting

Configura límites desde el dashboard del gateway:

- **Por peticiones:** máximo N requests por minuto/hora/día
- **Por tokens:** máximo N tokens consumidos por período
- **Por usuario:** usando el header `cf-aig-user-id`

```typescript
// Identificar usuarios para rate limiting individual
const respuesta = await fetch(gatewayUrl, {
  method: "POST",
  headers: {
    "Content-Type": "application/json",
    "x-api-key": env.ANTHROPIC_API_KEY,
    "anthropic-version": "2023-06-01",
    // ID de tu usuario — el gateway aplica el límite per-usuario
    "cf-aig-user-id": usuarioId,
    "cf-aig-metadata": JSON.stringify({ plan: "free", region: "es" }),
  },
  body: JSON.stringify({ ... }),
});

// Si supera el límite, recibirás un 429
if (respuesta.status === 429) {
  return Response.json(
    { error: "Has superado el límite de uso. Inténtalo en unos minutos." },
    { status: 429 }
  );
}
```

---

## Observabilidad — Lo que verás en el dashboard

Para cada petición que pasa por tu gateway tienes acceso a:

| Campo | Descripción |
|-------|-------------|
| Timestamp | Cuándo ocurrió |
| Proveedor y modelo | Qué modelo se usó |
| Tokens de entrada | Tokens del prompt |
| Tokens de salida | Tokens de la respuesta |
| Coste estimado | En USD, basado en tarifas públicas del proveedor |
| Latencia | Tiempo total en ms |
| Cache hit/miss | Si se sirvió desde caché o no |
| Status | 200, 429, 500... |
| Prompt y respuesta | El contenido completo (configurable, puede desactivarse por privacidad) |

Puedes filtrar por modelo, proveedor, usuario, rango de fechas y exportar los logs.

---

## Variables de entorno necesarias

```bash
# En local (.dev.vars para Workers, o .env para Node.js)
CF_ACCOUNT_ID="tu-account-id"           # En dash.cloudflare.com → parte derecha del panel
CF_GATEWAY_NAME="mi-app-produccion"      # El nombre que pusiste al crear el gateway
CLOUDFLARE_API_TOKEN="tu-api-token"      # Con permisos de AI Gateway
OPENAI_API_KEY="sk-..."
ANTHROPIC_API_KEY="sk-ant-..."

# Añadir secretos al Worker en producción
npx wrangler secret put ANTHROPIC_API_KEY
npx wrangler secret put CF_ACCOUNT_ID
npx wrangler secret put CF_GATEWAY_NAME
```

---

[← Volver al índice](../README.md)
