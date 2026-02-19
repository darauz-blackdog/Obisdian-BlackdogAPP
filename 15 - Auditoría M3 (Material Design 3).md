# Auditoría Material Design 3 - BlackDog App

> Fecha: 2026-02-19
> Referencia: [m3.material.io](https://m3.material.io/)

## Resumen Ejecutivo

Se realizó una auditoría completa de la app BlackDog contra las guías de Material Design 3, cubriendo **15 pantallas** y los archivos de tema, widgets y navegación.

---

## Problemas Encontrados y Corregidos

### 🔴 Bugs Críticos (4)

| Archivo | Problema | Fix |
|---------|----------|-----|
| `main_shell.dart` | Método `_onTap()` sin declaración de función — error de compilación | Agregada firma `void _onTap(BuildContext context, int index)` |
| `product_detail_screen.dart` | Import y comentario incrustados dentro del widget tree (líneas 32-34) — error de compilación | Removidas líneas erróneas |
| `catalog_screen.dart` | `CartBadge()` usado sin import | Agregado `import '../../widgets/cart_badge.dart'` |
| `profile_screen.dart` | `context.push('/orders')` usado sin `go_router` import | Agregado `import 'package:go_router/go_router.dart'` |

### 🟡 Bug de Navegación (1)

| Archivo | Problema | Fix |
|---------|----------|-----|
| `order_confirmation_screen.dart` | Botón "Mis pedidos" navegaba a `/cart` en vez de `/orders` | Cambiado a `context.go('/orders')` |

### 🟠 Tildes y Acentos (20+ fixes)

| Archivo | Incorrecto | Correcto |
|---------|-----------|----------|
| `home_screen.dart` | Categorias | Categorías |
| `catalog_screen.dart` | Catalogo | Catálogo |
| `catalog_screen.dart` | Mas recientes | Más recientes |
| `cart_screen.dart` | Tu carrito esta vacio | Tu carrito está vacío |
| `cart_screen.dart` | ¿Estas seguro | ¿Estás seguro |
| `product_detail_screen.dart` | Descripcion (x2) | Descripción |
| `profile_screen.dart` | Cerrar sesion | Cerrar sesión |
| `checkout_screen.dart` | via Tilopay | vía Tilopay |
| `checkout_screen.dart` | via Yappy | vía Yappy |
| `login_screen.dart` | Inicia sesion | Inicia sesión |
| `login_screen.dart` | Correo electronico (x2) | Correo electrónico |
| `login_screen.dart` | Correo invalido | Correo inválido |
| `login_screen.dart` | Contrasena (x3) | Contraseña |
| `login_screen.dart` | Minimo | Mínimo |
| `login_screen.dart` | Olvidaste tu contrasena? | ¿Olvidaste tu contraseña? |
| `login_screen.dart` | Iniciar Sesion | Iniciar Sesión |
| `login_screen.dart` | o continua con | o continúa con |
| `login_screen.dart` | Registrate | Regístrate |
| `register_screen.dart` | Correo electronico | Correo electrónico |
| `register_screen.dart` | Contrasena / contrasenas | Contraseña / contraseñas |
| `register_screen.dart` | Telefono | Teléfono |
| `register_screen.dart` | Inicia Sesion | Inicia Sesión |
| `register_screen.dart` | o registrate con | o regístrate con |

### 🟢 M3 Theme Enhancements

| Componente | Cambio |
|-----------|--------|
| `NavigationBarThemeData` | Indicador con color primario semitransparente, altura de 72px, labels siempre visibles, colores de icono para selected/unselected |
| `FadeInUp` widget | Curva animación actualizada de `Curves.easeOut` a `Curves.easeOutCubic` (M3 emphasized easing) |
| `CartBadge` | Usa `Badge.count()` oficial de M3 en vez de badge manual |
| `HomeScreen` | Reemplazado badge manual de Stack con widget reutilizable `CartBadge` |

---

## Guías M3 Aplicadas

### Navegación
- **`NavigationBar`** en vez de `BottomNavigationBar` (componente M3 oficial)
- 4 destinos máximo (Inicio, Mapa, Pedidos, Perfil) — dentro del rango recomendado (3-5)
- Indicador de selección con forma pill y color primario semitransparente
- Labels siempre visibles para accesibilidad

### Tipografía
- Sistema de tipos M3: `Display`, `Headline`, `Title`, `Body`, `Label`
- Fuentes: **Montserrat** para headings, **Inter** para body — ambas sans-serif modernas
- Jerarquía clara en toda la app

### Motion / Animaciones
- Curva **`easeOutCubic`** (M3 emphasized deceleration) en `FadeInUp`
- `AnimatedSwitcher` en checkout para transiciones suaves entre pasos
- `AnimatedContainer` en radio cards para cambios de estado fluidos
- Duración estándar de 500ms para entrada de contenido

### Color
- `ColorScheme.light()` con roles definidos: primary, onPrimary, secondary, surface, error
- Primary: `#F7B104` (dorado BlackDog) — consistente en toda la app
- Badges y chips usan `alpha` para variaciones tonales (patrón M3)

### Componentes
- `Badge.count()` — componente M3 oficial para contadores
- `Card` con `elevation: 0` y bordes sutiles (estilo M3 outlined)
- `ElevatedButton` con esquinas redondeadas de 12px y sin elevación (M3 tonal elevation)
- `InputDecoration` con bordes M3 y padding consistente

---

## Archivos Modificados

| Archivo | Tipo de Fix |
|---------|-------------|
| `lib/theme/app_theme.dart` | NavigationBarThemeData M3 |
| `lib/screens/common/main_shell.dart` | Bug crítico + limpieza |
| `lib/screens/home/home_screen.dart` | CartBadge + tilde |
| `lib/screens/catalog/catalog_screen.dart` | Import + tildes |
| `lib/screens/catalog/product_detail_screen.dart` | Bug crítico + tildes |
| `lib/screens/cart/cart_screen.dart` | Tildes |
| `lib/screens/checkout/checkout_screen.dart` | Tildes |
| `lib/screens/checkout/order_confirmation_screen.dart` | Bug navegación |
| `lib/screens/orders/orders_screen.dart` | ✅ Sin cambios |
| `lib/screens/orders/order_detail_screen.dart` | ✅ Sin cambios |
| `lib/screens/branches/branches_screen.dart` | ✅ Sin cambios |
| `lib/screens/profile/profile_screen.dart` | Import + tilde |
| `lib/screens/auth/login_screen.dart` | 11 tildes |
| `lib/screens/auth/register_screen.dart` | 8 tildes |
| `lib/widgets/fade_in_up.dart` | Curva M3 |
| `lib/widgets/cart_badge.dart` | ✅ Sin cambios |

---

## Fase 2 — Mejoras Avanzadas M3

### 🔄 Page Transitions (M3 Motion)

| Tipo de pantalla | Transición | Duración |
|-----------------|-----------|----------|
| Tabs principales (home, branches, orders, profile) | **Fade-through** | 300ms |
| Pantallas de detalle (producto, orden, checkout, registro) | **Shared axis Y** (slide-up + fade) | 350ms |
| Splash → Login | Fade-through | 300ms |

Curvas usadas: `Curves.easeOutCubic` (M3 emphasized deceleration)

### 🌙 Dark Mode

- `ThemeMode.system` — sigue preferencia del sistema automáticamente
- Colores dark: `#121212` (background), `#1E1E1E` (surface), `#2A2A2A` (surface container)
- Todos los component themes duplicados para dark (botones, inputs, cards, navigation)
- `AppColors` ahora incluye constantes `dark*` para ambos temas

### 📱 Responsive Layout

- **< 600dp**: `NavigationBar` (bottom bar) — experiencia móvil estándar
- **≥ 600dp**: `NavigationRail` (sidebar) — experiencia tablet M3
- `VerticalDivider` separa rail del contenido
- Mismos destinos e iconos en ambas configuraciones

### 🎨 Elevation & Surface Tint

- `scrolledUnderElevation: 0.5` + `surfaceTintColor` en AppBar (M3 tonal elevation)
- Removido `BoxShadow` manual del bottom bar de cart → usa `Theme.of(context).colorScheme.surface`
- Header glassmorphism de home ahora usa `colorScheme.surface` en vez de `Colors.white`
- `CardTheme` con `surfaceTintColor: Colors.transparent` para cards uniformes

### 🔤 Typography Scale (M3 Completo)

| Rol | Fuente | Tamaño | Peso |
|-----|--------|--------|------|
| Display L/M/S | Montserrat | 57/45/36 | w400 |
| Headline L/M/S | Montserrat | 32/28/24 | Bold/Bold/w600 |
| Title L/M/S | Montserrat/Inter | 22/16/14 | w600/w500/w500 |
| Body L/M/S | Inter | 16/14/12 | Regular |
| Label L/M/S | Inter | 14/12/11 | w600/w500/w500 |

### 🧩 Component Themes Nuevos

| Theme | Descripción |
|-------|-------------|
| `chipTheme` | Background M3, bordes redondeados 20px, sin side |
| `snackBarTheme` | Floating, bordes 12px |
| `dividerTheme` | Colores adaptivos light/dark |
| `navigationBarTheme` | Indicador, iconos, labels con colores diferenciados |

---

## Archivos Fase 2

| Archivo | Cambio |
|---------|--------|
| `lib/theme/app_theme.dart` | Rewrite completo: darkTheme, M3 type scale, component themes |
| `lib/config/routes.dart` | Todas las rutas con `pageBuilder` + transiciones M3 |
| `lib/screens/common/main_shell.dart` | NavigationRail responsivo para tablets |
| `lib/main.dart` | `darkTheme` + `ThemeMode.system` |
| `lib/screens/auth/splash_screen.dart` | 'Panamá' tilde |
| `lib/screens/health/health_screen.dart` | 'Próximamente' tilde |
| `lib/screens/cart/cart_screen.dart` | Colores dark-mode-aware |
| `lib/screens/home/home_screen.dart` | Colores dark-mode-aware |

