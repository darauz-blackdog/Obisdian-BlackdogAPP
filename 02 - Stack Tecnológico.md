---
tags: [blackdog-app, tech-stack]
created: 2026-02-15
status: planning
---

# Stack Tecnológico

## App Móvil — Flutter

| Aspecto | Tecnología |
|---------|-----------|
| **Framework** | Flutter 3.x (Dart) |
| **State Management** | Riverpod 2.x |
| **HTTP Client** | Dio |
| **Local Storage** | SharedPreferences + Hive |
| **Navigation** | GoRouter |
| **Auth** | supabase_flutter |
| **Push Notifications** | firebase_messaging |
| **Pagos** | url_launcher (WebView para Tilopay/Yappy) |
| **Mapas** | google_maps_flutter |
| **Imágenes** | cached_network_image |
| **Animaciones** | flutter_animate |

### Estructura del proyecto Flutter
```
blackdog_app/
├── lib/
│   ├── main.dart
│   ├── app/
│   │   ├── router.dart
│   │   └── theme.dart
│   ├── core/
│   │   ├── api/            # Cliente HTTP, interceptors
│   │   ├── auth/           # Supabase Auth provider
│   │   ├── constants/      # Colores, strings, config
│   │   └── utils/          # Helpers, formatters
│   ├── features/
│   │   ├── auth/           # Login, Register, Social Auth
│   │   ├── home/           # Home screen, banners, featured
│   │   ├── catalog/        # Categorías, lista productos, búsqueda
│   │   ├── product/        # Detalle de producto
│   │   ├── cart/           # Carrito de compras
│   │   ├── checkout/       # Checkout, dirección, pago
│   │   ├── orders/         # Mis pedidos, detalle, tracking
│   │   ├── branches/       # Mapa de sucursales
│   │   ├── profile/        # Mi cuenta, direcciones
│   │   └── notifications/  # Centro de notificaciones
│   ├── models/             # Data classes (freezed)
│   ├── providers/          # Riverpod providers
│   └── widgets/            # Componentes compartidos
├── assets/
│   ├── images/
│   ├── icons/
│   └── fonts/
├── android/
├── ios/
└── pubspec.yaml
```

## Backend API — Express + TypeScript

| Aspecto | Tecnología |
|---------|-----------|
| **Runtime** | Node.js 20+ con tsx |
| **Framework** | Express.js 5 |
| **Lenguaje** | TypeScript |
| **ORM/DB** | Supabase JS Client |
| **Odoo Client** | xmlrpc (paquete npm) |
| **Auth Middleware** | Supabase Auth (JWT verification) |
| **Validación** | Zod |
| **Cron Jobs** | node-cron (sync Odoo → Supabase) |
| **Logging** | Pino |
| **Rate Limiting** | express-rate-limit |

### Estructura del backend
```
blackdog-api/
├── src/
│   ├── server.ts                # Entry point
│   ├── config/
│   │   ├── env.ts               # Variables de entorno
│   │   ├── supabase.ts          # Cliente Supabase
│   │   └── odoo.ts              # Cliente Odoo XML-RPC
│   ├── middleware/
│   │   ├── auth.ts              # JWT verification
│   │   ├── rate-limit.ts
│   │   └── error-handler.ts
│   ├── routes/
│   │   ├── auth.routes.ts
│   │   ├── products.routes.ts
│   │   ├── cart.routes.ts
│   │   ├── orders.routes.ts
│   │   ├── payments.routes.ts
│   │   ├── branches.routes.ts
│   │   ├── profile.routes.ts
│   │   └── webhooks.routes.ts
│   ├── services/
│   │   ├── odoo.service.ts      # CRUD contra Odoo
│   │   ├── product.service.ts
│   │   ├── order.service.ts
│   │   ├── payment.service.ts
│   │   ├── delivery.service.ts
│   │   └── notification.service.ts
│   ├── sync/
│   │   ├── scheduler.ts         # Cron jobs
│   │   ├── products.sync.ts     # Sync productos
│   │   ├── stock.sync.ts        # Sync inventario
│   │   └── categories.sync.ts   # Sync categorías
│   └── types/
│       └── index.ts
├── package.json
└── tsconfig.json
```

## Base de Datos — Supabase (Cache)

Proyecto Supabase dedicado para BlackDog App (separado de PeopleBD).

### Tablas principales

```sql
-- Productos (cache de Odoo)
products (
  id bigint PK,              -- Odoo product.template ID
  name text,
  description text,
  list_price decimal,
  sale_price decimal,         -- Precio con descuento
  category_id bigint FK,
  image_url text,
  product_type text,          -- consu, product
  is_published boolean,
  odoo_updated_at timestamptz,
  synced_at timestamptz
)

-- Categorías
categories (
  id bigint PK,              -- Odoo category ID
  name text,
  parent_id bigint FK,
  icon text,
  sort_order int
)

-- Stock por sucursal
stock_by_branch (
  product_id bigint FK,
  branch_id bigint FK,
  qty_available decimal,
  synced_at timestamptz,
  PRIMARY KEY (product_id, branch_id)
)

-- Sucursales
branches (
  id bigint PK,              -- Odoo warehouse ID
  name text,
  code text,
  address text,
  latitude decimal,
  longitude decimal,
  phone text,
  opening_hours jsonb,
  is_pickup_enabled boolean,
  is_delivery_enabled boolean
)

-- Carritos
carts (
  id uuid PK DEFAULT gen_random_uuid(),
  user_id uuid FK → auth.users,
  status text DEFAULT 'active',
  created_at timestamptz,
  updated_at timestamptz
)

cart_items (
  id uuid PK,
  cart_id uuid FK,
  product_id bigint,
  quantity int,
  unit_price decimal
)

-- Direcciones del cliente
addresses (
  id uuid PK,
  user_id uuid FK,
  label text,               -- "Casa", "Oficina"
  address_line text,
  city text,
  zone text,
  latitude decimal,
  longitude decimal,
  is_default boolean
)

-- Órdenes (referencia a Odoo)
orders (
  id uuid PK,
  user_id uuid FK,
  odoo_order_id bigint,      -- sale.order ID en Odoo
  odoo_order_name text,      -- "S00123"
  status text,               -- pending_payment, confirmed, preparing, shipping, ready_pickup, delivered
  delivery_type text,        -- delivery, pickup
  branch_id bigint,          -- Sucursal de pickup o despacho
  address_id uuid FK,
  payment_method text,       -- tilopay, yappy, in_store
  subtotal decimal,
  delivery_fee decimal,
  total decimal,
  created_at timestamptz,
  updated_at timestamptz
)

-- Tracking de pedidos
order_tracking (
  id uuid PK,
  order_id uuid FK,
  status text,
  message text,
  driver_name text,
  driver_phone text,
  created_at timestamptz
)

-- FCM tokens
push_tokens (
  id uuid PK,
  user_id uuid FK,
  token text UNIQUE,
  platform text,             -- ios, android
  created_at timestamptz
)
```

## Servicios Externos

| Servicio | Uso | Costo |
|----------|-----|-------|
| **Supabase** (Free/Pro) | Auth + DB cache | Free tier o $25/mes |
| **Firebase** (Free) | Push notifications + Crashlytics | Gratis |
| **Tilopay** | Pagos con tarjeta | Comisión por transacción |
| **Yappy** | Pagos móviles | Comisión por transacción |
| **Google Maps** | Mapa de sucursales | Free tier suficiente |
| **Apple Developer** | Publicar en App Store | $99/año |
| **Google Play** | Publicar en Play Store | $25 una vez |
