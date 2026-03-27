# Application Services

Servicios de rendimiento, seguridad y fiabilidad para aplicaciones web. Cloudflare actúa como proxy inverso entre tus usuarios y tu servidor de origen, proporcionando caché, protección DDoS, WAF y optimización — sin cambiar tu hosting.

---

## Repositorios oficiales

| Repositorio | Descripción | Lenguaje |
|-------------|-------------|----------|
| [cloudflare/cloudflare-docs](https://github.com/cloudflare/cloudflare-docs) | Documentación oficial completa en código abierto (CC BY 4.0) | MDX |
| [cloudflare/quiche](https://github.com/cloudflare/quiche) | Implementación del protocolo QUIC y HTTP/3 | Rust |
| [cloudflare/cloudflare-go](https://github.com/cloudflare/cloudflare-go) | Librería Go oficial para la API de Cloudflare | Go |
| [cloudflare/terraform-provider-cloudflare](https://github.com/cloudflare/terraform-provider-cloudflare) | Proveedor oficial de Terraform para Cloudflare | Go |
| [cloudflare/cf-terraforming](https://github.com/cloudflare/cf-terraforming) | CLI para importar recursos existentes a Terraform | Go |

---

## Documentación por producto

| Producto | Documentación | Descripción |
|----------|--------------|-------------|
| CDN / Caché | [developers.cloudflare.com/cache](https://developers.cloudflare.com/cache/) | Cache Rules, Polish, Tiered Cache |
| WAF | [developers.cloudflare.com/waf](https://developers.cloudflare.com/waf/) | Firewall de aplicaciones, reglas personalizadas, rate limiting |
| DDoS | [developers.cloudflare.com/ddos-protection](https://developers.cloudflare.com/ddos-protection/) | Protección DDoS ilimitada y gratuita en todos los planes |
| DNS | [developers.cloudflare.com/dns](https://developers.cloudflare.com/dns/) | DNS autoritativo, DNSSEC, DNS sobre HTTPS |
| SSL/TLS | [developers.cloudflare.com/ssl](https://developers.cloudflare.com/ssl/) | Certificados, modos de cifrado, SSL for SaaS |
| Load Balancing | [developers.cloudflare.com/load-balancing](https://developers.cloudflare.com/load-balancing/) | Balanceo global de carga con health checks |
| Bot Management | [developers.cloudflare.com/bots](https://developers.cloudflare.com/bots/) | Detección y gestión de bots, puntuación de bot |
| Speed | [developers.cloudflare.com/speed](https://developers.cloudflare.com/speed/) | Speed Brain, Observatory, Early Hints |
| Images | [developers.cloudflare.com/images](https://developers.cloudflare.com/images/) | Optimización, transformación y entrega de imágenes |
| Stream | [developers.cloudflare.com/stream](https://developers.cloudflare.com/stream/) | Plataforma de vídeo con encoding y entrega adaptativa |

---

## Guía rápida: Añadir un dominio a Cloudflare

1. Crea una cuenta en [dash.cloudflare.com/sign-up](https://dash.cloudflare.com/sign-up)
2. Haz clic en **Add a site** e introduce tu dominio
3. Cloudflare escaneará tus registros DNS automáticamente
4. Selecciona el plan (Free, Pro, Business o Enterprise)
5. Actualiza los **nameservers** en tu registrador con los que Cloudflare te asigne
6. Espera la propagación (normalmente minutos, máximo 48h)
7. Tu dominio aparecerá como **Active** en el dashboard

> Documentación completa: [Primeros Pasos](https://developers.cloudflare.com/fundamentals/setup/account-setup/)

---

## Guía rápida: Configurar Cache Rules

Las Cache Rules permiten controlar exactamente qué se cachea y por cuánto tiempo.

```
Regla de ejemplo:
  Si: URL contiene /api/
  Entonces: Bypass cache (no cachear respuestas de API)

  Si: URL termina en .jpg, .png, .webp
  Entonces: Cache TTL = 1 año
```

> Documentación: [Cache Rules](https://developers.cloudflare.com/cache/how-to/cache-rules/)

---

[Volver al índice principal](../README.md)
