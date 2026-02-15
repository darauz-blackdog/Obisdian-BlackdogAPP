---
tags: [blackdog-app, roadmap, phases]
created: 2026-02-15
status: planning
---

# Fases de Desarrollo

## Resumen de fases

| Fase | Nombre | Dependencias |
|------|--------|-------------|
| **0** | Setup e infraestructura | Ninguna |
| **1** | Auth + Catálogo | Fase 0 |
| **2** | Carrito + Checkout + Pagos | Fase 1 |
| **3** | Pedidos + Tracking + Push | Fase 2 |
| **4** | Sucursales + Perfil | Fase 1 |
| **5** | QA + Deploy | Fases 1-4 |

---

## Fase 0 — Setup e Infraestructura

### Objetivo
Tener todos los repositorios, servicios y ambientes listos para empezar a desarrollar.

### Tareas

- [ ] Crear repo Git: `blackdog-app` (Flutter)
- [ ] Crear repo Git: `blackdog-api` (Express backend)
- [ ] Setup proyecto Flutter con estructura de carpetas
- [ ] Setup proyecto Express con TypeScript
- [ ] Crear proyecto Supabase dedicado (BlackDog App)
- [ ] Crear tablas en Supabase (schema completo)
- [ ] Configurar RLS policies en Supabase
- [ ] Setup Firebase project (FCM + Crashlytics)
- [ ] Configurar Supabase Auth (email + Google + Apple providers)
- [ ] Setup systemd service para el backend (puerto 3002)
- [ ] Implementar sync job básico: productos de Odoo → Supabase
- [ ] Implementar sync job: categorías
- [ ] Implementar sync job: stock por sucursal
- [ ] Implementar sync job: sucursales/warehouses
- [ ] Seed de datos de sucursales (coordenadas, horarios, teléfonos)
- [ ] CI/CD básico (deploy scripts como PeopleBD)

### Entregable
- Backend corriendo en puerto 3002
- Supabase con datos sincronizados de Odoo
- Flutter app compilando y mostrando "Hello World"

---

## Fase 1 — Auth + Catálogo

### Objetivo
Los usuarios pueden registrarse, iniciar sesión, navegar productos y buscar.

### Backend
- [ ] Endpoints de auth (register, login, social, forgot-password)
- [ ] Crear res.partner en Odoo al registrar usuario
- [ ] Endpoints de productos (list, detail, search, featured)
- [ ] Endpoints de categorías
- [ ] Middleware de autenticación (JWT)

### Flutter
- [ ] Splash screen + verificación de sesión
- [ ] Onboarding (3 slides)
- [ ] Login screen (email + Google + Apple)
- [ ] Register screen
- [ ] Forgot password screen
- [ ] Home screen (banners, categorías, productos destacados)
- [ ] Catálogo (lista por categoría, ordenar, infinite scroll)
- [ ] Detalle de producto (imágenes, precio, descripción, stock)
- [ ] Búsqueda de productos (full-text search)
- [ ] Bottom navigation bar

### Entregable
- Usuario se registra, ve productos, busca, navega categorías
- Datos vienen de Odoo vía sync → Supabase → API → Flutter

---

## Fase 2 — Carrito + Checkout + Pagos

### Objetivo
Flujo de compra completo end-to-end.

### Backend
- [ ] Endpoints de carrito (CRUD items)
- [ ] Endpoint crear orden (validar stock, crear sale.order en Odoo)
- [ ] Integración Tilopay (crear payment link, webhook)
- [ ] Integración Yappy (crear solicitud, webhook)
- [ ] Lógica de pago en tienda (orden draft)
- [ ] Cálculo de delivery fee por zona

### Flutter
- [ ] Pantalla de carrito (items, cantidades, totales)
- [ ] Checkout paso 1: método de entrega (delivery/pick-up)
- [ ] Seleccionar dirección o agregar nueva
- [ ] Seleccionar sucursal para pick-up (con validación de stock)
- [ ] Checkout paso 2: método de pago
- [ ] WebView para Tilopay/Yappy
- [ ] Pantalla de confirmación de orden
- [ ] Manejo de errores de pago

### Entregable
- Compra completa: agregar al carrito → checkout → pagar → orden creada en Odoo

---

## Fase 3 — Pedidos + Tracking + Notificaciones Push

### Objetivo
Post-compra: ver pedidos, tracking, y recibir notificaciones.

### Backend
- [ ] Endpoints de órdenes (list, detail)
- [ ] Endpoint de tracking (timeline de estados)
- [ ] Servicio de push notifications (Firebase)
- [ ] Enviar push en cada cambio de estado
- [ ] Registro de FCM tokens
- [ ] Tabla de notificaciones (historial)
- [ ] Mecanismo para actualizar estados (desde Odoo o panel)

### Flutter
- [ ] Pantalla "Mis Pedidos" (tabs por estado)
- [ ] Detalle de pedido con timeline de tracking
- [ ] Integración Firebase Messaging
- [ ] Solicitar permisos de notificación
- [ ] Centro de notificaciones (historial)
- [ ] Deep links desde notificaciones → detalle de orden

### Entregable
- Cliente ve sus pedidos, tracking en tiempo real, recibe push notifications

---

## Fase 4 — Sucursales + Perfil

### Objetivo
Completar la experiencia con mapa de sucursales y gestión de perfil.

### Backend
- [ ] Endpoints de sucursales (list con distancia, detail)
- [ ] Endpoints de perfil (get, update)
- [ ] Endpoints de direcciones (CRUD)

### Flutter
- [ ] Mapa de sucursales (Google Maps)
- [ ] Lista de sucursales con horarios y estado
- [ ] Botones "Cómo llegar" y "Llamar"
- [ ] Pantalla Mi Perfil (editar nombre, teléfono)
- [ ] Gestión de direcciones (CRUD)
- [ ] Configuración de notificaciones (on/off)
- [ ] Pantalla "Sobre la app" (versión, términos, privacidad)
- [ ] Cerrar sesión

### Entregable
- App completa con todas las funcionalidades del MVP

---

## Fase 5 — QA + Deploy

### Objetivo
Calidad, testing y publicación en stores.

### Tareas
- [ ] Testing manual de todos los flujos
- [ ] Testing en dispositivos reales (iOS + Android)
- [ ] Performance testing (tiempos de carga, sync)
- [ ] Pruebas de pago end-to-end (Tilopay sandbox, Yappy sandbox)
- [ ] Fix de bugs encontrados
- [ ] Configurar Apple Developer account
- [ ] Configurar Google Play Console
- [ ] Preparar assets para stores (screenshots, descripción, ícono)
- [ ] Build de release iOS + Android
- [ ] Submit a App Store review
- [ ] Submit a Google Play review
- [ ] Monitoreo post-launch (Crashlytics, logs)

### Entregable
- App publicada y disponible en App Store y Google Play

---

## Fase 2+ (Post-MVP / Futuro)

| Feature | Descripción |
|---------|-------------|
| Perfil de mascotas | Ver mascotas registradas (`x_mascota`), vacunas, historial |
| Agendar servicios | Grooming, veterinaria, desde la app |
| Programa de lealtad | Puntos por compra, canjear por descuentos (`loyalty.card`) |
| Comunidad | Feed social, historias, fotos de mascotas |
| App para repartidores | Flutter app separada con GPS tracking |
| Tracking en vivo | Mapa en tiempo real del repartidor |
| Recompras rápidas | "Reordenar" pedidos anteriores con un tap |
| Wishlists | Guardar productos favoritos |
| Reviews de productos | Calificar y comentar productos comprados |
| Chat de soporte | Chat en vivo con atención al cliente |
