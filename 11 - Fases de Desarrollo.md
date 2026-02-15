---
tags: [blackdog-app, roadmap, phases]
created: 2026-02-15
updated: 2026-02-15
status: in-progress
---

# Fases de Desarrollo

## Resumen de fases

| Fase | Nombre | Estado |
|------|--------|--------|
| **0** | Setup e infraestructura | COMPLETADA |
| **1** | Auth + Catalogo | COMPLETADA |
| **2** | Carrito + Checkout + Pagos | EN PROGRESO |
| **3** | Pedidos + Tracking + Push | EN PROGRESO |
| **4** | Sucursales + Perfil | Pendiente |
| **5** | QA + Deploy | Pendiente |

---

## Fase 0 — Setup e Infraestructura — COMPLETADA

### Tareas

- [x] Crear repo Git: `blackdog-app` (Flutter)
- [x] Crear repo Git: `blackdog-api` (Express backend)
- [x] Setup proyecto Flutter con estructura de carpetas
- [x] Setup proyecto Express con TypeScript
- [x] Crear proyecto Supabase dedicado (BlackDog App)
- [x] Crear tablas en Supabase (schema completo — 13 tablas)
- [x] Configurar RLS policies en Supabase
- [ ] Setup Firebase project (FCM + Crashlytics) — pospuesto a Fase 3
- [ ] Configurar Supabase Auth providers (Google + Apple) — configurar en dashboard
- [x] Setup systemd service para el backend (puerto 3002)
- [x] Implementar sync job: productos de Odoo → Supabase (942 productos)
- [x] Implementar sync job: categorias (141 categorias)
- [x] Implementar sync job: stock por sucursal (14,334 registros)
- [x] Implementar sync job: sucursales/warehouses (17 tiendas)
- [x] CI/CD basico (deploy script `/opt/update-blackdog-api.sh`)

---

## Fase 1 — Auth + Catalogo — COMPLETADA

### Backend
- [x] Middleware de autenticacion (JWT via Supabase)
- [x] Endpoints de auth (register, complete-profile, profile CRUD)
- [x] Crear res.partner en Odoo al registrar usuario
- [x] Endpoints de productos (list, detail, search, featured)
- [x] Endpoints de categorias (arbol y lista plana)
- [x] Endpoint de sucursales

### Flutter
- [x] Splash screen + verificacion de sesion
- [x] Login screen (email + Google + Apple)
- [x] Register screen
- [x] Home screen (categorias, productos destacados, barra busqueda)
- [x] Catalogo (grid por categoria, ordenar por nombre/precio/fecha)
- [x] Detalle de producto (precio, descuento, stock por sucursal)
- [x] Busqueda de productos (debounce + full-text search)
- [x] Bottom navigation bar (4 tabs)
- [x] Theme con colores de marca
- [ ] Onboarding (3 slides) — pospuesto
- [ ] Forgot password screen — integrado como bottom sheet en login
- [ ] Imagenes de productos — pendiente (Odoo no tiene URLs publicas aun)

---

## Fase 2 — Carrito + Checkout + Pagos — EN PROGRESO

### Objetivo
Flujo de compra completo end-to-end.

### Backend
- [x] Endpoints de carrito (CRUD items)
- [x] Endpoint crear orden (validar stock, crear sale.order en Odoo)
- [x] Logica de pago en tienda (orden confirmed)
- [x] Calculo de delivery fee basico ($3.50 flat)
- [ ] Integracion Tilopay (crear payment link, webhook)
- [ ] Integracion Yappy (crear solicitud, webhook)
- [ ] Calculo de delivery fee por zona

### Flutter
- [ ] Provider de carrito (Riverpod)
- [ ] Pantalla de carrito (items, cantidades, totales)
- [ ] Checkout paso 1: metodo de entrega (delivery/pick-up)
- [ ] Seleccionar direccion o agregar nueva
- [ ] Seleccionar sucursal para pick-up (con validacion de stock)
- [ ] Checkout paso 2: metodo de pago
- [ ] WebView para Tilopay/Yappy
- [ ] Pantalla de confirmacion de orden
- [ ] Manejo de errores de pago

### Entregable
- Compra completa: agregar al carrito → checkout → pagar → orden creada en Odoo

---

## Fase 3 — Pedidos + Tracking + Notificaciones Push — EN PROGRESO

### Objetivo
Post-compra: ver pedidos, tracking, y recibir notificaciones.

### Backend — COMPLETADO
- [x] Endpoints de ordenes (list, detail) — hecho en Fase 2
- [x] Endpoint de tracking (timeline de estados) — incluido en order detail
- [x] Servicio de push notifications (Firebase Admin SDK, graceful fallback)
- [x] Enviar push en cada cambio de estado
- [x] Registro de FCM tokens (POST/DELETE /api/push-tokens)
- [x] Historial de notificaciones (GET, mark read, mark all read)
- [x] Mecanismo para actualizar estados (POST /api/admin/orders/:id/status con validacion de transiciones)

### Flutter
- [ ] Pantalla "Mis Pedidos" (tabs por estado)
- [ ] Detalle de pedido con timeline de tracking
- [ ] Integracion Firebase Messaging
- [ ] Solicitar permisos de notificacion
- [ ] Centro de notificaciones (historial)
- [ ] Deep links desde notificaciones → detalle de orden

---

## Fase 4 — Sucursales + Perfil — PENDIENTE

### Objetivo
Completar la experiencia con mapa de sucursales y gestion de perfil.

### Backend
- [ ] Endpoints de sucursales (list con distancia, detail)
- [ ] Endpoints de direcciones (CRUD)

### Flutter
- [ ] Mapa de sucursales (Google Maps)
- [ ] Lista de sucursales con horarios y estado
- [ ] Botones "Como llegar" y "Llamar"
- [ ] Pantalla Mi Perfil (editar nombre, telefono)
- [ ] Gestion de direcciones (CRUD)
- [ ] Configuracion de notificaciones (on/off)
- [ ] Pantalla "Sobre la app" (version, terminos, privacidad)

---

## Fase 5 — QA + Deploy — PENDIENTE

### Tareas
- [ ] Testing manual de todos los flujos
- [ ] Testing en dispositivos reales (iOS + Android)
- [ ] Performance testing (tiempos de carga, sync)
- [ ] Pruebas de pago end-to-end (Tilopay sandbox, Yappy sandbox)
- [ ] Fix de bugs encontrados
- [ ] Configurar Apple Developer account
- [ ] Configurar Google Play Console
- [ ] Preparar assets para stores (screenshots, descripcion, icono)
- [ ] Build de release iOS + Android
- [ ] Submit a App Store + Google Play
- [ ] Monitoreo post-launch (Crashlytics, logs)

---

## Post-MVP (Futuro)

| Feature | Descripcion |
|---------|-------------|
| Perfil de mascotas | Ver mascotas registradas (`x_mascota`), vacunas, historial |
| Agendar servicios | Grooming, veterinaria, desde la app |
| Programa de lealtad | Puntos por compra, canjear por descuentos (`loyalty.card`) |
| Recompras rapidas | "Reordenar" pedidos anteriores con un tap |
| Wishlists | Guardar productos favoritos |
| Reviews de productos | Calificar y comentar productos comprados |
| Chat de soporte | Chat en vivo con atencion al cliente |
| App para repartidores | Flutter app separada con GPS tracking |
