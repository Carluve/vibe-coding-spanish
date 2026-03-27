# ⚡ Developer Platform

La plataforma serverless de Cloudflare te permite construir aplicaciones completas — APIs, sitios web, bases de datos, colas de mensajes — que corren directamente en el edge global. Tu código se despliega en más de **330 ciudades** simultáneamente, con arranque en frío de **menos de 5ms**.

---

## ¿Qué hace diferente a Cloudflare Workers?

A diferencia de AWS Lambda o Google Cloud Functions, Workers no usa contenedores. Usa el mismo motor de JavaScript que Chrome (V8), lo que significa:

- **Sin arranque en frío** perceptible — el Worker está listo en ~1ms
- **Límite de memoria bajo** — 128 MB por Worker (diseñado para eficiencia, no para cómputo pesado)
- **Entorno restringido** — no tienes acceso al sistema de ficheros ni a Node.js APIs nativas
- **APIs web estándar** — `fetch`, `Request`, `Response`, `crypto`, `WebSocket`... lo mismo que en el navegador

> 💡 Piensa en un Worker como en una función que recibe una `Request` HTTP y devuelve una `Response`. Todo lo demás es contexto.

---

## Repositorios oficiales

| Repositorio | ⭐ | Descripción |
|-------------|-----|-------------|
| [cloudflare/workers-sdk](https://github.com/cloudflare/workers-sdk) | ~3.9k | SDK oficial + Wrangler CLI. El punto de entrada para todo |
| [cloudflare/workerd](https://github.com/cloudflare/workerd) | ~8k | El runtime JavaScript open source que impulsa Workers |
| [cloudflare/workers-rs](https://github.com/cloudflare/workers-rs) | ~3.4k | Escribe Workers en Rust compilado a WebAssembly |
| [cloudflare/templates](https://github.com/cloudflare/templates) | — | Templates oficiales listos para usar |
| [cloudflare/next-on-pages](https://github.com/cloudflare/next-on-pages) | ~1k | Despliega Next.js en Cloudflare Pages |
| [cloudflare/cloudflare-docs](https://github.com/cloudflare/cloudflare-docs) | ~5k | Toda la documentación oficial (CC BY 4.0, pull requests bienvenidos) |

---

## Documentación por producto

| Producto | Docs | Para qué sirve |
|----------|------|----------------|
| Workers | [developers.cloudflare.com/workers](https://developers.cloudflare.com/workers/) | Funciones serverless en el edge |
| Pages | [developers.cloudflare.com/pages](https://developers.cloudflare.com/pages/) | Hosting de apps con deploy desde Git |
| D1 | [developers.cloudflare.com/d1](https://developers.cloudflare.com/d1/) | Base de datos SQLite serverless |
| R2 | [developers.cloudflare.com/r2](https://developers.cloudflare.com/r2/) | Object storage sin coste de egress |
| KV | [developers.cloudflare.com/kv](https://developers.cloudflare.com/kv/) | Clave-valor distribuido globalmente |
| Durable Objects | [developers.cloudflare.com/durable-objects](https://developers.cloudflare.com/durable-objects/) | Estado persistente y coordinación |
| Queues | [developers.cloudflare.com/queues](https://developers.cloudflare.com/queues/) | Colas de mensajes fiables |
| Workflows | [developers.cloudflare.com/workflows](https://developers.cloudflare.com/workflows/) | Orquestación de tareas duraderas |

---

## Inicio rápido con Wrangler

**Wrangler** es la CLI oficial para desarrollar, probar y desplegar Workers.

```bash
# Crea un proyecto nuevo con el asistente interactivo
npm create cloudflare@latest mi-proyecto

# El asistente te preguntará:
# ¿Qué quieres crear? → Workers, Pages, etc.
# ¿Qué framework? → Hono, vanilla JS, etc.
# ¿Desplegar ahora? → No (primero prueba en local)

cd mi-proyecto

# Desarrolla en local con hot-reload
npm run dev
# → Tu Worker está en http://localhost:8787

# Despliega a producción
npx wrangler deploy
# → https://mi-proyecto.tu-usuario.workers.dev
```

---

## Cloudflare Workers — Guía completa

### Estructura de un Worker

```typescript
// src/index.ts

export interface Env {
  // Aquí declaras los bindings (D1, R2, KV, etc.)
  DB: D1Database;
  MI_BUCKET: R2Bucket;
  API_KEY: string; // Variable de entorno
}

export default {
  // El método fetch se llama con cada petición HTTP
  async fetch(request: Request, env: Env, ctx: ExecutionContext): Promise<Response> {
    const url = new URL(request.url);

    // Enrutamiento básico
    if (url.pathname === "/") {
      return new Response("¡Hola desde el edge!", {
        headers: { "Content-Type": "text/plain; charset=utf-8" }
      });
    }

    return new Response("No encontrado", { status: 404 });
  },

  // Opcional: se ejecuta con un cron trigger
  async scheduled(event: ScheduledEvent, env: Env, ctx: ExecutionContext) {
    console.log("Cron ejecutado:", event.cron);
  },
};
```

### Enrutamiento con Hono (recomendado)

[Hono](https://hono.dev/) es un framework web ultraligero diseñado para Workers. Es el equivalente a Express.js para el edge.

```bash
npm create cloudflare@latest mi-api -- --template hono
```

```typescript
import { Hono } from "hono";
import { cors } from "hono/cors";
import { logger } from "hono/logger";

type Bindings = {
  DB: D1Database;
};

const app = new Hono<{ Bindings: Bindings }>();

// Middleware
app.use("*", logger());
app.use("/api/*", cors());

// Rutas
app.get("/", (c) => c.text("API funcionando"));

app.get("/api/productos", async (c) => {
  const { results } = await c.env.DB.prepare(
    "SELECT id, nombre, precio FROM productos WHERE activo = 1 ORDER BY nombre"
  ).all();
  return c.json(results);
});

app.post("/api/productos", async (c) => {
  const body = await c.req.json();
  const { nombre, precio } = body;

  if (!nombre || !precio) {
    return c.json({ error: "nombre y precio son obligatorios" }, 400);
  }

  const resultado = await c.env.DB.prepare(
    "INSERT INTO productos (nombre, precio, activo) VALUES (?, ?, 1)"
  )
    .bind(nombre, precio)
    .run();

  return c.json({ id: resultado.meta.last_row_id, nombre, precio }, 201);
});

export default app;
```

---

## D1 — Base de datos SQLite serverless

D1 es la base de datos de Cloudflare. Es SQLite ejecutándose en el edge, replicada globalmente en modo de lectura. Para escrituras, tiene un nodo primario al que se redirigen automáticamente.

### Cuándo usar D1

✅ Ideal para: apps con más lecturas que escrituras, datos relacionales, menos de ~10 GB
❌ No ideal para: cargas de escritura muy intensas, más de 10 GB de datos, transacciones complejas

### Crear y conectar una base de datos

```bash
# Crear la base de datos en Cloudflare
npx wrangler d1 create mi-base-de-datos

# Wrangler te devolverá algo como:
# ✅ Successfully created DB 'mi-base-de-datos'
# database_id = "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
```

Añade el binding en `wrangler.jsonc`:

```jsonc
{
  "name": "mi-worker",
  "main": "src/index.ts",
  "compatibility_date": "2024-09-23",
  "d1_databases": [
    {
      "binding": "DB",
      "database_name": "mi-base-de-datos",
      "database_id": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
    }
  ]
}
```

### Migraciones con D1

```sql
-- migrations/0001_crear_productos.sql
CREATE TABLE IF NOT EXISTS productos (
  id        INTEGER PRIMARY KEY AUTOINCREMENT,
  nombre    TEXT NOT NULL,
  precio    REAL NOT NULL CHECK(precio > 0),
  stock     INTEGER NOT NULL DEFAULT 0,
  activo    INTEGER NOT NULL DEFAULT 1,
  creado_en TEXT NOT NULL DEFAULT (datetime('now'))
);

CREATE INDEX idx_productos_activo ON productos(activo);
```

```bash
# Ejecutar en local (para desarrollo)
npx wrangler d1 execute mi-base-de-datos --local --file=migrations/0001_crear_productos.sql

# Ejecutar en producción
npx wrangler d1 execute mi-base-de-datos --file=migrations/0001_crear_productos.sql
```

### Consultas en el Worker

```typescript
// Una fila
const producto = await env.DB.prepare(
  "SELECT * FROM productos WHERE id = ?"
).bind(productoId).first();

// Múltiples filas
const { results } = await env.DB.prepare(
  "SELECT * FROM productos WHERE activo = 1"
).all();

// Insertar / actualizar / eliminar
const { success, meta } = await env.DB.prepare(
  "INSERT INTO productos (nombre, precio) VALUES (?, ?)"
).bind("Camiseta", 29.99).run();

console.log("ID insertado:", meta.last_row_id);

// Transacción
const statements = [
  env.DB.prepare("UPDATE stock SET cantidad = cantidad - ? WHERE producto_id = ?")
    .bind(1, productoId),
  env.DB.prepare("INSERT INTO pedidos (producto_id, cantidad) VALUES (?, ?)")
    .bind(productoId, 1),
];
await env.DB.batch(statements);
```

> ⚠️ **Importante:** Usa siempre `.bind()` para pasar valores. Nunca interpolación de strings — es vulnerable a SQL injection.

---

## R2 — Object Storage sin coste de egress

R2 es el equivalente a Amazon S3 de Cloudflare. Compatible con la API de S3, pero **sin coste de egress** (salida de datos). Ideal para almacenar imágenes, vídeos, backups o cualquier archivo.

### Crear un bucket

```bash
npx wrangler r2 bucket create mi-bucket
```

En `wrangler.jsonc`:

```jsonc
{
  "r2_buckets": [
    {
      "binding": "MI_BUCKET",
      "bucket_name": "mi-bucket"
    }
  ]
}
```

### Operaciones desde un Worker

```typescript
export default {
  async fetch(request: Request, env: Env): Promise<Response> {
    const url = new URL(request.url);
    // La clave del objeto es el path de la URL (sin la /)
    const key = url.pathname.slice(1);

    switch (request.method) {
      case "PUT": {
        // Subir archivo
        await env.MI_BUCKET.put(key, request.body, {
          httpMetadata: {
            contentType: request.headers.get("Content-Type") ?? "application/octet-stream",
          },
        });
        return new Response(`Archivo '${key}' subido`, { status: 200 });
      }

      case "GET": {
        // Descargar archivo
        const objeto = await env.MI_BUCKET.get(key);
        if (!objeto) {
          return new Response("Archivo no encontrado", { status: 404 });
        }
        return new Response(objeto.body, {
          headers: {
            "Content-Type": objeto.httpMetadata?.contentType ?? "application/octet-stream",
            "Cache-Control": "public, max-age=31536000",
            "ETag": objeto.etag,
          },
        });
      }

      case "DELETE": {
        await env.MI_BUCKET.delete(key);
        return new Response(null, { status: 204 });
      }

      default:
        return new Response("Método no permitido", { status: 405 });
    }
  },
};
```

### Listar objetos

```typescript
const listado = await env.MI_BUCKET.list({
  prefix: "imagenes/",   // Filtrar por prefijo (como una carpeta)
  limit: 100,
  cursor: cursor,        // Para paginación
});

for (const objeto of listado.objects) {
  console.log(objeto.key, objeto.size, objeto.uploaded);
}
```

---

## KV — Almacenamiento clave-valor global

Workers KV es un almacén clave-valor distribuido. Las escrituras se propagan globalmente en segundos, pero las lecturas son **eventualmente consistentes** (puede haber un retardo de hasta 60 segundos entre escritura y lectura en todos los nodos).

> 💡 **Cuándo usar KV vs D1:** Usa KV para datos que se leen muchísimo y se escriben poco (configuración, feature flags, sesiones). Usa D1 cuando necesitas relaciones, consultas complejas o consistencia inmediata.

```bash
npx wrangler kv namespace create MI_KV
```

```typescript
// Leer (devuelve null si no existe)
const valor = await env.MI_KV.get("usuario:123");

// Leer como JSON
const config = await env.MI_KV.get<Config>("app:config", { type: "json" });

// Escribir con TTL de 1 hora
await env.MI_KV.put("sesion:abc", JSON.stringify(datos), {
  expirationTtl: 3600,
});

// Eliminar
await env.MI_KV.delete("sesion:abc");

// Listar claves con prefijo
const lista = await env.MI_KV.list({ prefix: "sesion:" });
for (const clave of lista.keys) {
  console.log(clave.name, clave.expiration);
}
```

---

## Queues — Colas de mensajes

Workers Queues permite enviar mensajes entre Workers de forma fiable. Garantiza entrega **al menos una vez** y reintenta automáticamente si el Consumer falla.

```bash
npx wrangler queues create mi-cola
```

```jsonc
{
  "queues": {
    "producers": [{ "queue": "mi-cola", "binding": "MI_COLA" }],
    "consumers": [{ "queue": "mi-cola", "max_batch_size": 10 }]
  }
}
```

```typescript
// Producer — envía mensajes a la cola
export default {
  async fetch(request: Request, env: Env) {
    await env.MI_COLA.send({
      tipo: "email_bienvenida",
      usuario_id: 123,
      email: "usuario@ejemplo.com",
    });
    return new Response("Mensaje enviado a la cola");
  },
};

// Consumer — procesa mensajes de la cola
export default {
  async queue(batch: MessageBatch<{ tipo: string; email: string }>, env: Env) {
    for (const mensaje of batch.messages) {
      try {
        await enviarEmail(mensaje.body.email);
        mensaje.ack(); // Confirma que se procesó correctamente
      } catch (error) {
        mensaje.retry(); // Reintenta más tarde
      }
    }
  },
};
```

---

## Cloudflare Pages

Pages es la plataforma de hosting para frontend y apps full-stack. Se integra directamente con GitHub/GitLab: cada push genera un deploy, cada PR genera una preview URL.

```bash
# Conectar repo desde el dashboard (recomendado)
# O desplegar manualmente:
npx wrangler pages deploy ./dist --project-name=mi-sitio
```

### Frameworks con soporte oficial

| Framework | Adaptador necesario |
|-----------|-------------------|
| Astro | Adaptador Cloudflare (oficial) |
| Next.js | `@cloudflare/next-on-pages` |
| Remix | Adaptador Cloudflare (oficial) |
| SvelteKit | Adaptador Cloudflare (oficial) |
| Nuxt | Adaptador Cloudflare (oficial) |
| Qwik | Adaptador Cloudflare (oficial) |
| Solid Start | Adaptador Cloudflare (oficial) |

### Functions — Backend en Pages

Crea archivos en `functions/` y se convierten en Workers automáticamente:

```typescript
// functions/api/usuarios/[id].ts
import type { PagesFunction } from "@cloudflare/workers-types";

interface Env {
  DB: D1Database;
}

export const onRequestGet: PagesFunction<Env> = async (context) => {
  const { id } = context.params;
  const usuario = await context.env.DB.prepare(
    "SELECT id, nombre, email FROM usuarios WHERE id = ?"
  ).bind(id).first();

  if (!usuario) {
    return Response.json({ error: "Usuario no encontrado" }, { status: 404 });
  }

  return Response.json(usuario);
};
```

---

## Configuración recomendada de `wrangler.jsonc`

```jsonc
{
  // Nombre del Worker (aparece en el dashboard)
  "name": "mi-api",

  // Punto de entrada
  "main": "src/index.ts",

  // Fecha de compatibilidad — actualiza periódicamente
  "compatibility_date": "2024-09-23",

  // Flags opcionales para habilitar APIs más nuevas
  "compatibility_flags": ["nodejs_compat"],

  // Variables de entorno (no secretos — usa wrangler secret para eso)
  "vars": {
    "ENTORNO": "produccion"
  },

  // Base de datos D1
  "d1_databases": [
    {
      "binding": "DB",
      "database_name": "mi-api-db",
      "database_id": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
    }
  ],

  // Object storage R2
  "r2_buckets": [
    { "binding": "ARCHIVOS", "bucket_name": "mi-api-archivos" }
  ],

  // KV Namespace
  "kv_namespaces": [
    { "binding": "CACHE", "id": "xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx" }
  ],

  // Cola de mensajes
  "queues": {
    "producers": [{ "queue": "mi-cola", "binding": "COLA" }]
  },

  // Cron triggers
  "triggers": {
    "crons": ["0 4 * * *"] // Cada día a las 4:00 AM UTC
  },

  // Configuración para entorno de staging (sobreescribe lo anterior)
  "env": {
    "staging": {
      "name": "mi-api-staging",
      "vars": { "ENTORNO": "staging" }
    }
  }
}
```

---

## Secretos y variables de entorno

```bash
# Añadir un secreto (se cifra y no aparece en wrangler.jsonc)
npx wrangler secret put API_KEY_EXTERNA
# → Te pedirá el valor por stdin

# Listar secretos configurados
npx wrangler secret list

# Eliminar un secreto
npx wrangler secret delete API_KEY_EXTERNA

# En staging
npx wrangler secret put API_KEY_EXTERNA --env staging
```

---

## Comandos útiles de Wrangler

```bash
# Ver logs en tiempo real de producción
npx wrangler tail

# Ver logs de staging
npx wrangler tail --env staging

# Desplegar a staging
npx wrangler deploy --env staging

# Ejecutar consulta en D1 (producción)
npx wrangler d1 execute mi-db --command "SELECT COUNT(*) FROM usuarios"

# Subir un archivo a R2
npx wrangler r2 object put mi-bucket/archivo.txt --file=./archivo.txt

# Listar objetos en R2
npx wrangler r2 object list mi-bucket

# Ver métricas del Worker
npx wrangler deployments list
```

---

[← Volver al índice](../README.md)
