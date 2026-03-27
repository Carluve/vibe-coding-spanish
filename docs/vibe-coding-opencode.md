# 🎵 Vibe Coding con OpenCode

**Vibe Coding** es programar describiendo lo que quieres en lenguaje natural. El agente de IA escribe el código, lo ejecuta, lee los errores y los corrige — tú guías, él teclea. [OpenCode](https://opencode.ai/) es un cliente open source de terminal que lleva este flujo a cualquier proyecto.

---

## ¿Qué es OpenCode?

OpenCode es una CLI de codificación con IA que corre en tu terminal. Entiende tu proyecto, puede leer y escribir ficheros, ejecutar comandos y mantener una conversación larga mientras construye lo que le pides.

A diferencia de Claude Code (que solo usa Claude de Anthropic), OpenCode soporta múltiples proveedores de IA y, lo más interesante, puede apuntar a **Workers AI via AI Gateway** — usando modelos como Llama 3.1 sin coste adicional.

> Repositorio: [github.com/sst/opencode](https://github.com/sst/opencode)

---

## Instalación

```bash
# Con npm (recomendado — siempre la última versión)
npm install -g opencode-ai

# Verificar instalación
opencode --version
```

Para usar OpenCode necesitas al menos un proveedor configurado. Las opciones más comunes:

```bash
# Opción A: Usar Anthropic (Claude)
export ANTHROPIC_API_KEY="sk-ant-..."

# Opción B: Usar OpenAI
export OPENAI_API_KEY="sk-..."

# Opción C: Usar Workers AI via AI Gateway (gratis dentro de límites)
# Ver configuración más abajo
```

---

## Configurar Workers AI como proveedor

Esta es la integración más interesante para proyectos de Cloudflare: usas modelos de Workers AI (Llama 3.1, Mistral, Qwen...) a través de tu AI Gateway, con observabilidad y caché incluidos.

### Paso 1: Crear un gateway

Si aún no tienes uno, ve a [dash.cloudflare.com](https://dash.cloudflare.com/) → **AI → AI Gateway → Create Gateway**.

### Paso 2: Configurar OpenCode

Crea o edita `~/.config/opencode/config.json`:

```json
{
  "provider": "openai",
  "model": "@cf/meta/llama-3.3-70b-instruct-fp8-fast",
  "baseURL": "https://gateway.ai.cloudflare.com/v1/{TU_ACCOUNT_ID}/{TU_GATEWAY_NAME}/workers-ai/v1",
  "apiKey": "TU_CLOUDFLARE_API_TOKEN"
}
```

> ℹ️ Workers AI expone una API compatible con OpenAI cuando se accede vía AI Gateway. Por eso `"provider": "openai"` aunque el modelo sea de Cloudflare.

### Modelos recomendados de Workers AI para coding

| Modelo | Para qué |
|--------|---------|
| `@cf/meta/llama-3.3-70b-instruct-fp8-fast` | Mejor balance calidad/velocidad para código |
| `@cf/meta/llama-3.1-8b-instruct` | Más rápido, tareas sencillas |
| `@cf/qwen/qwen2.5-coder-32b-instruct` | Especializado en código |
| `@cf/mistral/mistral-7b-instruct-v0.2` | Alternativa ligera |

Consulta el catálogo completo en [developers.cloudflare.com/workers-ai/models/](https://developers.cloudflare.com/workers-ai/models/).

---

## Instalar Cloudflare Skills

Los **Skills** son ficheros de conocimiento especializado que el agente lee antes de responder. Con los Skills de Cloudflare, OpenCode sabe exactamente cómo funciona Wrangler, D1, los Durable Objects, el Agents SDK...

```bash
# Instalar todos los skills de Cloudflare de golpe
opencode skills install cloudflare

# O desde el repositorio directamente (también funciona con Claude Code)
npx skills add https://github.com/cloudflare/skills
```

### Skills disponibles

| Skill | Cuándo se activa |
|-------|-----------------|
| `cloudflare` | Preguntas generales sobre la plataforma |
| `workers-best-practices` | Al escribir o revisar código de Workers |
| `wrangler` | Al usar la CLI o configurar `wrangler.jsonc` |
| `durable-objects` | Al diseñar sistemas con estado persistente |
| `agents-sdk` | Al construir agentes con el Agents SDK |
| `building-ai-agent` | Tutorial paso a paso para crear agentes |
| `building-mcp-server` | Al crear servidores MCP en Workers |
| `sandbox-sdk` | Al usar entornos de ejecución aislados |
| `web-perf` | Optimización de rendimiento en el edge |
| `find-skills` | Descubre qué skills están disponibles |

---

## Flujo de trabajo real — Vibe Coding con Cloudflare

### Arrancar un proyecto

```bash
# Crea el proyecto base con create-cloudflare
npm create cloudflare@latest mi-api
# → Elige "Hono" como framework

cd mi-api

# Abre OpenCode en el directorio del proyecto
opencode
```

OpenCode leerá automáticamente tu `wrangler.jsonc`, `package.json`, y los ficheros existentes para entender el contexto del proyecto.

### Ejemplo de sesión de vibe coding

```
Tú: Necesito un endpoint GET /api/productos que devuelva los productos
    de la tabla "productos" en D1, ordenados por nombre, con paginación
    de 20 en 20 usando los parámetros ?pagina=1

OpenCode: [Lee src/index.ts, wrangler.jsonc]
          [Escribe el endpoint con paginación]
          [Actualiza el tipo Env con el binding DB]
          Listo. He añadido el endpoint en src/index.ts con...

Tú: Ahora añade autenticación. El token va en el header Authorization: Bearer <token>.
    El valor válido del token está en la variable de entorno API_SECRET.
    Si no hay token o es incorrecto, devuelve 401.

OpenCode: [Modifica el endpoint añadiendo el middleware de auth]
          [Añade API_SECRET al tipo Env]
          He añadido la autenticación. También he creado un middleware
          reutilizable en src/middleware/auth.ts para que puedas usarlo
          en otros endpoints...

Tú: Crea la migración SQL para la tabla productos: id, nombre, descripcion,
    precio (decimal), stock (entero), activo (boolean), creado_en

OpenCode: [Crea migrations/0001_crear_productos.sql]
          Creada la migración. Para ejecutarla en local:
          npx wrangler d1 execute mi-db --local --file=migrations/0001_crear_productos.sql

Tú: Escribe tests con Vitest para el endpoint de productos

OpenCode: [Crea test/productos.test.ts con casos de éxito, paginación y auth]
```

### Comandos que OpenCode puede ejecutar por ti

```bash
# OpenCode puede proponer ejecutar estos comandos y tú los apruebas:
npx wrangler dev                          # Arrancar en local
npx wrangler d1 execute ... --local       # Ejecutar migraciones
npm test                                  # Ejecutar tests
npx wrangler deploy                       # Desplegar a producción
npx wrangler tail                         # Ver logs en tiempo real
```

---

## Casos de uso reales

### Caso 1: API REST completa en una sesión

```
"Crea una API REST con Hono y D1 para una tienda online simple.
Necesito gestión de productos y pedidos. Los productos tienen nombre,
precio y stock. Los pedidos tienen lista de productos con cantidad.
Al crear un pedido, reduce el stock. Añade validación con Zod y
tests con Vitest."
```

OpenCode generará en una sola sesión: esquema SQL, endpoints CRUD, validación, tests, y configuración de Wrangler.

### Caso 2: Migrar una app Express a Workers

```
"Tengo esta app Express en server.js [pega el código]. Necesito
migrarla a Cloudflare Workers con Hono. La base de datos MySQL
la reemplazamos por D1. Mantén la misma estructura de rutas."
```

### Caso 3: Añadir IA a una app existente

```
"Añade un endpoint POST /api/soporte que reciba una pregunta del usuario
y la responda usando Claude a través de mi AI Gateway. La URL del gateway
está en la variable de entorno AI_GATEWAY_URL y la API key de Anthropic
en ANTHROPIC_API_KEY."
```

---

## Comparativa: OpenCode vs Claude Code

| | OpenCode | Claude Code |
|---|----------|-------------|
| **Interfaz** | TUI en terminal (muy visual) | Terminal |
| **Open Source** | ✅ Sí | ❌ No |
| **Modelos** | Multi-proveedor | Solo Claude (Anthropic) |
| **Workers AI** | ✅ Via AI Gateway | ❌ No directamente |
| **Skills de Cloudflare** | ✅ Sí | ✅ Sí |
| **MCP** | ✅ Sí | ✅ Sí |
| **Precio** | Pagas tu propio uso de API | Suscripción Claude Max |
| **Contexto de proyecto** | Excelente | Excelente |
| **Mejor para** | Control total del modelo y coste | Máxima calidad con Claude |

> 💡 No son excluyentes. Muchos equipos usan OpenCode con Workers AI para tareas rutinarias y Claude Code para las más complejas.

---

## Recursos

- Documentación de OpenCode: [opencode.ai/docs](https://opencode.ai/docs)
- Repositorio: [github.com/sst/opencode](https://github.com/sst/opencode)
- Skills de Cloudflare: [github.com/cloudflare/skills](https://github.com/cloudflare/skills)
- Workers AI (modelos disponibles): [developers.cloudflare.com/workers-ai/models](https://developers.cloudflare.com/workers-ai/models/)
- AI Gateway: [ver guía completa](ai-gateway.md)

---

[← Volver al índice](../README.md)
