# SASE / Zero Trust

Cloudflare One es la plataforma SASE (Secure Access Service Edge) que reemplaza VPNs tradicionales y dispositivos de seguridad heredados. Conecta usuarios, dispositivos y aplicaciones de forma segura sin exponer tu red.

---

## Repositorios oficiales

| Repositorio | Descripción | Lenguaje |
|-------------|-------------|----------|
| [cloudflare/cloudflared](https://github.com/cloudflare/cloudflared) | Cliente de Cloudflare Tunnel — conecta tu infraestructura sin abrir puertos | Go |
| [cloudflare/terraform-provider-cloudflare](https://github.com/cloudflare/terraform-provider-cloudflare) | Recursos Terraform para Access, Gateway, Tunnel, DLP, etc. | Go |
| [cloudflare/cf-terraforming](https://github.com/cloudflare/cf-terraforming) | Importa configuración Zero Trust existente a Terraform | Go |

---

## Documentación por producto

| Producto | Documentación | Descripción |
|----------|--------------|-------------|
| Cloudflare One | [developers.cloudflare.com/cloudflare-one](https://developers.cloudflare.com/cloudflare-one/) | Visión general de la plataforma SASE completa |
| Access | [/cloudflare-one/policies/access](https://developers.cloudflare.com/cloudflare-one/policies/access/) | Control de acceso sin VPN con integración de IdP |
| Gateway | [/cloudflare-one/policies/gateway](https://developers.cloudflare.com/cloudflare-one/policies/gateway/) | Secure Web Gateway — filtrado DNS, HTTP y red |
| Tunnel | [/cloudflare-one/connections/connect-networks](https://developers.cloudflare.com/cloudflare-one/connections/connect-networks/) | Túneles seguros sin exponer IPs de origen |
| WARP Client | [/cloudflare-one/connections/connect-devices/warp](https://developers.cloudflare.com/cloudflare-one/connections/connect-devices/warp/) | Cliente de dispositivo para acceso seguro |
| DLP | [/cloudflare-one/policies/data-loss-prevention](https://developers.cloudflare.com/cloudflare-one/policies/data-loss-prevention/) | Prevención de pérdida de datos en tránsito |
| CASB | [/cloudflare-one/applications/scan-apps](https://developers.cloudflare.com/cloudflare-one/applications/scan-apps/) | Cloud Access Security Broker para aplicaciones SaaS |
| DEX | [/cloudflare-one/insights/dex](https://developers.cloudflare.com/cloudflare-one/insights/dex/) | Digital Experience Monitoring — visibilidad end-to-end |
| API / Terraform | [/cloudflare-one/api-terraform](https://developers.cloudflare.com/cloudflare-one/api-terraform/) | Gestión programática de Zero Trust |

---

## Guía rápida: Cloudflare Tunnel

Cloudflare Tunnel crea una conexión saliente desde tu servidor a la red de Cloudflare. No necesitas abrir puertos ni configurar firewalls.

### Instalación

```bash
# macOS
brew install cloudflared

# Linux (Debian/Ubuntu)
curl -L --output cloudflared.deb \
  https://github.com/cloudflare/cloudflared/releases/latest/download/cloudflared-linux-amd64.deb
sudo dpkg -i cloudflared.deb

# Windows — descarga desde:
# https://github.com/cloudflare/cloudflared/releases
```

### Crear y ejecutar un tunnel

```bash
# Autenticarse con Cloudflare
cloudflared tunnel login

# Crear el tunnel
cloudflared tunnel create mi-tunnel

# Asociar un dominio
cloudflared tunnel route dns mi-tunnel mi-app.midominio.com

# Ejecutar el tunnel
cloudflared tunnel run mi-tunnel
```

### Archivo de configuración (`~/.cloudflared/config.yml`)

```yaml
tunnel: <TUNNEL_ID>
credentials-file: /root/.cloudflared/<TUNNEL_ID>.json

ingress:
  - hostname: mi-app.midominio.com
    service: http://localhost:3000
  - service: http_status:404
```

> Guía completa con Terraform: [Deploy Tunnels with Terraform](https://developers.cloudflare.com/cloudflare-one/connections/connect-networks/deployment-guides/terraform/)

---

## Guía rápida: Zero Trust Access

Protege cualquier aplicación interna con autenticación sin VPN.

1. En el dashboard, ve a **Zero Trust → Access → Applications**
2. Haz clic en **Add an application**
3. Selecciona el tipo: Self-hosted, SaaS o Browser-rendered
4. Define las **políticas de acceso** (por email, grupo de IdP, país, etc.)
5. Configura el **Identity Provider** (Google, Okta, Azure AD, GitHub, etc.)
6. Los usuarios acceden a tu app a través de `<nombre>.cloudflareaccess.com`

> Documentación: [Zero Trust Access](https://developers.cloudflare.com/cloudflare-one/policies/access/)

---

[Volver al índice principal](../README.md)
