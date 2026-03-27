# Infraestructura como Código (IaC)

Gestiona toda tu infraestructura de Cloudflare con código, control de versiones y pipelines de CI/CD.

---

## Repositorios oficiales

| Repositorio | Descripción | Lenguaje |
|-------------|-------------|----------|
| [cloudflare/terraform-provider-cloudflare](https://github.com/cloudflare/terraform-provider-cloudflare) | Proveedor oficial de Terraform — más de 2,500 endpoints | Go |
| [cloudflare/cf-terraforming](https://github.com/cloudflare/cf-terraforming) | Genera HCL e importa estado desde tu cuenta existente | Go |
| [cloudflare/cloudflare-go](https://github.com/cloudflare/cloudflare-go) | Cliente Go para la API — base del proveedor Terraform | Go |

---

## Documentación

| Recurso | Enlace |
|---------|--------|
| Proveedor Terraform | [registry.terraform.io/providers/cloudflare/cloudflare](https://registry.terraform.io/providers/cloudflare/cloudflare/latest/docs) |
| Gestión de Workers con Terraform | [developers.cloudflare.com/workers/wrangler/terraform](https://developers.cloudflare.com/workers/wrangler/terraform/) |
| Zero Trust con Terraform | [developers.cloudflare.com/cloudflare-one/api-terraform](https://developers.cloudflare.com/cloudflare-one/api-terraform/) |
| Tunnels con Terraform | [developers.cloudflare.com/cloudflare-one/connections/connect-networks/deployment-guides/terraform](https://developers.cloudflare.com/cloudflare-one/connections/connect-networks/deployment-guides/terraform/) |

---

## Guía rápida: Terraform con Cloudflare

### Configuración inicial

```hcl
terraform {
  required_providers {
    cloudflare = {
      source  = "cloudflare/cloudflare"
      version = "~> 4.0"
    }
  }
}

provider "cloudflare" {
  api_token = var.cloudflare_api_token
}
```

### Variables recomendadas

```hcl
variable "cloudflare_api_token" {
  description = "Token de API de Cloudflare"
  type        = string
  sensitive   = true
}

variable "zone_id" {
  description = "ID de la zona DNS"
  type        = string
}
```

### Ejemplos de recursos

**Registro DNS:**
```hcl
resource "cloudflare_record" "www" {
  zone_id = var.zone_id
  name    = "www"
  value   = "192.0.2.1"
  type    = "A"
  proxied = true
}
```

**Worker Script:**
```hcl
resource "cloudflare_workers_script" "mi_worker" {
  account_id = var.account_id
  name       = "mi-worker"
  content    = file("worker.js")
}
```

**Cloudflare Tunnel:**
```hcl
resource "cloudflare_tunnel" "mi_tunnel" {
  account_id = var.account_id
  name       = "mi-tunnel"
  secret     = base64encode(random_password.tunnel_secret.result)
}

resource "cloudflare_tunnel_config" "mi_tunnel_config" {
  account_id = var.account_id
  tunnel_id  = cloudflare_tunnel.mi_tunnel.id

  config {
    ingress_rule {
      hostname = "mi-app.midominio.com"
      service  = "http://localhost:3000"
    }
    ingress_rule {
      service = "http_status:404"
    }
  }
}
```

---

## cf-terraforming: Importar recursos existentes

Si ya tienes recursos configurados manualmente en Cloudflare, `cf-terraforming` genera el código HCL automáticamente:

```bash
# Instalar
go install github.com/cloudflare/cf-terraforming/cmd/cf-terraforming@latest

# Generar HCL para todos los registros DNS de una zona
cf-terraforming generate \
  --token $CF_API_TOKEN \
  --zone $ZONE_ID \
  --resource-type cloudflare_record

# Importar estado de Terraform
cf-terraforming import \
  --token $CF_API_TOKEN \
  --zone $ZONE_ID \
  --resource-type cloudflare_record
```

---

[Volver al índice principal](../README.md)
