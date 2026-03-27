# 🇪🇸 Guías en Español

Guías prácticas ordenadas por nivel. Empieza por donde te corresponda según tu experiencia con Cloudflare.

---

## Documentación oficial en español

| Recurso | Enlace | Descripción |
|---------|--------|-------------|
| Soporte en Español | [/support/other-languages/español](https://developers.cloudflare.com/support/other-languages/espa%C3%B1ol-espa%C3%B1a/) | Artículos de soporte traducidos |
| Centro de Recursos | [cloudflare.com/es-es/resource-hub](https://www.cloudflare.com/es-es/resource-hub/) | Guías de soluciones e informes |
| Centro de Aprendizaje | [cloudflare.com/es-es/learning](https://www.cloudflare.com/es-es/learning/) | Conceptos de seguridad y redes |
| Centro de Arquitectura | [cloudflare.com/es-la/architecture](https://www.cloudflare.com/es-la/architecture/) | Arquitecturas de referencia |

---

## 🟢 Nivel Principiante — Primeros pasos con Cloudflare

### ¿Qué es Cloudflare y cómo funciona?

Cloudflare es una red global que actúa como intermediario entre tus usuarios y tu servidor. Cuando alguien visita tu sitio web, la petición no llega directamente a tu servidor — primero pasa por el datacenter de Cloudflare más cercano al visitante.

```
Usuario (Madrid) → Datacenter Cloudflare Madrid → Tu servidor (en AWS, Hetzner, etc.)
```

Esto tiene varias ventajas:
- **Velocidad:** Cloudflare cachea el contenido estático (HTML, CSS, imágenes) y lo sirve desde el nodo más cercano al usuario.
- **Seguridad:** Los ataques DDoS y peticiones maliciosas se filtran antes de llegar a tu servidor, que permanece oculto.
- **Disponibilidad:** Si tu servidor tiene un pico de tráfico, Cloudflare absorbe gran parte del impacto.
- **Gratis en el plan Free:** CDN, DDoS básico, SSL, DNS autoritativo — todo sin coste.

### Añadir tu dominio (paso a paso)

**Requisitos:** Tener un dominio registrado en cualquier registrador (Namecheap, GoDaddy, Ionos, etc.)

**Paso 1 — Crear cuenta**

Ve a [dash.cloudflare.com/sign-up](https://dash.cloudflare.com/sign-up) y crea una cuenta gratuita. Solo necesitas email y contraseña.

**Paso 2 — Añadir el dominio**

En el dashboard, haz clic en **Add a site**, escribe tu dominio (ej: `miempresa.es`) y pulsa **Add site**.

**Paso 3 — Elegir plan**

Para la mayoría de proyectos personales y pequeñas empresas, el plan **Free** es suficiente. Incluye:
- CDN global
- Protección DDoS
- SSL automático
- DNS autoritativo
- Firewall básico

**Paso 4 — Revisar registros DNS**

Cloudflare escaneará automáticamente los registros DNS de tu dominio e intentará importarlos. Revisa que estén todos y que sean correctos antes de continuar.

**Paso 5 — Cambiar nameservers**

Cloudflare te dará dos nameservers propios (algo como `ada.ns.cloudflare.com`). Debes entrar en el panel de tu registrador de dominios y reemplazar los nameservers actuales con los de Cloudflare.

Este paso varía según el registrador:
- **Namecheap:** Domain List → Manage → Nameservers → Custom DNS
- **GoDaddy:** My Products → DNS → Nameservers → Change
- **Ionos:** Dominios → (tu dominio) → Nameservers

**Paso 6 — Esperar la propagación**

El cambio de nameservers tarda entre unos minutos y 48 horas en propagarse por Internet. En la práctica, suele ser menos de 1 hora.

Cuando Cloudflare detecte el cambio, el estado de tu dominio en el dashboard pasará a **Active** y recibirás un email de confirmación.

> 📖 Documentación oficial: [Primeros pasos](https://developers.cloudflare.com/fundamentals/setup/account-setup/)

---

### Entender el proxy de Cloudflare (nube naranja vs gris)

En la sección DNS del dashboard verás que cada registro tiene un ícono de nube:

**🟠 Nube naranja (Proxied):** El tráfico pasa por Cloudflare. Activa CDN, WAF, DDoS y todas las optimizaciones. La IP de tu servidor queda oculta — los visitantes solo ven IPs de Cloudflare.

**⚪ Nube gris (DNS only):** Cloudflare solo resuelve el DNS. El tráfico va directamente a tu servidor, sin protección ni caché.

**Cuándo usar cada uno:**
- Nube naranja: registros `A` y `CNAME` de tu web (`www`, `@`, `app`...)
- Nube gris: registros de email (`mail`, `smtp`), subdominios internos, VPNs, o servicios que necesiten la IP real

---

### Configurar SSL/TLS

Con Cloudflare, el SSL es automático — no necesitas comprar ni renovar certificados. Pero hay que configurar el **modo de cifrado** correctamente:

| Modo | Descripción | Cuándo usarlo |
|------|-------------|---------------|
| Off | Sin SSL | ❌ Nunca — inseguro |
| Flexible | Cloudflare ↔ Usuario: HTTPS / Cloudflare ↔ Servidor: HTTP | Solo si tu servidor no puede usar SSL (no recomendado) |
| Full | Ambos tramos HTTPS, pero Cloudflare no valida el certificado del servidor | Tu servidor tiene SSL autofirmado |
| Full (strict) | Ambos HTTPS, Cloudflare valida el certificado | **Recomendado** — tu servidor tiene SSL válido (Let's Encrypt, etc.) |

Ve a **SSL/TLS → Overview** en el dashboard y selecciona el modo adecuado.

> ⚠️ El modo **Flexible** puede provocar bucles de redirección si tu servidor ya redirige HTTP a HTTPS. Si tienes ese problema, cambia a **Full**.

---

## 🟡 Nivel Intermedio — Developer Platform

Para este nivel asumimos que ya tienes tu dominio en Cloudflare y quieres construir APIs o aplicaciones serverless.

### Tu primer Worker — Explicado paso a paso

```bash
# Crear el proyecto
npm create cloudflare@latest mi-primer-worker

# El asistente te pregunta:
# "What would you like to create?" → Hello World Worker
# "Which language?" → TypeScript
# "Do you want to deploy?" → No (primero prueba en local)

cd mi-primer-worker
```

Abre `src/index.ts`. Verás algo así:

```typescript
export default {
  async fetch(request: Request, env: Env, ctx: ExecutionContext): Promise<Response> {
    return new Response("Hello World!");
  },
};
```

Este es el Worker más simple posible. Cuando llega cualquier petición HTTP, devuelve "Hello World!". Vamos a mejorarlo paso a paso.

**Paso 1 — Añadir enrutamiento básico:**

```typescript
export default {
  async fetch(request: Request, env: Env): Promise<Response> {
    const url = new URL(request.url);
    const method = request.method;

    // GET / → página de inicio
    if (url.pathname === "/" && method === "GET") {
      return new Response("¡Bienvenido a mi API!", {
        headers: { "Content-Type": "text/plain; charset=utf-8" },
      });
    }

    // GET /saludo/:nombre
    if (url.pathname.startsWith("/saludo/") && method === "GET") {
      const nombre = url.pathname.slice("/saludo/".length);
      return Response.json({ mensaje: `¡Hola, ${nombre}!` });
    }

    // 404 para todo lo demás
    return Response.json({ error: "Ruta no encontrada" }, { status: 404 });
  },
};
```

**Paso 2 — Arrancar en local:**

```bash
npm run dev
# → Worker disponible en http://localhost:8787
```

Prueba en el navegador o con curl:
```bash
curl http://localhost:8787/saludo/María
# → {"mensaje":"¡Hola, María!"}
```

**Paso 3 — Desplegar:**

```bash
npx wrangler login      # Solo la primera vez
npx wrangler deploy

# → https://mi-primer-worker.TU-USUARIO.workers.dev
```

### Añadir una base de datos D1

D1 es SQLite en el edge. Para añadirla a tu Worker:

```bash
# Crear la base de datos
npx wrangler d1 create mi-db
# → Copia el database_id que te devuelve
```

```jsonc
// wrangler.jsonc — añade el binding
{
  "d1_databases": [
    {
      "binding": "DB",          // Nombre con el que accedes en el código
      "database_name": "mi-db",
      "database_id": "PEGA-AQUÍ-EL-ID"
    }
  ]
}
```

```sql
-- migrations/0001_inicio.sql
CREATE TABLE IF NOT EXISTS tareas (
  id         INTEGER PRIMARY KEY AUTOINCREMENT,
  titulo     TEXT NOT NULL,
  completada INTEGER NOT NULL DEFAULT 0,
  creado_en  TEXT NOT NULL DEFAULT (datetime('now'))
);
```

```bash
# Ejecutar migración en local
npx wrangler d1 execute mi-db --local --file=migrations/0001_inicio.sql

# Ejecutar en producción
npx wrangler d1 execute mi-db --file=migrations/0001_inicio.sql
```

```typescript
// Ahora el tipo Env tiene acceso a DB
interface Env {
  DB: D1Database;
}

export default {
  async fetch(request: Request, env: Env): Promise<Response> {
    const url = new URL(request.url);

    if (url.pathname === "/tareas" && request.method === "GET") {
      const { results } = await env.DB.prepare(
        "SELECT * FROM tareas ORDER BY creado_en DESC"
      ).all();
      return Response.json(results);
    }

    if (url.pathname === "/tareas" && request.method === "POST") {
      const { titulo } = await request.json<{ titulo: string }>();
      await env.DB.prepare("INSERT INTO tareas (titulo) VALUES (?)").bind(titulo).run();
      return Response.json({ ok: true }, { status: 201 });
    }

    return Response.json({ error: "No encontrado" }, { status: 404 });
  },
};
```

---

## 🔴 Nivel Avanzado — Agentic AI

### Construir un chatbot con memoria persistente

Un agente de Cloudflare es diferente a una función serverless normal: **recuerda** las conversaciones anteriores porque su estado vive en un Durable Object con SQLite.

```bash
npx create-cloudflare@latest mi-chatbot --template cloudflare/agents-starter
cd mi-chatbot
npm install

# Añade tu API key de Anthropic
echo "ANTHROPIC_API_KEY=sk-ant-..." >> .dev.vars

npm run dev
# → Abre http://localhost:5173
```

El template ya incluye la interfaz de chat, streaming de respuestas y gestión del historial. Para personalizarlo:

```typescript
// src/agent.ts — modifica el prompt del sistema
const system = `Eres un asistente de soporte para Acme Corp.
Tienes acceso al historial completo de conversaciones con este usuario.
Responde siempre en español y de forma concisa.
Si el usuario pregunta algo que no sabes, díselo honestamente.`;
```

### Añadir tu propia base de datos al agente

Los agentes pueden acceder a bindings de Cloudflare (D1, R2, KV) además de su SQLite interno:

```typescript
import { Agent } from "agents";

interface Env {
  MiChatbot: AgentNamespace<MiChatbot>;
  DB: D1Database;              // Tu base de datos de negocio
  ANTHROPIC_API_KEY: string;
}

export class MiChatbot extends Agent<Env> {
  async onMessage(mensaje: string) {
    // El agente puede consultar tu D1
    const { results } = await this.env.DB.prepare(
      "SELECT * FROM productos WHERE activo = 1 LIMIT 10"
    ).all();

    // Y usar esos datos para responder al usuario
    // ...
  }
}
```

### Arquitectura RAG — Chatbot que "sabe" tu documentación

El patrón RAG (Retrieval Augmented Generation) permite que el agente responda preguntas basándose en tu propia documentación, sin necesidad de hacer fine-tuning.

```
Pregunta del usuario
      ↓
Convertir a embedding (vector numérico)
      ↓
Buscar los documentos más similares en Vectorize
      ↓
Incluir esos documentos como contexto en el prompt
      ↓
El LLM genera una respuesta basada en TU documentación
```

Consulta la [guía completa de Agentic AI](agentic-ai.md#pipeline-rag-completo) para el código paso a paso.

---

[← Volver al índice](../README.md)
