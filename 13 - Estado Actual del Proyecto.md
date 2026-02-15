---
tags: [blackdog-app, status, progress]
created: 2026-02-15
updated: 2026-02-15
status: in-progress
---

# Estado Actual del Proyecto

> Ultima actualizacion: 2026-02-15

## Resumen

Fase 0 y Fase 1 completadas. Fase 2 backend en progreso (carrito + ordenes listos, pagos pendientes). El backend API esta corriendo en produccion (puerto 3002) con sync de Odoo funcionando. La app Flutter tiene la estructura completa con pantallas de auth y catalogo.

---

## Fase 0 — COMPLETADA

### Backend API (`/opt/blackdog-api`)
- Express.js + TypeScript corriendo en puerto 3002
- Systemd service: `blackdog-api.service`
- Deploy script: `/opt/update-blackdog-api.sh`

### Conexion Odoo
- Odoo 18 Enterprise via XML-RPC
- DB: `dev-psdc-blackdogpanama-prod-3782039`
- Usuario: `soporte@blackdogpanama.com` (UID: 274)

### Sync Jobs (cron automatico)
| Job | Frecuencia | Registros |
|-----|------------|-----------|
| Productos | Cada 5 min | 942 productos retail |
| Categorias | Cada hora | 141 categorias |
| Stock | Cada 5 min | 14,334 registros |
| Sucursales | Diario 3am | 17 tiendas |

### Base de datos Supabase
- URL: `https://nhuixqohuoqjaijgthpf.supabase.co`
- 13 tablas creadas con RLS
- Datos sincronizandose correctamente

---

## Fase 1 — COMPLETADA

### Backend - Endpoints implementados

#### Publicos (sin auth)
| Endpoint | Descripcion | Estado |
|----------|-------------|--------|
| `GET /api/health` | Status Odoo + Supabase | OK |
| `GET /api/products` | Lista paginada, filtro por categoria, sort | OK (942 productos) |
| `GET /api/products/search?q=` | Full-text search espanol | OK |
| `GET /api/products/featured` | Productos con stock | OK |
| `GET /api/products/:id` | Detalle + stock por sucursal | OK |
| `GET /api/categories` | Arbol o lista plana | OK (141 categorias) |
| `GET /api/branches` | Todas las sucursales | OK (17 tiendas) |

#### Requieren auth (Bearer token Supabase)
| Endpoint | Descripcion | Estado |
|----------|-------------|--------|
| `POST /api/auth/register` | Signup + crea partner Odoo | OK |
| `POST /api/auth/complete-profile` | Post social login | OK |
| `GET /api/auth/profile` | Perfil + direcciones | OK |
| `PUT /api/auth/profile` | Update perfil (sync Odoo) | OK |

### Flutter App - Pantallas implementadas

| Pantalla | Archivo | Estado |
|----------|---------|--------|
| Splash | `screens/auth/splash_screen.dart` | OK |
| Login | `screens/auth/login_screen.dart` | OK (email + Google + Apple) |
| Register | `screens/auth/register_screen.dart` | OK |
| Home | `screens/home/home_screen.dart` | OK (featured + categories) |
| Catalogo | `screens/catalog/catalog_screen.dart` | OK (grid + filtros) |
| Detalle producto | `screens/catalog/product_detail_screen.dart` | OK (stock por sucursal) |
| Busqueda | `screens/catalog/search_screen.dart` | OK (debounce + full-text) |
| Carrito | `screens/cart/cart_screen.dart` | Placeholder (Fase 2) |
| Perfil | `screens/profile/profile_screen.dart` | Basico (Fase 4) |
| Bottom Nav | `screens/common/main_shell.dart` | OK (4 tabs) |

