# SDKs y Librerías Cliente

SDKs oficiales de Cloudflare para interactuar con la API REST desde diferentes lenguajes.

---

## Repositorios

| Repositorio | Lenguaje | Descripción |
|-------------|----------|-------------|
| [cloudflare/cloudflare-typescript](https://github.com/cloudflare/cloudflare-typescript) | TypeScript / Node.js | SDK con tipos completos para toda la API REST — funciona en Node.js y Workers |
| [cloudflare/cloudflare-python](https://github.com/cloudflare/cloudflare-python) | Python | SDK oficial con soporte sync/async y validación Pydantic |
| [cloudflare/cloudflare-go](https://github.com/cloudflare/cloudflare-go) | Go | Librería Go con request/response tipados — base del proveedor Terraform |
| [cloudflare/workers-rs](https://github.com/cloudflare/workers-rs) | Rust | SDK Rust → WebAssembly para escribir Workers en Rust |

---

## TypeScript / Node.js

```bash
npm install cloudflare
```

```typescript
import Cloudflare from "cloudflare";

const cliente = new Cloudflare({
  apiToken: process.env.CLOUDFLARE_API_TOKEN,
});

// Listar zonas
const zonas = await cliente.zones.list();

// Crear un registro DNS
const registro = await cliente.dns.records.create({
  zone_id: "tu-zone-id",
  type: "A",
  name: "www",
  content: "192.0.2.1",
  proxied: true,
});

// Purgar caché
await cliente.cache.purge({
  zone_id: "tu-zone-id",
  purge_everything: true,
});
```

> Documentación: [cloudflare-typescript en npm](https://www.npmjs.com/package/cloudflare)

---

## Python

```bash
pip install cloudflare
```

```python
import cloudflare

cliente = cloudflare.Cloudflare(api_token="tu-api-token")

# Listar zonas
zonas = cliente.zones.list()
for zona in zonas:
    print(zona.name, zona.id)

# Crear registro DNS
registro = cliente.dns.records.create(
    zone_id="tu-zone-id",
    type="A",
    name="www",
    content="192.0.2.1",
    proxied=True,
)

# Uso asíncrono
import asyncio
import cloudflare

async def main():
    async with cloudflare.AsyncCloudflare(api_token="tu-api-token") as cliente:
        zonas = await cliente.zones.list()

asyncio.run(main())
```

> Documentación: [cloudflare en PyPI](https://pypi.org/project/cloudflare/)

---

## Go

```bash
go get github.com/cloudflare/cloudflare-go/v4
```

```go
package main

import (
    "context"
    "fmt"
    cloudflare "github.com/cloudflare/cloudflare-go/v4"
    "github.com/cloudflare/cloudflare-go/v4/option"
)

func main() {
    cliente := cloudflare.NewClient(
        option.WithAPIToken("tu-api-token"),
    )

    ctx := context.Background()

    // Listar zonas
    zonas, err := cliente.Zones.List(ctx, cloudflare.ZoneListParams{})
    if err != nil {
        panic(err)
    }

    for _, zona := range zonas.Result {
        fmt.Println(zona.Name, zona.ID)
    }
}
```

---

## Rust (Workers)

```toml
# Cargo.toml
[dependencies]
worker = "0.4"
```

```rust
use worker::*;

#[event(fetch)]
async fn main(req: Request, env: Env, _ctx: Context) -> Result<Response> {
    let router = Router::new();

    router
        .get("/", |_, _| Response::ok("Hola desde Rust!"))
        .get_async("/kv/:key", |_, ctx| async move {
            let key = ctx.param("key").unwrap();
            let store = ctx.kv("MI_KV")?;
            let valor = store.get(key).text().await?;
            Response::ok(valor.unwrap_or_default())
        })
        .run(req, env)
        .await
}
```

> Repositorio: [cloudflare/workers-rs](https://github.com/cloudflare/workers-rs)

---

[Volver al índice principal](../README.md)
