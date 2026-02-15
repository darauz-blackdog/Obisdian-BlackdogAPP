---
tags: [blackdog-app, screens, ux]
created: 2026-02-15
status: planning
---

# Pantallas y Flujo de Usuario

## Mapa de navegación

```
                    ┌─────────────┐
                    │   Splash    │
                    └──────┬──────┘
                           │
                    ┌──────▼──────┐
                    │  Onboarding │ (primera vez)
                    └──────┬──────┘
                           │
              ┌────────────▼────────────┐
              │     Login / Register     │
              │  Email | Google | Apple  │
              └────────────┬────────────┘
                           │
                    ┌──────▼──────┐
                    │    HOME     │
                    └──────┬──────┘
                           │
         ┌─────────┬───────┼───────┬──────────┐
         │         │       │       │          │
    ┌────▼───┐ ┌───▼──┐ ┌──▼──┐ ┌──▼───┐ ┌───▼───┐
    │Catálogo│ │Buscar│ │Cart │ │Orders│ │Profile│
    └────┬───┘ └───┬──┘ └──┬──┘ └──┬───┘ └───┬───┘
         │         │       │       │          │
    ┌────▼───┐     │  ┌────▼────┐  │     ┌────▼────┐
    │Producto│     │  │Checkout │  │     │Addresses│
    │Detalle │     │  └────┬────┘  │     │Settings │
    └────────┘     │       │       │     └─────────┘
                   │  ┌────▼────┐  │
                   │  │ Pago    │  │
                   │  │WebView  │  │
                   │  └────┬────┘  │
                   │       │       │
                   │  ┌────▼────┐  │
                   │  │Confirm  │──┘
                   │  └─────────┘
                   │
              ┌────▼─────┐
              │Sucursales│
              │  (Mapa)  │
              └──────────┘
```

## Bottom Navigation

| Tab | Icono | Pantalla |
|-----|-------|----------|
| Home | `home` | Home Screen |
| Catálogo | `shopping_bag` | Categorías + Productos |
| Carrito | `shopping_cart` (badge) | Carrito de compras |
| Pedidos | `receipt_long` | Mis Pedidos |
| Perfil | `person` | Mi Cuenta |

## Detalle de cada pantalla

### 1. Splash Screen
- Logo BlackDog animado
- Verificar sesión activa → Home o Login

### 2. Onboarding (primera vez)
- 3 slides: "Compra para tu mascota", "Delivery o Pick-up", "Pagos fáciles"
- Botón "Empezar"

### 3. Login
- Email + Password
- Botones: "Continuar con Google", "Continuar con Apple"
- Link: "¿No tienes cuenta? Regístrate"
- Link: "¿Olvidaste tu contraseña?"

### 4. Registro
- Campos: Nombre completo, Email, Teléfono, Contraseña
- Checkbox: Términos y condiciones
- Mismos botones de social login

### 5. Home Screen
- **Header**: Logo BlackDog + ícono notificaciones (badge)
- **Barra de búsqueda**
- **Banners promocionales** (carousel horizontal, configurables)
- **Categorías rápidas** (íconos horizontales)
- **Productos destacados** (grid 2 columnas)
- **Sección "Ofertas"** (productos con descuento)

### 6. Catálogo / Lista de productos
- **Filtro por categoría** (tabs horizontales)
- **Ordenar por**: Nombre, Precio (asc/desc), Más vendidos, Nuevos
- **Grid de productos** (imagen, nombre, precio, botón "Agregar")
- **Infinite scroll** o paginación

### 7. Detalle de Producto
- **Carousel de imágenes** (zoom on tap)
- **Nombre y SKU**
- **Precio** (original tachado si hay descuento + precio de oferta)
- **Descripción** (HTML renderizado)
- **Stock por sucursal** (expandible: "Disponible en 12 tiendas")
- **Selector de cantidad** (+/-)
- **Botón "Agregar al Carrito"** (prominente, amarillo)
- **Productos relacionados** (carousel horizontal)

### 8. Carrito
- **Lista de items**: imagen, nombre, precio, cantidad (+/-), eliminar
- **Resumen**:
  - Subtotal
  - Delivery fee (si aplica)
  - Total
- **Botón "Proceder al Checkout"**
- **Link "Seguir comprando"**

### 9. Checkout
- **Paso 1: Método de entrega**
  - Radio: "Delivery a domicilio" / "Recoger en tienda"
  - Si delivery: seleccionar dirección o agregar nueva
  - Si pick-up: seleccionar sucursal (lista con stock validado)

- **Paso 2: Método de pago**
  - Tilopay (tarjeta)
  - Yappy
  - Pagar al recoger (solo si pick-up)

- **Paso 3: Resumen**
  - Items, subtotal, delivery, total
  - Botón "Confirmar Orden"

### 10. Pago (WebView)
- WebView embebido con URL de Tilopay o Yappy
- Al completar → redirect a pantalla de confirmación

### 11. Confirmación de Orden
- Animación de check/éxito
- Número de orden: #S00123
- Resumen breve
- Botones: "Ver mi pedido" / "Seguir comprando"

### 12. Mis Pedidos
- **Tabs**: Todos | En proceso | Completados
- **Lista de órdenes**: #orden, fecha, total, estado (badge color)
- Tap → detalle del pedido

### 13. Detalle de Pedido / Tracking
- **Info de orden**: número, fecha, método de pago
- **Timeline de tracking** (visual con estados)
- **Items del pedido** (lista con imágenes)
- **Info de delivery/pickup**:
  - Si delivery: nombre del repartidor, teléfono
  - Si pick-up: dirección de la sucursal, mapa
- **Botón "Contactar"** (si en camino)

### 14. Sucursales (Mapa)
- **Google Maps** con pins de todas las sucursales
- **Lista debajo** del mapa (o bottom sheet)
- Cada sucursal: nombre, dirección, horario, abierto/cerrado, distancia
- **Botones**: "Cómo llegar" (abre Google Maps) / "Llamar"

### 15. Perfil / Mi Cuenta
- Foto, nombre, email, teléfono
- **Mis direcciones** (CRUD)
- **Notificaciones** (on/off)
- **Sobre la app** (versión, términos, privacidad)
- **Cerrar sesión**

### 16. Centro de Notificaciones
- Lista cronológica de notificaciones
- Indicador de no leídas
- Tap → navega al pedido o promoción relevante

## Diseño

Basado en el prototipo de Stitch (proyecto "Brand Style Guide Variant 1"):
- Color primario: `#F7B104`
- Dark mode soportado
- Tipografía: Montserrat + Inter
- Border radius generoso (8px-24px)
- Sombras suaves

Ver [[10 - Sistema de Diseño (Brand Guide)]] para detalles completos.