### Flutter - Arquitectura implementada
- **Riverpod** providers para auth, productos, categorias
- **GoRouter** con redirect por estado de auth
- **Dio** con interceptor JWT automatico
- **Tema** con colores de marca (#F7B104)
- **Modelos** con `fromJson()` para Product, Category, Branch

---

## Fase 2 — Carrito + Checkout + Pagos — EN PROGRESO

### Backend — Carrito y Ordenes (COMPLETADO)
- [x] `GET /api/cart` — Carrito activo del usuario (get-or-create, items + subtotal)
- [x] `POST /api/cart/items` — Agregar item (valida producto publicado + stock agregado)
- [x] `PUT /api/cart/items/:id` — Actualizar cantidad (valida ownership via cart join + stock)
- [x] `DELETE /api/cart/items/:id` — Eliminar item (valida ownership)
- [x] `DELETE /api/cart` — Limpiar carrito completo
- [x] `POST /api/orders` — Crear orden (valida stock en sucursal, crea sale.order en Odoo, tracking, convierte carrito)
- [x] `GET /api/orders` — Lista paginada con filtro por status
- [x] `GET /api/orders/:id` — Detalle con items + tracking timeline + branch info
- [x] `POST /api/orders/:id/cancel` — Cancelar orden (valida status, cancela en Odoo)
- [x] Logica pago en tienda (orden status=confirmed directamente)
- [x] Delivery fee basico ($3.50 para delivery, $0 para pickup)

### Backend — Pagos (pendiente)
- [ ] Integracion Tilopay (crear payment link, webhook)
- [ ] Integracion Yappy (crear solicitud, webhook)
- [ ] Calculo de delivery fee por zona (actualmente flat $3.50)

### Flutter (por hacer)
- [ ] Provider de carrito (add, remove, update quantity, clear)
- [ ] Pantalla carrito (items, cantidades, subtotal, total)
- [ ] Checkout paso 1: metodo entrega (delivery / pick-up)
- [ ] Seleccionar direccion o agregar nueva
- [ ] Seleccionar sucursal para pick-up (con stock)
- [ ] Checkout paso 2: metodo de pago
- [ ] WebView para Tilopay / Yappy
- [ ] Pantalla confirmacion de orden
- [ ] Manejo errores de pago

---

## Datos de las 17 sucursales

| Sucursal | ID Odoo |
|----------|---------|
| Albrook Fields | 1 |
| Bella Vista | 3 |
| Brisas del Golf | 4 |
| Brisas Norte | 6 |
| Calle 50 | 7 |
| Cantabria | 8 |
| Chiriqui David | 10 |
| Coco del Mar | 15 |
| Condado del Rey | 16 |
| Costa Verde | 17 |
| Los Pueblos | 18 |
| Ocean Mall | 21 |
| Parque Omar | 22 |
| Plaza Emporio | 23 |
| Santa Maria | 29 |
| Versalles | 30 |
| Villa Zaita | 31 |

---

## Archivos clave

### Backend (`blackdog-api`)
| Archivo | Funcion |
|---------|---------|
| `src/server.ts` | Entry point Express |
| `src/config/odoo.ts` | XML-RPC client para Odoo |
| `src/config/supabase.ts` | Clients Supabase (service role + anon) |
| `src/middleware/auth.ts` | JWT verification via Supabase |
| `src/routes/auth.routes.ts` | Auth endpoints |
| `src/routes/products.routes.ts` | Productos, categorias, sucursales |
| `src/routes/cart.routes.ts` | Cart CRUD (get, add, update, delete, clear) |
| `src/routes/orders.routes.ts` | Orders (create, list, detail, cancel) |
| `src/sync/scheduler.ts` | Cron jobs |
| `src/sync/products.sync.ts` | Sync productos Odoo → Supabase |
| `database/schema.sql` | Schema completo de Supabase |

### Flutter (`blackdog-app`)
| Archivo | Funcion |
|---------|---------|
| `lib/main.dart` | Entry point + Supabase init |
| `lib/config/env.dart` | URLs de API y Supabase |
| `lib/config/routes.dart` | GoRouter config |
| `lib/services/api_service.dart` | Dio HTTP client |
| `lib/providers/auth_provider.dart` | Auth state + actions |
| `lib/providers/products_provider.dart` | Products, search, categories |
| `lib/theme/app_theme.dart` | Brand theme completo |
