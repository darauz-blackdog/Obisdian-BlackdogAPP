---
tags: [blackdog-app, notifications, firebase]
created: 2026-02-15
status: planning
---

# Notificaciones Push

## Tecnología

**Firebase Cloud Messaging (FCM)** — gratuito, soporta iOS y Android.

## Flujo de registro

```
1. Usuario instala la app y hace login
2. Flutter solicita permiso de notificaciones (iOS requiere prompt)
3. Firebase genera un FCM token
4. App envía el token al backend: POST /api/notifications/register
5. Backend guarda en tabla push_tokens (user_id + token + platform)
```

## Tipos de notificaciones

### Transaccionales (orden)

| Evento | Título | Mensaje |
|--------|--------|---------|
| Pago confirmado | "Pago recibido" | "Tu orden #S00123 ha sido confirmada. Estamos preparándola." |
| Pedido preparándose | "Preparando tu pedido" | "Estamos alistando tus productos en Brisas del Golf." |
| Listo para recoger | "Pedido listo" | "Tu pedido #S00123 está listo para recoger en Brisas del Golf." |
| En camino | "En camino" | "Alex está llevando tu pedido. Llegada estimada: 12:45 PM." |
| Entregado | "Pedido entregado" | "Tu pedido #S00123 ha sido entregado. Gracias por tu compra." |
| Pago fallido | "Problema con el pago" | "No pudimos procesar tu pago. Intenta de nuevo." |
| Cancelación | "Orden cancelada" | "Tu orden #S00123 ha sido cancelada." |

### Promocionales (futuro)

| Evento | Título | Mensaje |
|--------|--------|---------|
| Oferta flash | "Oferta del día" | "20% off en alimentos Royal Canin. Solo hoy." |
| Nuevo producto | "Nuevo en BlackDog" | "Llegó la nueva línea de juguetes Kong." |
| Recordatorio carrito | "Olvidaste algo" | "Tienes items en tu carrito. Completa tu compra." |

## Implementación backend

```typescript
// services/notification.service.ts
import admin from 'firebase-admin';

async function sendPushNotification(
  userId: string,
  title: string,
  body: string,
  data?: Record<string, string>
) {
  // Obtener tokens del usuario
  const { data: tokens } = await supabase
    .from('push_tokens')
    .select('token')
    .eq('user_id', userId);

  if (!tokens?.length) return;

  // Enviar a todos los dispositivos del usuario
  const message = {
    notification: { title, body },
    data: data || {},
    tokens: tokens.map(t => t.token),
  };

  const response = await admin.messaging().sendEachForMulticast(message);

  // Limpiar tokens inválidos
  response.responses.forEach((resp, idx) => {
    if (resp.error?.code === 'messaging/registration-token-not-registered') {
      supabase.from('push_tokens').delete().eq('token', tokens[idx].token);
    }
  });
}
```

## Implementación Flutter

```dart
// core/notifications/notification_service.dart
class NotificationService {
  final FirebaseMessaging _messaging = FirebaseMessaging.instance;

  Future<void> initialize() async {
    // Solicitar permisos (iOS)
    await _messaging.requestPermission(
      alert: true,
      badge: true,
      sound: true,
    );

    // Obtener token
    final token = await _messaging.getToken();
    if (token != null) {
      await _registerToken(token);
    }

    // Escuchar cambios de token
    _messaging.onTokenRefresh.listen(_registerToken);

    // Manejar notificaciones en foreground
    FirebaseMessaging.onMessage.listen(_handleForegroundMessage);

    // Manejar tap en notificación (app en background)
    FirebaseMessaging.onMessageOpenedApp.listen(_handleNotificationTap);
  }
}
```

## Centro de notificaciones en la app

La app tendrá una pantalla de **historial de notificaciones** donde el usuario puede ver todas las notificaciones recibidas, incluso si no las leyó.

```sql
-- Tabla en Supabase
notifications (
  id uuid PK,
  user_id uuid FK,
  title text,
  body text,
  type text,           -- 'order', 'promo', 'system'
  reference_id text,   -- order_id o product_id
  is_read boolean DEFAULT false,
  created_at timestamptz
)
```
