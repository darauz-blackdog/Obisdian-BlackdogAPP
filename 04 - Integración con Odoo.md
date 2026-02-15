---
tags: [blackdog-app, odoo, integration]
created: 2026-02-15
status: planning
---

# Integración con Odoo

## Conexión

La app se conecta a Odoo via **XML-RPC** a través del backend Express. Las credenciales de Odoo **nunca** llegan al cliente móvil.

```typescript
// config/odoo.ts
import xmlrpc from 'xmlrpc';

const odooConfig = {
  url: process.env.ODOO_URL,
  db: process.env.ODOO_DB,
  username: process.env.ODOO_USERNAME,
  password: process.env.ODOO_PASSWORD,
};
```

## Modelos de Odoo utilizados

### Lectura (Sync → Supabase cache)

| Modelo Odoo | Campos | Uso en App |
|-------------|--------|-----------|
| `product.template` | name, list_price, image_1920, categ_id, type, default_code, website_published, website_description | Catálogo de productos |
| `product.product` | product_tmpl_id, lst_price, qty_available, default_code | Variantes y stock |
| `product.category` | name, parent_id, child_id | Árbol de categorías |
| `stock.quant` | product_id, location_id, quantity | Stock por sucursal |
| `stock.warehouse` | name, code, partner_id | Sucursales |
| `res.partner` (warehouse) | street, city, phone, email | Dirección de sucursal |

### Escritura (App → Odoo)

| Acción | Modelo Odoo | Operación |
|--------|------------|-----------|
| Registro de cliente | `res.partner` | create |
| Crear orden | `sale.order` | create |
| Líneas de orden | `sale.order.line` | create |
| Confirmar orden | `sale.order` | action_confirm |
| Registrar pago | `account.payment` | create |

## Crear un cliente en Odoo

Cuando un usuario se registra en la app:

```typescript
// services/odoo.service.ts
async createCustomer(data: {
  name: string;
  email: string;
  phone: string;
}): Promise<number> {
  const partnerId = await odoo.execute_kw(
    'res.partner', 'create', [{
      name: data.name,
      email: data.email,
      phone: data.phone,
      customer_rank: 1,
      // Tags o categoría para identificar clientes de la app
      category_id: [[6, 0, [APP_CUSTOMER_TAG_ID]]],
    }]
  );
  return partnerId;
}
```

## Crear una orden en Odoo

Cuando el cliente completa el checkout:

```typescript
async createSaleOrder(data: {
  partnerId: number;
  items: { productId: number; qty: number; price: number }[];
  warehouseId: number;
  notes?: string;
}): Promise<{ orderId: number; orderName: string }> {
  // 1. Crear la sale.order
  const orderId = await odoo.execute_kw(
    'sale.order', 'create', [{
      partner_id: data.partnerId,
      warehouse_id: data.warehouseId,
      note: data.notes,
      // Tag para identificar órdenes de la app
      origin: 'BlackDog App',
    }]
  );

  // 2. Crear las líneas
  for (const item of data.items) {
    await odoo.execute_kw(
      'sale.order.line', 'create', [{
        order_id: orderId,
        product_id: item.productId,
        product_uom_qty: item.qty,
        price_unit: item.price,
      }]
    );
  }

  // 3. Leer el nombre generado
  const [order] = await odoo.execute_kw(
    'sale.order', 'read', [[orderId], ['name']]
  );

  return { orderId, orderName: order.name };
}
```

## Confirmar orden tras pago exitoso

```typescript
async confirmOrder(orderId: number): Promise<void> {
  // Confirma la sale.order (draft → sale)
  await odoo.execute_kw(
    'sale.order', 'action_confirm', [[orderId]]
  );
}
```

## Mapeo de categorías para la App

No todas las categorías de Odoo se muestran en la app. Solo las de **retail**:

| Categoría Odoo | Categoría en App | Mostrar |
|---------------|-----------------|---------|
| Vendibles / Alimentos / Perro | Alimento Perro | Si |
| Vendibles / Alimentos / Gato | Alimento Gato | Si |
| Vendibles / Alimentos / Treats | Treats & Snacks | Si |
| Vendibles / Alimentos / Moz Pet | Moz Pet | Si |
| Vendibles / Accesorios / Perro | Accesorios | Si |
| Vendibles / Medicamentos | Salud | Parcial (solo OTC) |
| Vendibles / Servicios | - | No (fase 2) |
| Vendibles / Humano | Humano | Si |
| Gastos/* | - | No |

## Consideraciones importantes

1. **`website_published`**: Solo sincronizar productos marcados como publicados en Odoo
2. **`qty_available`**: Stock real disponible (no virtual/forecasted)
3. **Warehouse filtering**: Al consultar stock, filtrar por warehouses que son tiendas (excluir bodegas, consumo interno, descarte)
4. **Shopify coexistencia**: La app y Shopify comparten inventario. Las órdenes de ambos canales decrementan el mismo stock en Odoo
5. **Precios**: Usar `list_price` del product.template. Si hay pricelist especial para la app, configurar en Odoo
6. **Imágenes**: `image_1920` es base64 en Odoo. El sync job debe convertirlas a URLs (subir a Supabase Storage o CDN)
