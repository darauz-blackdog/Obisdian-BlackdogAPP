---
tags: [blackdog-app, sync, odoo, supabase]
created: 2026-02-15
status: planning
---

# Sincronización Odoo → Supabase

## Por qué sincronizar

La app Flutter **no consulta Odoo directamente**. Lee de Supabase (PostgreSQL) que actúa como cache. Esto garantiza:

- Respuestas rápidas (<100ms)
- No sobrecargar Odoo con queries de miles de usuarios
- Disponibilidad aunque Odoo esté en mantenimiento
- Full-text search optimizado en PostgreSQL

## Jobs de sincronización

| Job | Frecuencia | Modelo Odoo | Tabla Supabase |
|-----|-----------|-------------|----------------|
| Productos | Cada 5 min | product.template + product.product | products |
| Categorías | Cada 1 hora | product.category | categories |
| Stock | Cada 5 min | stock.quant | stock_by_branch |
| Sucursales | Cada 24 horas | stock.warehouse + res.partner | branches |
| Imágenes | Cada 15 min | product.template.image_1920 | Supabase Storage |

## Flujo de sync de productos

```
1. Leer de Odoo: product.template donde:
   - website_published = true
   - type in ('consu', 'product')  (no servicios)
   - categ_id en categorías de retail

2. Para cada producto:
   - Si existe en Supabase → UPDATE
   - Si no existe → INSERT
   - Si existe en Supabase pero no en Odoo → SOFT DELETE (is_published = false)

3. Registrar timestamp de sync
```

```typescript
// sync/products.sync.ts
async function syncProducts() {
  const lastSync = await getLastSyncTimestamp('products');

  // Leer productos modificados desde última sync
  const products = await odoo.execute_kw(
    'product.template', 'search_read',
    [[
      ['website_published', '=', true],
      ['type', 'in', ['consu', 'product']],
      ['write_date', '>', lastSync],
    ]],
    {
      fields: [
        'name', 'list_price', 'categ_id', 'type',
        'default_code', 'website_description',
        'image_128', // thumbnail
      ],
      limit: 500,
    }
  );

  // Upsert en Supabase
  for (const product of products) {
    await supabase.from('products').upsert({
      id: product.id,
      name: product.name,
      list_price: product.list_price,
      category_id: product.categ_id[0],
      description: product.website_description,
      default_code: product.default_code,
      is_published: true,
      odoo_updated_at: product.write_date,
      synced_at: new Date().toISOString(),
    });
  }

  await setLastSyncTimestamp('products', new Date());
  console.log(`Synced ${products.length} products`);
}
```

## Flujo de sync de stock

```typescript
// sync/stock.sync.ts
async function syncStock() {
  // IDs de warehouses que son tiendas (excluir bodegas)
  const storeWarehouseIds = [1, 3, 4, 6, 7, 8, 10, 15, 16, 17, 18, 21, 22, 23, 29, 30, 31];

  // Leer stock locations de esas warehouses
  const quants = await odoo.execute_kw(
    'stock.quant', 'search_read',
    [[
      ['location_id.warehouse_id', 'in', storeWarehouseIds],
      ['quantity', '>', 0],
    ]],
    {
      fields: ['product_id', 'location_id', 'quantity'],
      limit: 10000,
    }
  );

  // Agrupar por producto + warehouse
  // Upsert en stock_by_branch
}
```

## Sync de imágenes

Las imágenes en Odoo son base64. Para la app necesitamos URLs públicas:

```
1. Leer image_1920 de product.template (base64)
2. Decodificar y subir a Supabase Storage (bucket: product-images)
3. Guardar la URL pública en products.image_url
4. Solo re-subir si el hash de la imagen cambió
```

## Monitoreo

El sync job debe loggear:
- Cantidad de registros sincronizados
- Errores de conexión a Odoo
- Tiempo de ejecución
- Productos con datos faltantes (sin imagen, sin precio)

Si el sync falla 3 veces seguidas → enviar alerta (email o Telegram).

## Tabla de control de sync

```sql
sync_logs (
  id uuid PK,
  job_name text,          -- 'products', 'stock', 'categories'
  status text,            -- 'success', 'error'
  records_synced int,
  duration_ms int,
  error_message text,
  started_at timestamptz,
  finished_at timestamptz
)
```
