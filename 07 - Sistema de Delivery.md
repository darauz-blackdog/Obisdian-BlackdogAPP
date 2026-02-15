---
tags: [blackdog-app, delivery, logistics]
created: 2026-02-15
status: planning
---

# Sistema de Delivery

## Tipos de entrega

| Tipo | Descripción |
|------|-------------|
| **Pick-up en sucursal** | Cliente recoge en la tienda seleccionada |
| **Delivery a domicilio** | Repartidor de Black Dog lleva al cliente |

## Pick-up en sucursal

### Flujo
```
1. Cliente selecciona "Recoger en tienda" en checkout
2. Elige sucursal (se valida stock en esa sucursal)
3. Completa el pago
4. Sucursal recibe la orden (notificación interna)
5. Personal prepara el pedido
6. Cuando está listo → push notification "Tu pedido está listo para recoger"
7. Cliente va a la sucursal y recoge
8. Personal marca como entregado
```

### Validación de stock
- Solo se muestran sucursales donde **todos los items** del carrito tengan stock
- Si un item no tiene stock en ninguna sucursal → mensaje de agotado

## Delivery a domicilio

### Flujo
```
1. Cliente selecciona "Delivery" en checkout
2. Ingresa o selecciona dirección guardada
3. Se calcula la tarifa de delivery según zona
4. Completa el pago
5. Orden llega al sistema → se asigna repartidor
6. Repartidor recoge en sucursal más cercana con stock
7. Tracking de estados:
   - Confirmado
   - Preparando
   - En camino (repartidor asignado)
   - Entregado
8. Push notification en cada cambio de estado
```

### Tarifas de delivery

| Zona | Tarifa | Descripción |
|------|--------|-------------|
| Zona 1 | $3.50 | Ciudad de Panamá centro |
| Zona 2 | $5.00 | Áreas suburbanas (Condado del Rey, Brisas, etc.) |
| Zona 3 | $7.50 | Áreas lejanas (Clayton, Arraiján) |
| Zona 4 | $10.00+ | David, Chiriquí (si aplica) |
| Gratis | $0.00 | Órdenes arriba de $X (promocional) |

> Las tarifas son configurables. Pueden definirse en el backend (tabla en Supabase) o en Odoo.

### Repartidores

Los repartidores son **empleados de Black Dog**. Para la fase 1, la asignación es **manual**:

1. La orden llega al dashboard interno (puede ser Odoo o un panel web)
2. Un coordinador asigna el repartidor
3. El repartidor recibe notificación (WhatsApp o app interna futura)
4. El repartidor actualiza el estado (en camino, entregado)

**Fase 2 (futuro):**
- App para repartidores (Flutter) con GPS tracking
- Asignación automática por cercanía
- Tracking en tiempo real para el cliente

### Actualización de estados

Para fase 1, los estados se actualizan manualmente desde:
- **Odoo**: El personal de tienda/coordinador cambia el estado del picking (`stock.picking`)
- **Panel web simple**: Una interfaz ligera donde se actualiza el estado de delivery

El backend hace polling o recibe webhooks para notificar al cliente.

## Tracking de pedidos

### Estados del pedido

```
pending_payment → confirmed → preparing → ready_pickup / shipping → delivered
                                                                  → cancelled
```

| Estado | Descripción | Quién lo activa |
|--------|-------------|----------------|
| `pending_payment` | Orden creada, esperando pago | Sistema (al crear orden) |
| `confirmed` | Pago recibido | Webhook de pago |
| `preparing` | Tienda preparando el pedido | Personal de tienda |
| `ready_pickup` | Listo para recoger en sucursal | Personal de tienda |
| `shipping` | En camino al cliente | Repartidor/coordinador |
| `delivered` | Entregado al cliente | Repartidor/coordinador |
| `cancelled` | Cancelado | Cliente o admin |

### Timeline en la app (ejemplo delivery)

```
✅ Orden confirmada          — 12:15 PM
✅ Preparando tu pedido      — 12:28 PM
🔵 En camino                 — Alex está de camino
⚪ Entregado                 — Pendiente
```

### Timeline en la app (ejemplo pick-up)

```
✅ Orden confirmada          — 12:15 PM
✅ Preparando tu pedido      — 12:28 PM
🔵 Listo para recoger       — Tu pedido te espera en Brisas del Golf
⚪ Recogido                  — Pendiente
```

## Horarios de delivery

- Lunes a Sábado: 9:00 AM - 7:00 PM
- Domingo: 10:00 AM - 5:00 PM
- No se aceptan órdenes de delivery fuera de horario (solo pick-up)

> Los horarios son configurables en el backend.
