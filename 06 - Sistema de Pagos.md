---
tags: [blackdog-app, payments, tilopay, yappy]
created: 2026-02-15
status: planning
---

# Sistema de Pagos

## Métodos soportados

| Método | Tipo | Flujo |
|--------|------|-------|
| **Tilopay** | Tarjeta de crédito/débito | Redirect a WebView → Webhook |
| **Yappy** | Pago móvil (Banco General) | Redirect a Yappy → Webhook |
| **Pago en tienda** | Efectivo/tarjeta al recoger | Sin procesamiento online |

## Flujo general de pago

```
┌───────────┐     ┌──────────┐     ┌──────────────┐
│ Flutter   │────▶│ Backend  │────▶│ Tilopay/Yappy│
│ Checkout  │     │ /payments│     │ API          │
└───────────┘     └──────────┘     └──────┬───────┘
                                          │
                  ┌──────────┐            │ Payment URL
                  │ WebView  │◀───────────┘
                  │ in App   │
                  └────┬─────┘
                       │ Cliente paga
                       ▼
              ┌──────────────┐
              │  Webhook     │
              │  /webhooks/* │
              └──────┬───────┘
                     │
              ┌──────▼──────┐     ┌──────────┐
              │ Confirmar   │────▶│  Odoo    │
              │ Orden       │     │ confirm  │
              └──────┬──────┘     └──────────┘
                     │
              ┌──────▼──────┐
              │ Push Notif  │
              │ "Pago OK"   │
              └─────────────┘
```

## Tilopay

Tilopay es un gateway de pagos panameño. Ya está integrado en Odoo.

### Flujo
1. App envía `POST /api/payments/tilopay/create` con el `order_id`
2. Backend crea un payment link via Tilopay API
3. Backend retorna la `payment_url`
4. Flutter abre un WebView con esa URL
5. Cliente ingresa datos de tarjeta y paga
6. Tilopay envía webhook a `POST /api/webhooks/tilopay`
7. Backend valida la firma del webhook
8. Si exitoso: confirma la sale.order en Odoo + notifica al cliente
9. Si fallido: marca la orden como payment_failed + notifica

### Datos para Tilopay
```json
{
  "amount": 67.00,
  "currency": "USD",
  "description": "BlackDog App - Orden #S00123",
  "order_id": "uuid-de-la-orden",
  "callback_url": "https://api.blackdogapp.com/api/webhooks/tilopay",
  "redirect_url": "blackdogapp://payment/success",
  "customer": {
    "name": "Juan Pérez",
    "email": "juan@email.com"
  }
}
```

## Yappy

Yappy es el sistema de pagos móviles de Banco General, muy popular en Panamá.

### Flujo
1. App envía `POST /api/payments/yappy/create` con el `order_id`
2. Backend crea solicitud de pago via Yappy API
3. Backend retorna la `yappy_url` o deep link
4. Flutter abre Yappy (deep link) o WebView
5. Cliente confirma pago en su app Yappy
6. Yappy envía webhook a `POST /api/webhooks/yappy`
7. Backend valida y procesa igual que Tilopay

## Pago en tienda

Para pedidos con pick-up en sucursal:

1. Cliente selecciona "Pagar al recoger" en checkout
2. Backend crea la sale.order en Odoo en estado **draft**
3. La orden se marca como `pending_payment` en la app
4. Cuando el cliente llega a la sucursal, paga en el POS
5. El POS asocia el pago a la sale.order
6. Webhook o sync detecta el pago → notifica al cliente

## Estados de pago

| Estado | Descripción |
|--------|-------------|
| `pending` | Orden creada, esperando pago |
| `processing` | Pago en proceso (WebView abierto) |
| `paid` | Pago confirmado exitosamente |
| `failed` | Pago rechazado o con error |
| `refunded` | Pago reembolsado |
| `in_store_pending` | Pendiente de pago en sucursal |

## Seguridad

- Webhooks validados por **firma HMAC** (secret compartido con Tilopay/Yappy)
- Los montos se validan en el backend contra la orden original (evitar manipulación)
- Payment URLs tienen **expiración de 30 minutos**
- SSL obligatorio en todos los endpoints
- PCI compliance: los datos de tarjeta **nunca** pasan por nuestro backend (Tilopay los maneja)
