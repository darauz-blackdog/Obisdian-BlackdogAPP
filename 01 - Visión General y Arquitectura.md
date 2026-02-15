---
tags: [blackdog-app, architecture]
created: 2026-02-15
status: planning
---

# Visión General y Arquitectura

## Problema

Black Dog Panamá tiene **20+ sucursales** y **4,484 productos** en Odoo. Actualmente venden online a través de Shopify (sincronizado con Odoo vía `shopify_ept`), pero necesitan una **app móvil propia** que:

1. Refleje la identidad de marca BlackDog
2. Permita compras con delivery propio y pick-up en sucursal
3. Soporte pagos locales (Tilopay, Yappy)
4. Envíe notificaciones push a los clientes
5. Se integre completamente con Odoo (inventario, órdenes, clientes)

## Arquitectura General

```
┌─────────────────┐
│   Flutter App    │
│  (iOS + Android) │
└────────┬────────┘
         │ HTTPS (REST API)
         │
┌────────▼────────┐     ┌──────────────┐
│ Express Backend  │────▶│    Odoo 17    │
│  (TypeScript)    │     │  (XML-RPC)   │
│  Puerto: 3002    │     │  4,484 prods │
└────────┬────────┘     │  20+ tiendas │
         │              └──────────────┘
    ┌────▼────┐
    │Supabase │
    │ - Auth  │
    │ - Cache │
    │ - RLS   │
    └────┬────┘
         │
    ┌────▼─────┐
    │ Firebase │
    │  - FCM   │
    │  - Crash │
    └──────────┘
```

## Por qué esta arquitectura

### ¿Por qué un backend intermedio y no API directa a Odoo?

| Aspecto | API directa Odoo | Backend intermedio |
|---------|------------------|-------------------|
| **Velocidad** | 500ms+ por request | <100ms con cache |
| **Auth** | Sesiones Odoo (frágil) | JWT / Supabase Auth |
| **Caching** | No tiene | Supabase como cache |
| **Push Notifications** | No soporta | Firebase FCM |
| **Webhooks de pago** | Difícil de manejar | Endpoints dedicados |
| **Seguridad** | Credenciales Odoo en el cliente | Credenciales solo en el server |
| **Rate limiting** | No tiene | Configurable |

### ¿Por qué Flutter y no React Native?

- UI pixel-perfect consistente en iOS y Android
- Rendimiento nativo real (compilado a ARM)
- Hot reload para desarrollo rápido
- Gran ecosistema de paquetes para e-commerce
- El diseño del prototipo Stitch se traduce directamente a Flutter widgets

### ¿Por qué Supabase?

- Ya lo usamos en PeopleBD — conocemos el stack
- Auth integrado (email + social login)
- PostgreSQL con RLS para seguridad
- Realtime subscriptions (futuro: chat, tracking en vivo)
- Edge Functions para lógica serverless si se necesita

## Flujo de datos

### Lectura (productos, categorías, stock)
```
Odoo → [Sync Job cada 5 min] → Supabase (cache) → Backend → Flutter App
```

### Escritura (órdenes, pagos, registro)
```
Flutter App → Backend → Odoo (sale.order, res.partner)
                     → Supabase (sesión, carrito)
```

### Pagos
```
Flutter App → Backend → Tilopay/Yappy API → Redirect
Tilopay/Yappy → Webhook → Backend → Confirma orden en Odoo
```

## Sucursales Black Dog (Warehouses en Odoo)

| Sucursal | Código | ID |
|----------|--------|----|
| Plaza Emporio | PE | 1 |
| IMPA (Bodega) | BODE | 2 |
| Ocean Mall | OM | 3 |
| Bella Vista | BV | 4 |
| Albrook Fields | AF | 6 |
| Brisas del Golf | BDG | 7 |
| Calle 50 | CA50 | 8 |
| Santa Maria | SM | 10 |
| Costa Verde | CV | 15 |
| Villa Zaita | VZ | 16 |
| Condado del Rey | CDREY | 17 |
| Brisas Norte | BNORT | 18 |
| Versalles | VS | 21 |
| Coco del Mar | CDM | 22 |
| Chiriquí David | DAV | 23 |
| Parque Omar | PO | 29 |
| Los Pueblos | LP | 30 |
| Cantabria | CANT | 31 |

> **Nota:** Mobile Grooming (MG), Consumo Interno (CONSU), Mercancía de Descarte (DESCA) y las bodegas de distribución NO se muestran a clientes.

## Categorías de Productos (Odoo)

```
Vendibles/
├── Accesorios/
│   └── Perro/
├── Alimentos/
│   ├── Perro/
│   ├── Gato/
│   ├── Treats/
│   └── Moz Pet/
├── Medicamentos/ (20+ subcategorías)
├── Servicios/ (7 subcategorías)
└── Humano/
```

> **Nota:** Los productos de tipo "servicio" y "Medicamentos" (recetados) probablemente no se venderán en la app. Solo productos de retail (Alimentos, Accesorios, Treats).
