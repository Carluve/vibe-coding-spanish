<p align="center">
  <img src="https://www.cloudflare.com/img/logo-web-badges/cf-logo-on-white-bg.svg" alt="Cloudflare Logo" width="300">
</p>

<h1 align="center">☁️ Cloudflare en Español</h1>

<p align="center">
  <strong>El índice de recursos más completo para la comunidad hispanohablante de Cloudflare</strong><br>
  Repositorios oficiales · Guías prácticas · Ejemplos reales · Herramientas de IA
</p>

<p align="center">
  <a href="https://developers.cloudflare.com/">📚 Docs Oficiales</a> &nbsp;·&nbsp;
  <a href="https://github.com/cloudflare">🐙 GitHub Cloudflare</a> &nbsp;·&nbsp;
  <a href="https://community.cloudflare.com/">💬 Foro</a> &nbsp;·&nbsp;
  <a href="https://discord.cloudflare.com/">🎮 Discord</a> &nbsp;·&nbsp;
  <a href="https://blog.cloudflare.com/">✍️ Blog</a>
</p>

---

> ⚠️ **Aviso importante:** Este es un recurso comunitario, independiente y no oficial.
> La información puede contener errores o estar desactualizada. **Verifica siempre en la
> [documentación oficial](https://developers.cloudflare.com/) antes de usar en producción.**
> Los mantenedores no asumen ninguna responsabilidad por problemas derivados del uso de este contenido.

---

## ¿Qué encontrarás aquí?

Este repositorio nació con una idea simple: que cualquier desarrollador hispanohablante pueda entender y usar Cloudflare sin tener que navegar entre documentación en inglés dispersa por decenas de repositorios.

Aquí tienes todo organizado: desde añadir tu primer dominio hasta construir agentes de IA con estado persistente en el edge.

---

## 🗂️ Índice de contenidos

### La plataforma

| Sección | Qué aprenderás |
|---------|---------------|
| [🌐 Application Services](docs/application-services.md) | CDN, WAF, DDoS, DNS, SSL/TLS, Load Balancing, Bot Management |
| [🔒 SASE / Zero Trust](docs/zero-trust.md) | Cloudflare One, Access, Gateway, Tunnel, WARP, DLP |
| [⚡ Developer Platform](docs/developer-platform.md) | Workers, Pages, D1, R2, KV, Durable Objects, Queues, Workflows |
| [🤖 Agentic AI](docs/agentic-ai.md) | Workers AI, Agents SDK, Vectorize, MCP |
| [🔀 AI Gateway](docs/ai-gateway.md) | Proxy de inferencia, caché, fallbacks, observabilidad |
| [🏗️ Infraestructura como Código](docs/iac.md) | Terraform, cf-terraforming, automatización |
| [📦 SDKs y Librerías](docs/sdks.md) | TypeScript, Python, Go, Rust |
| [🚀 Templates y Starters](docs/templates.md) | Proyectos listos para arrancar |

### Flujos de trabajo con IA

| Sección | Qué aprenderás |
|---------|---------------|
| [🎵 Vibe Coding con OpenCode](docs/vibe-coding-opencode.md) | Desarrollo asistido por IA en terminal, Workers AI como backend |
| [💻 Integraciones con VS Code](docs/vscode-integraciones.md) | MCP, Skills, GitHub Copilot, Continue.dev, debugging |

### Comunidad y recursos

| Sección | Qué encontrarás |
|---------|----------------|
| [🇪🇸 Guías en Español](docs/guias-espanol.md) | Tutoriales paso a paso, desde cero hasta avanzado |
| [🌍 Recursos Comunitarios](docs/recursos-comunitarios.md) | Discord, foro, blog, Radar, eventos |
| [💰 Cloudflare for Startups](docs/startups.md) | Hasta $250,000 USD en créditos para startups en Iberia |

---

## ⚡ Arranca en 5 minutos

Elige tu punto de partida según lo que quieras construir:

**Un Worker serverless (TypeScript):**
```bash
npm create cloudflare@latest mi-worker
# Selecciona "Hello World Worker"
cd mi-worker && npm run dev
# → http://localhost:8787
```

**Una app full-stack con base de datos:**
```bash
npm create cloudflare@latest mi-app
# Selecciona "Hono" y activa D1
cd mi-app && npm run dev
```

**Un agente de IA conversacional:**
```bash
npx create-cloudflare@latest mi-agente --template cloudflare/agents-starter
cd mi-agente && npm run dev
# → http://localhost:5173
```

**Desarrollo con IA (Vibe Coding):**
```bash
npm install -g opencode-ai
cd mi-proyecto && opencode
# Describe lo que quieres construir en lenguaje natural
```

---

## 🧭 ¿Por dónde empezar?

```
¿Tienes un sitio web y quieres protegerlo?
  → Application Services: CDN + WAF + DDoS

¿Quieres reemplazar tu VPN corporativa?
  → SASE / Zero Trust: Cloudflare One + Tunnel

¿Quieres construir una API o app serverless?
  → Developer Platform: Workers + D1 + R2

¿Quieres añadir IA a tu aplicación?
  → Agentic AI + AI Gateway + Workers AI

¿Quieres hacer todo lo anterior con IA?
  → Vibe Coding con OpenCode o Claude Code
```

---

## 🤝 Contribuir

¿Encontraste un error? ¿Tienes una guía que añadir? Lee la [Guía de Contribución](CONTRIBUTING.md).

Las contribuciones más valiosas son:
- Correcciones de información incorrecta o desactualizada
- Guías prácticas basadas en experiencia real
- Ejemplos de código que realmente funcionen

---

<p align="center">
  Hecho con ❤️ por la comunidad hispanohablante de Cloudflare<br>
  <sub>No oficial · No afiliado con Cloudflare, Inc. · Los mantenedores no asumen responsabilidad por errores u omisiones</sub>
</p>
