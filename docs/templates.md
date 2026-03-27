# Templates y Starters

Templates oficiales para empezar proyectos en Cloudflare Workers, Pages y Agents sin configuración manual.

---

## Cómo usar un template

```bash
# Crear proyecto con asistente interactivo
npm create cloudflare@latest mi-proyecto

# O especificar un template directamente
npx create-cloudflare@latest -- --template <repositorio>
```

---

## Templates oficiales

| Template | Stack | Descripción | Comando |
|----------|-------|-------------|---------|
| Agents Starter | React + Workers AI | Agente IA completo con chat, tools y scheduling | `--template cloudflare/agents-starter` |
| Hono + D1 | Hono, TypeScript | API REST con base de datos SQLite y tests | Seleccionar en asistente |
| Next.js on Workers | Next.js 14+ | App full-stack Next.js en el edge | `--template cloudflare/next-on-pages` |
| Astro | Astro | Sitio estático/SSR con Astro en Pages | Seleccionar en asistente |
| React Router | React | App SPA con React Router en Pages | Seleccionar en asistente |
| SvelteKit | SvelteKit | App full-stack SvelteKit en Pages | Seleccionar en asistente |
| Worker básico | TypeScript | Worker vacío con tipos y Vitest | Seleccionar en asistente |
| R2 Explorer | React | Interfaz tipo Google Drive para buckets R2 | Repositorio en templates |

> Lista completa: [developers.cloudflare.com/workers/get-started/quickstarts](https://developers.cloudflare.com/workers/get-started/quickstarts/)

---

## Agents Starter — El más completo para IA

El template recomendado para construir agentes IA con Cloudflare:

```bash
npx create-cloudflare@latest --template cloudflare/agents-starter
cd agents-starter
npm install
npm run dev
```

Incluye de serie:
- Chat con streaming y gestión de historial
- Herramientas (tools) para el agente
- Scheduling — tareas programadas sin usuario
- Soporte para visión (imágenes)
- Workers AI configurado (sin API key en local)
- React frontend con Tailwind CSS

---

## Repositorio de templates oficiales

El repositorio [cloudflare/templates](https://github.com/cloudflare/templates) contiene todos los templates mantenidos por el equipo de Cloudflare.

```bash
# Clonar para ver todos los templates
git clone https://github.com/cloudflare/templates
ls templates/
```

---

## Estructura típica de un proyecto Workers

```
mi-proyecto/
├── src/
│   └── index.ts        # Punto de entrada del Worker
├── test/
│   └── index.spec.ts   # Tests con Vitest
├── wrangler.jsonc       # Configuración de Wrangler
├── package.json
└── tsconfig.json
```

### `wrangler.jsonc` básico

```jsonc
{
  "name": "mi-proyecto",
  "main": "src/index.ts",
  "compatibility_date": "2024-01-01",

  // Base de datos D1
  "d1_databases": [
    {
      "binding": "DB",
      "database_name": "mi-db",
      "database_id": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
    }
  ],

  // Bucket R2
  "r2_buckets": [
    { "binding": "MI_BUCKET", "bucket_name": "mi-bucket" }
  ],

  // KV Namespace
  "kv_namespaces": [
    { "binding": "MI_KV", "id": "xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx" }
  ]
}
```

---

[Volver al índice principal](../README.md)
