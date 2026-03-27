# Developer Platform

La plataforma serverless de Cloudflare para construir aplicaciones full-stack en el edge. Tu código se ejecuta en más de 330 ciudades de todo el mundo, con latencia mínima para cualquier usuario.

---

## Repositorios oficiales — Core

| Repositorio | Estrellas | Descripción | Lenguaje |
|-------------|-----------|-------------|----------|
| [cloudflare/workers-sdk](https://github.com/cloudflare/workers-sdk) | ~3.9k | SDK oficial + **Wrangler CLI** — el punto de partida para Workers | TypeScript |
| [cloudflare/workerd](https://github.com/cloudflare/workerd) | ~8k | El runtime JavaScript/Wasm que impulsa Cloudflare Workers | C++ |
| [cloudflare/miniflare](https://github.com/cloudflare/miniflare) | ~3k | Simulador local de Workers (ahora integrado en Wrangler) | TypeScript |
| [cloudflare/workers-rs](https://github.com/cloudflare/workers-rs) | ~3.4k | Escribe Workers en Rust puro vía WebAssembly | Rust |
| [cloudflare/templates](https://github.com/cloudflare/templates) | — | Templates oficiales para Workers y Pages | Varios |
| [cloudflare/cloudflare-docs](https://github.com/cloudflare/cloudflare-docs) | ~5k | Documentación completa en código abierto | MDX |

## Repositorios oficiales — Frameworks

| Repositorio | Descripción | Lenguaje |
|-------------|-------------|----------|
| [cloudflare/next-on-pages](https://github.com/cloudflare/next-on-pages) | Despliega apps Next.js en Cloudflare Pages | TypeScript |
| [cloudflare/workers-for-platforms-example](https://github.com/cloudflare/workers-for-platforms-example) | Ejemplo multi-tenant para SaaS en Workers | TypeScript |

---

## Documentación por producto

| Producto | Documentación | Descripción |
|----------|--------------|-------------|
| Workers | [developers.cloudflare.com/workers](https://developers.cloudflare.com/workers/) | Funciones serverless en 330+ ciudades |
| Pages | [developers.cloudflare.com/pages](https://developers.cloudflare.com/pages/) | Hosting full-stack con deploy desde Git |
| D1 | [developers.cloudflare.com/d1](https://developers.cloudflare.com/d1/) | Base de datos SQLite serverless global |
| R2 | [developers.cloudflare.com/r2](https://developers.cloudflare.com/r2/) | Object storage sin egress fees (compatible S3) |
| KV | [developers.cloudflare.com/kv](https://developers.cloudflare.com/kv/) | Almacenamiento clave-valor distribuido globalmente |
| Durable Objects | [developers.cloudflare.com/durable-objects](https://developers.cloudflare.com/durable-objects/) | Estado persistente + coordinación con SQLite por instancia |
| Queues | [developers.cloudflare.com/queues](https://developers.cloudflare.com/queues/) | Cola de mensajes integrada con Workers |
| Workflows | [developers.cloudflare.com/workflows](https://developers.cloudflare.com/workflows/) | Orquestación de tareas multi-paso con reintentos automáticos |

---

## Inicio rápido con Wrangler

```bash
# Crea un proyecto nuevo con asistente interactivo
npm create cloudflare@latest mi-proyecto

# Entra al proyecto y ejecuta en local
cd mi-proyecto
npm run dev

# Despliega a producción
npx wrangler deploy
```

---

## Workers — Ejemplos prácticos

### Worker básico (TypeScript)

```typescript
export default {
  async fetch(request: Request, env: Env): Promise<Response> {
    const url = new URL(request.url);

    if (url.pathname === "/api/hola") {
      return Response.json({ mensaje: "¡Hola desde el edge!" });
    }

    return new Response("No encontrado", { status: 404 });
  },
};
```

### Worker con base de datos D1

```bash
# Crear la base de datos
npx wrangler d1 create mi-base-de-datos

# Añadir binding en wrangler.jsonc
# "d1_databases": [{ "binding": "DB", "database_name": "mi-base-de-datos", "database_id": "..." }]

# Ejecutar migraciones en local
npx wrangler d1 execute mi-base-de-datos --local --file=schema.sql

# Ejecutar en producción
npx wrangler d1 execute mi-base-de-datos --file=schema.sql
```

```typescript
export default {
  async fetch(request: Request, env: Env) {
    const { results } = await env.DB.prepare(
      "SELECT * FROM productos WHERE activo = 1"
    ).all();
    return Response.json(results);
  },
};
```

### Worker con almacenamiento R2

```typescript
export default {
  async fetch(request: Request, env: Env) {
    const { method, url } = request;
    const key = new URL(url).pathname.slice(1);

    if (method === "PUT") {
      await env.MI_BUCKET.put(key, request.body);
      return new Response("Archivo subido", { status: 200 });
    }

    if (method === "GET") {
      const objeto = await env.MI_BUCKET.get(key);
      if (!objeto) return new Response("No encontrado", { status: 404 });
      return new Response(objeto.body);
    }

    return new Response("Método no permitido", { status: 405 });
  },
};
```

### Worker con KV (clave-valor)

```typescript
export default {
  async fetch(request: Request, env: Env) {
    // Leer un valor
    const valor = await env.MI_KV.get("clave");

    // Escribir un valor (con TTL opcional en segundos)
    await env.MI_KV.put("clave", "valor", { expirationTtl: 3600 });

    return Response.json({ valor });
  },
};
```

---

## Cloudflare Pages

Deploy automático desde Git con preview por rama:

```bash
# Conecta tu repositorio desde el dashboard
# O usa Wrangler directamente
npx wrangler pages deploy ./dist --project-name=mi-sitio
```

Frameworks soportados con zero-config: **Astro, Next.js, Remix, SvelteKit, Nuxt, Vue, React, Angular**.

> Documentación: [Cloudflare Pages](https://developers.cloudflare.com/pages/)

---

[Volver al índice principal](../README.md)
