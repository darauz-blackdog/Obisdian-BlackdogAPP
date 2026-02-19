---
tags: [blackdog-app, index]
created: 2026-02-15
updated: 2026-02-15
status: in-progress
---

# BlackDog App - Documentacion del Proyecto

## Resumen Ejecutivo

BlackDog App es una **aplicacion movil de e-commerce** (iOS y Android) para la cadena de pet shops **Black Dog Panama**. Permite a los clientes comprar productos para mascotas, elegir entre delivery o pick-up en sucursal, y dar seguimiento a sus pedidos.

## Documentos

### Arquitectura y Tecnico
- [[01 - Vision General y Arquitectura]]
- [[02 - Stack Tecnologico]]
- [[03 - Diseno del Backend API]]
- [[04 - Integracion con Odoo]]
- [[05 - Sincronizacion Odoo-Supabase]]
- [[06 - Sistema de Pagos]]
- [[07 - Sistema de Delivery]]
- [[08 - Notificaciones Push]]

### Producto
- [[09 - Pantallas y Flujo de Usuario]]
- [[10 - Sistema de Diseno (Brand Guide)]]
- [[11 - Fases de Desarrollo]]

### Infraestructura
- [[12 - Hosting e Infraestructura]]
- [[13 - Estado Actual del Proyecto]]
- [[14 - Modulo de Compras (Employee App)]]

---

## Datos Clave

| Aspecto | Decision |
|---------|----------|
| **Plataforma** | Flutter 3.41.1 (iOS + Android) |
| **Backend** | Express.js + TypeScript (puerto 3002) |
| **Base de datos** | Supabase (cache) + Odoo 18 (fuente de verdad) |
| **Auth** | Supabase Auth (email + Google + Apple) |
| **Pagos** | Tilopay + Yappy + Pago en tienda |
| **Entrega** | Delivery propio + Pick-up en sucursal |
| **Hosting** | VPS actual de PeopleBD |
| **Diseno** | Basado en prototipo Stitch |

## Repositorios

| Repo | URL |
|------|-----|
| Flutter App | https://github.com/darauz-blackdog/blackdog-app |
| Backend API | https://github.com/darauz-blackdog/blackdog-api |
| Documentacion | https://github.com/darauz-blackdog/Obisdian-BlackdogAPP |

## Estado del Proyecto

- [x] Investigacion y prototipo en Stitch
- [x] Definicion de stack y arquitectura
- [x] **Fase 0**: Infraestructura (backend, Supabase, sync Odoo, systemd)
- [x] **Fase 1**: Auth + Catalogo (backend endpoints + Flutter screens)
- [ ] **Fase 2**: Carrito + Checkout + Pagos
- [ ] **Fase 3**: Pedidos + Tracking + Notificaciones
- [ ] **Fase 4**: Sucursales + Perfil completo
- [ ] **Fase 5**: QA + Deploy a Stores
