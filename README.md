<p align="center">
  <img src="https://www.cloudflare.com/img/logo-web-badges/cf-logo-on-white-bg.svg" alt="Cloudflare Logo" width="280">
</p>

<h1 align="center">Cloudflare España — Índice de Recursos</h1>

<p align="center">
  Repositorios oficiales · Documentación · Guías en Español<br>
  <em>Todo lo que necesitas para construir, proteger y escalar en Cloudflare</em>
</p>

<p align="center">
  <a href="https://github.com/cloudflare">GitHub Oficial</a> ·
  <a href="https://developers.cloudflare.com/">Documentación</a> ·
  <a href="https://community.cloudflare.com/">Foro</a> ·
  <a href="https://discord.cloudflare.com/">Discord</a> ·
  <a href="https://blog.cloudflare.com/">Blog</a>
</p>

> **Aviso importante:** Este es un recurso comunitario, no oficial. La información puede estar desactualizada, ser incorrecta o incompleta. Verifica siempre en la [documentación oficial de Cloudflare](https://developers.cloudflare.com/) antes de usarla en producción. Los mantenedores de este repositorio no nos hacemos responsables de ningún problema derivado del uso de esta información.

---

## Sobre este repositorio

Índice comunitario de recursos para desarrolladores e ingenieros hispanohablantes que trabajan con Cloudflare. Encontrarás enlaces directos a repositorios oficiales, documentación clave y guías prácticas en español.

La plataforma de Cloudflare se organiza en cuatro grandes áreas:

| Área | Qué cubre |
|------|-----------|
| [**Application Services**](docs/application-services.md) | CDN, WAF, DDoS, Bot Management, DNS, Load Balancing, SSL/TLS |
| [**SASE / Zero Trust**](docs/zero-trust.md) | Cloudflare One, WARP, Gateway, Access, Tunnel, DLP, CASB |
| [**Developer Platform**](docs/developer-platform.md) | Workers, Pages, D1, R2, KV, Durable Objects, Queues, Workflows |
| [**Agentic AI**](docs/agentic-ai.md) | Workers AI, Agents SDK, AI Gateway, Vectorize, MCP |

---

## Índice completo

**Plataforma**
- [Application Services](docs/application-services.md) — Rendimiento, seguridad y fiabilidad web
- [SASE / Zero Trust](docs/zero-trust.md) — Acceso seguro sin VPN
- [Developer Platform](docs/developer-platform.md) — Serverless y edge computing
- [Agentic AI](docs/agentic-ai.md) — IA en el edge con agentes y MCP
- [AI Gateway](docs/ai-gateway.md) — Proxy de inferencia con caché, logs y fallbacks
- [Infraestructura como Código](docs/iac.md) — Terraform, cf-terraforming
- [SDKs y Librerías](docs/sdks.md) — TypeScript, Python, Go, Rust
- [Templates y Starters](docs/templates.md) — Proyectos de inicio oficiales

**Herramientas y flujos de trabajo**
- [Vibe Coding con OpenCode](docs/vibe-coding-opencode.md) — Desarrollo asistido por IA en terminal
- [Integraciones con VS Code](docs/vscode-integraciones.md) — MCP, Skills, extensiones y debugging

**Recursos en español**
- [Guías en Español](docs/guias-espanol.md) — Documentación y tutoriales por nivel
- [Recursos Comunitarios](docs/recursos-comunitarios.md) — Foro, Discord, Blog, Radar
- [Cloudflare for Startups](docs/startups.md) — Hasta $250,000 USD en créditos para Iberia

---

## Inicio rápido

**Developer Platform (Workers):**
```bash
npm create cloudflare@latest mi-proyecto
cd mi-proyecto
npm run dev
```

**Agente IA:**
```bash
npx create-cloudflare@latest --template cloudflare/agents-starter
cd agents-starter
npm run dev
```

**Vibe Coding con OpenCode:**
```bash
npm install -g opencode-ai
cd mi-proyecto
opencode
```

**Zero Trust (Tunnel):**
```bash
brew install cloudflared
cloudflared tunnel create mi-tunnel
cloudflared tunnel run mi-tunnel
```

---

## Contribuir

Lee la [Guía de Contribución](CONTRIBUTING.md) para añadir recursos, corregir errores o mejorar guías.

---

<p align="center">
  <strong>Recurso comunitario — no afiliado oficialmente con Cloudflare, Inc.</strong><br>
  <em>Los mantenedores no asumen responsabilidad por errores, omisiones ni daños derivados del uso de esta información.</em>
</p>
