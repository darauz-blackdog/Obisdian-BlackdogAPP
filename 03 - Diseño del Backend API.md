---
tags: [blackdog-app, backend, api]
created: 2026-02-15
status: planning
---

# Diseño del Backend API

## Información General

| Campo | Valor |
|-------|-------|
| **Puerto** | 3002 |
| **Host temporal** | VPS de PeopleBD |
| **Dominio futuro** | `api.blackdogapp.com` (por definir) |
| **Servicio systemd** | `blackdog-api.service` |

## Endpoints

### Auth
```
POST /api/auth/register
  Body: { email, password, name, phone }
  → Crea usuario en Supabase Auth
  → Crea res.partner en Odoo
  → Retorna: { user, session }

POST /api/auth/login
  Body: { email, password }
  → Supabase Auth signInWithPassword
  → Retorna: { user, session }

POST /api/auth/social
  Body: { provider: "google" | "apple", token }
  → Supabase Auth signInWithIdToken
  → Crea/vincula res.partner en Odoo si es nuevo
  → Retorna: { user, session }

POST /api/auth/forgot-password
  Body: { email }
  → Supabase Auth resetPasswordForEmail

POST /api/auth/refresh
  Body: { refresh_token }
  → Retorna nuevo access_token
```

### Productos
```
GET /api/products
  Query: ?category_id=6&page=1&limit=20&sort=newest
  → Lee de Supabase (cache)
  → Retorna: { products[], total, page, pages }

GET /api/products/:id
  → Producto + stock por sucursal
  → Retorna: { product, stock_by_branch[] }

GET /api/products/search
  Query: ?q=royal+canin&category_id=6
  → Full-text search en Supabase
  → Retorna: { products[], total }

GET /api/products/featured
  → Productos destacados (configurados en Odoo o manual)
  → Retorna: { products[] }

GET /api/categories
  → Árbol de categorías (cache)
  → Retorna: { categories[] } (nested tree)
```

### Carrito
```
GET /api/cart
  Auth: Bearer token
  → Carrito activo del usuario
  → Retorna: { cart, items[], subtotal }

POST /api/cart/items
  Auth: Bearer token
  Body: { product_id, quantity }
  → Agrega o actualiza item
  → Valida stock disponible
  → Retorna: { cart, items[] }

PUT /api/cart/items/:id
  Auth: Bearer token
  Body: { quantity }
  → Actualiza cantidad

DELETE /api/cart/items/:id
  Auth: Bearer token
  → Elimina item del carrito

DELETE /api/cart
  Auth: Bearer token
  → Vacía el carrito
```

### Checkout / Órdenes
```
POST /api/orders
  Auth: Bearer token
  Body: {
    delivery_type: "delivery" | "pickup",
    branch_id: 7,              // Sucursal para pickup o despacho
    address_id: "uuid",        // Solo si delivery
    payment_method: "tilopay" | "yappy" | "in_store",
    notes: "Dejar con el portero"
  }
  → Valida stock final
  → Crea sale.order en Odoo (estado draft)
  → Crea orden en Supabase
  → Inicia flujo de pago si aplica
  → Retorna: { order, payment_url? }

GET /api/orders
  Auth: Bearer token
  Query: ?status=confirmed&page=1
  → Mis órdenes
  → Retorna: { orders[], total }

GET /api/orders/:id
  Auth: Bearer token
  → Detalle completo + tracking
  → Retorna: { order, items[], tracking[] }

POST /api/orders/:id/cancel
  Auth: Bearer token
  → Cancela si está en estado cancelable
```

### Pagos
```
POST /api/payments/tilopay/create
  Auth: Bearer token
  Body: { order_id }
  → Crea payment link en Tilopay API
  → Retorna: { payment_url }

POST /api/payments/yappy/create
  Auth: Bearer token
  Body: { order_id }
  → Crea solicitud en Yappy API
  → Retorna: { yappy_url }

POST /api/webhooks/tilopay   (sin auth, validado por signature)
  → Tilopay notifica pago exitoso/fallido
  → Confirma sale.order en Odoo
  → Envía push notification al cliente

POST /api/webhooks/yappy     (sin auth, validado por signature)
  → Yappy notifica pago exitoso/fallido
  → Confirma sale.order en Odoo
  → Envía push notification al cliente
```

### Sucursales
```
GET /api/branches
  Query: ?lat=9.0&lng=-79.5
  → Sucursales ordenadas por distancia
  → Retorna: { branches[] } con distancia calculada

GET /api/branches/:id
  → Detalle de sucursal
  → Retorna: { branch, opening_hours }

GET /api/branches/:id/stock
  Query: ?product_ids=123,456
  → Stock de productos específicos en esa sucursal
```

### Perfil
```
GET /api/profile
  Auth: Bearer token
  → Datos del cliente
  → Retorna: { user, addresses[] }

PUT /api/profile
  Auth: Bearer token
  Body: { name, phone }
  → Actualiza en Supabase + Odoo (res.partner)

POST /api/profile/addresses
  Auth: Bearer token
  Body: { label, address_line, city, zone, lat, lng }

PUT /api/profile/addresses/:id
DELETE /api/profile/addresses/:id
```

### Notificaciones
```
POST /api/notifications/register
  Auth: Bearer token
  Body: { fcm_token, platform }
  → Registra token para push notifications

GET /api/notifications
  Auth: Bearer token
  → Historial de notificaciones
```

## Autenticación

Todas las rutas (excepto webhooks y productos públicos) requieren un **Bearer token** de Supabase Auth. El middleware verifica el JWT y extrae el `user_id`.

```typescript
// middleware/auth.ts
const authMiddleware = async (req, res, next) => {
  const token = req.headers.authorization?.replace('Bearer ', '');
  const { data: { user }, error } = await supabase.auth.getUser(token);
  if (error) return res.status(401).json({ error: 'Unauthorized' });
  req.user = user;
  next();
};
```

## Rate Limiting

| Ruta | Límite |
|------|--------|
| `/api/auth/*` | 5 req/min por IP |
| `/api/products/*` | 60 req/min por IP |
| `/api/orders/*` | 20 req/min por usuario |
| `/api/webhooks/*` | 100 req/min por IP |

## Variables de Entorno

```env
# Server
PORT=3002
NODE_ENV=production

# Supabase
SUPABASE_URL=https://xxx.supabase.co
SUPABASE_ANON_KEY=xxx
SUPABASE_SERVICE_ROLE_KEY=xxx

# Odoo
ODOO_URL=https://blackdog.odoo.com
ODOO_DB=blackdog
ODOO_USERNAME=api@blackdog.com
ODOO_PASSWORD=xxx

# Tilopay
TILOPAY_API_KEY=xxx
TILOPAY_API_SECRET=xxx
TILOPAY_WEBHOOK_SECRET=xxx

# Yappy
YAPPY_MERCHANT_ID=xxx
YAPPY_SECRET_KEY=xxx

# Firebase
FIREBASE_PROJECT_ID=xxx
FIREBASE_PRIVATE_KEY=xxx
FIREBASE_CLIENT_EMAIL=xxx

# App
APP_URL=https://api.blackdogapp.com
DELIVERY_FEE_DEFAULT=3.50
```
