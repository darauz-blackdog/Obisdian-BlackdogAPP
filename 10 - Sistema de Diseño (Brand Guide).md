---
tags: [blackdog-app, design, brand]
created: 2026-02-15
status: planning
---

# Sistema de Diseño (Brand Guide)

Basado en el prototipo de **Stitch** (proyecto: Brand Style Guide Variant 1).

## Paleta de colores

### Colores principales
| Token | Hex | Uso |
|-------|-----|-----|
| **Primary** | `#F7B104` | CTAs, acentos, glow, marca |
| **Secondary** | `#1A1A1A` | Texto principal, fondos oscuros |
| **Background Light** | `#FAFAFA` | Fondo de página (light mode) |
| **Background Dark** | `#121212` | Fondo de página (dark mode) |
| **Surface Light** | `#FFFFFF` | Cards y superficies (light) |
| **Surface Dark** | `#1E1E1E` | Cards y superficies (dark) |
| **Text Subtle** | `#888888` | Texto secundario, placeholders |

### Espectro amarillo (variantes)
| Shade | Hex |
|-------|-----|
| yellow-50 | `#FEFCE8` |
| yellow-100 | `#FEF9C3` |
| yellow-200 | `#FEF08A` |
| yellow-300 | `#FDE047` |
| yellow-400 | `#FACC15` |
| **yellow-500 (Primary)** | **`#F7B104`** |
| yellow-600 | `#CA8A04` |
| yellow-700 | `#A16207` |
| yellow-800 | `#854D0E` |
| yellow-900 | `#713F12` |

### Colores semánticos
| Token | Hex | Uso |
|-------|-----|-----|
| Success | `#22C55E` | Confirmaciones, entregado |
| Error | `#EF4444` | Errores, cancelado |
| Warning | `#F59E0B` | Alertas |
| Info | `#3B82F6` | Información |

## Tipografía

| Rol | Fuente | Peso | Tamaño |
|-----|--------|------|--------|
| **H1 (Display)** | Montserrat | Bold (700) | 32px |
| **H2** | Montserrat | SemiBold (600) | 24px |
| **H3** | Montserrat | SemiBold (600) | 18px |
| **Body** | Inter | Regular (400) | 16px |
| **Body Small** | Inter | Regular (400) | 14px |
| **Caption** | Inter | Medium (500) | 12px |
| **Button** | Inter | SemiBold (600) | 16px |

### En Flutter
```dart
// app/theme.dart
final textTheme = TextTheme(
  displayLarge: GoogleFonts.montserrat(fontSize: 32, fontWeight: FontWeight.bold),
  headlineMedium: GoogleFonts.montserrat(fontSize: 24, fontWeight: FontWeight.w600),
  titleLarge: GoogleFonts.montserrat(fontSize: 18, fontWeight: FontWeight.w600),
  bodyLarge: GoogleFonts.inter(fontSize: 16),
  bodyMedium: GoogleFonts.inter(fontSize: 14),
  labelSmall: GoogleFonts.inter(fontSize: 12, fontWeight: FontWeight.w500),
);
```

## Espaciado

Sistema basado en múltiplos de **4px** (escala Tailwind):

| Token | Valor | Uso |
|-------|-------|-----|
| xs | 4px | Separación mínima entre elementos |
| sm | 8px | Padding interno de badges |
| md | 16px | Padding de cards, gaps de grid |
| lg | 24px | Márgenes de sección |
| xl | 32px | Padding de página |
| 2xl | 48px | Separación entre secciones |

## Border Radius

| Token | Valor | Uso |
|-------|-------|-----|
| sm | 8px (0.5rem) | Botones, inputs, badges |
| md | 12px (0.75rem) | Cards pequeñas |
| lg | 16px (1rem) | Cards medianas, modals |
| xl | 24px (1.5rem) | Cards grandes, hero sections |
| full | 9999px | Avatares, pills, FAB |

## Sombras

| Token | Valor | Uso |
|-------|-------|-----|
| **soft** | `0 4px 20px -2px rgba(0,0,0,0.05)` | Elevación sutil de cards |
| **glow** | `0 0 15px rgba(247,177,4,0.3)` | CTAs activos, focus ring (efecto de marca) |
| **float** | `0 10px 25px -5px rgba(0,0,0,0.1)` | Elementos flotantes, bottom sheet |

### En Flutter
```dart
final shadowSoft = BoxShadow(
  color: Colors.black.withOpacity(0.05),
  blurRadius: 20,
  offset: Offset(0, 4),
  spreadRadius: -2,
);

final shadowGlow = BoxShadow(
  color: Color(0xFFF7B104).withOpacity(0.3),
  blurRadius: 15,
);
```

## Componentes

### Botón Primario
- Background: `#F7B104`
- Texto: `#1A1A1A` (dark)
- Border radius: 8px
- Padding: 16px vertical, 24px horizontal
- Shadow on press: `shadow-glow`

### Botón Secundario
- Background: transparent
- Border: 1px `#1A1A1A`
- Texto: `#1A1A1A`
- Border radius: 8px

### Product Card
- Background: Surface (white/dark)
- Border radius: 16px
- Shadow: `shadow-soft`
- Imagen: aspect ratio 1:1, radius top 16px
- Padding content: 12px
- Botón "Agregar": ícono `add_shopping_cart`
- Heart/favorito: top-right corner

### Input Field
- Border: 1px `#E5E5E5` (light) / `#333` (dark)
- Border radius: 8px
- Padding: 12px 16px
- Focus border: `#F7B104`
- Placeholder: `#888888`

### Bottom Navigation
- Background: Surface
- Active: `#F7B104` (ícono + label)
- Inactive: `#888888`
- Shadow top: `shadow-float` invertido

## Dark Mode

Dark mode soportado desde día 1 usando la estrategia de tokens:

| Token | Light | Dark |
|-------|-------|------|
| Background | `#FAFAFA` | `#121212` |
| Surface | `#FFFFFF` | `#1E1E1E` |
| Text Primary | `#1A1A1A` | `#FAFAFA` |
| Text Secondary | `#888888` | `#A0A0A0` |
| Border | `#E5E5E5` | `#333333` |
| Primary | `#F7B104` | `#F7B104` (sin cambio) |

## Iconografía

Google Material Symbols (outlined) — consistente con el prototipo Stitch.

Íconos principales de la app:
- `pets` — Logo / mascota
- `home` — Home
- `shopping_bag` — Tienda
- `shopping_cart` — Carrito
- `receipt_long` — Pedidos
- `person` — Perfil
- `notifications` — Notificaciones
- `search` — Búsqueda
- `favorite` — Favoritos
- `local_shipping` — Delivery
- `store` — Pick-up
- `map` — Sucursales
