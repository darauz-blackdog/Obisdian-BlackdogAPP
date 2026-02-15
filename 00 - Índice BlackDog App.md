---
tags: [blackdog-app, index]
created: 2026-02-15
status: planning
---

# BlackDog App - Documentación del Proyecto

## Resumen Ejecutivo

BlackDog App es una **aplicación móvil de e-commerce** (iOS y Android) para la cadena de pet shops **Black Dog Panamá**. Permite a los clientes comprar productos para mascotas, elegir entre delivery o pick-up en sucursal, y dar seguimiento a sus pedidos.

## Documentos

### Arquitectura y Técnico
- [[01 - Visión General y Arquitectura]]
- [[02 - Stack Tecnológico]]
- [[03 - Diseño del Backend API]]
- [[04 - Integración con Odoo]]
- [[05 - Sincronización Odoo-Supabase]]
- [[06 - Sistema de Pagos]]
- [[07 - Sistema de Delivery]]
- [[08 - Notificaciones Push]]

### Producto
- [[09 - Pantallas y Flujo de Usuario]]
- [[10 - Sistema de Diseño (Brand Guide)]]
- [[11 - Fases de Desarrollo]]

### Infraestructura
- [[12 - Hosting e Infraestructura]]

---

## Datos Clave

| Aspecto | Decisión |
|---------|----------|
| **Plataforma** | Flutter (iOS + Android) |
| **Backend** | Express.js + TypeScript |
| **Base de datos** | Supabase (cache) + Odoo (fuente de verdad) |
| **Auth** | Supabase Auth (email + Google + Apple) |
| **Pagos** | Tilopay + Yappy + Pago en tienda |
| **Entrega** | Delivery propio + Pick-up en sucursal |
| **Hosting** | VPS actual de PeopleBD (temporal) |
| **Diseño** | Basado en prototipo Stitch |

## Estado del Proyecto

- [x] Investigación y prototipo en Stitch
- [x] Definición de stack y arquitectura
- [ ] Setup de repositorios
- [ ] Fase 0: Infraestructura
- [ ] Fase 1: Auth + Catálogo
- [ ] Fase 2: Carrito + Checkout + Pagos
- [ ] Fase 3: Pedidos + Tracking + Notificaciones
- [ ] Fase 4: Sucursales + Perfil
- [ ] Fase 5: QA + Deploy a Stores
