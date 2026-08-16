# Cortina Textil Oriol — Shopify Theme Project

Tema personalizado para tienda de telas y cortinas. El objetivo es entregar una experiencia de e-commerce de calidad premium, acorde a los estándares visuales de tiendas de nicho especializadas (textiles, moda, interiorismo).

---

## Estado general del proyecto

**Fase actual:** Desarrollo activo — PDP funcional, swatches operativos vía colecciones, acordeones rediseñados. Pendiente: mega menú, iconos de cuidado, PLP, home.

---

## Lo que ya está hecho

### Header (`sections/header.liquid`)
- Barra superior (`.header-top`) con logo, búsqueda, iconos de cuenta y carrito
- Barra de categorías (`.header-bottom`) con navegación dinámica por `linklists`
- Menú lateral de hamburguesa activado desde el botón "Todas las Categorías"
- Alturas compactas: 50px (top) + 36px (bottom) = 86px total
- Logo: `max-height: 34px`; fuentes: 13px nav, 12px categorías
- `body { padding-top: 86px }` para compensar header fijo
- Variables CSS para colores y tipografía editables desde el editor de temas

### PDP — Página de Producto (`sections/pdp.liquid`)

#### Layout y galería
- Dos columnas: **60% galería / 40% tarjeta** con `grid-template-columns: 3fr 2fr`
- Section padding: `10px 0 80px !important` (sobreescribe CSS global que metía 47px horizontal)
- Galería desktop: apilada verticalmente
- Galería mobile: slider horizontal con `scroll-snap-type: x mandatory` y dots JS
- Tarjeta sticky: `position: sticky; top: 90px`

#### Tarjeta de información
- Contador de metros: pasos de 0.5m (0.5 → 1.0 → 1.5…), botones +/- en JS
- Fecha de entrega dinámica en JS: calcula rango de días hábiles, omite domingos
- Botón "Comprar ahora" y "Agregar al carrito" (AJAX `/cart/add.js`)
- Botón wishlist con localStorage

#### Swatches de colores relacionados
- **Método actual (producción):** detecta automáticamente la colección `{tipotela}-colores` desde la primera palabra del título del producto
  ```liquid
  product.title → split → first word → downcase → + '-colores' → collections[handle]
  ```
  Ejemplo: "RASO AZUL 6017" → busca colección `raso-colores`
- **Colecciones creadas:** RASO, CREP, MUSELINA, POPELIN, LAREDO, GASA — Colores (condición automática por nombre)
- Color HEX por metacampo `custom.color_hex` (tipo `color`) — ya configurado en todos los productos
- Swatch activo: anillo exterior via `::after`, sheen de tela via `radial-gradient`
- Tooltip CSS: `attr(data-color-name)` en `::before`
- Swatches demo visibles solo en el editor (`request.design_mode`): Beige Natural, Terracota, Gris Piedra, Azul Noche, Verde Salvia, Blanco Roto

#### Acordeones
- Toggle `+` / `−` via CSS `::before` / `::after` (reemplazó chevron SVG que era enorme)
- Hover con `opacity: 0.65` en el header completo
- Transición: `cubic-bezier(0.4, 0, 0.2, 1)` suave
- **Acordeón 1 — Descripción:** muestra `product.description` con CSS normalizado para `h1–h5`, `p`, `ul/ol`, `li`, `strong` (todos en 14px, sin tamaños heredados del tema)
- **Acordeón 2 — Instrucciones de cuidado:**
  - Metacampo: `product.metafields.shopify['care-instructions']` tipo `list.metaobject_reference`
  - Devuelve MetaobjectDrop — se accede con `.label | .name | .title | .handle`
  - **Estado actual:** muestra lista de texto simple (bullet points) mientras se resuelve el acceso exacto al campo del metaobjeto
  - El bloque de iconos SVG está oculto (`display:none`) listo para activar cuando se confirme el nombre del campo
  - Los SVGs de iconos están escritos (tub para lavado, triángulo para blanqueador, cuadrado+círculo para secadora, plancha lateral, círculo para dry-clean)
- **Acordeón 3 — Envío:** opcional, oculto por defecto (`show_accordion3: false`)

#### Instrucciones de cuidado — estado técnico
- El metacampo `shopify.care-instructions` es de tipo `list.metaobject_reference`, no `list.single_line_text_field`
- Cada entrada es un metaobjeto con campos propios (nombre desconocido hasta confirmar en admin)
- Valores configurados en el producto de prueba: "Lavable a máquina", "Instrucciones de planchado", "Apto para secadora", "Machine washable"
- **Pendiente:** confirmar el nombre del campo del metaobjeto (`.label`, `.name`, `.title` u otro) para que aparezca el texto correcto y luego activar los iconos SVG

### Páginas de cuenta (`sections/snc-main-*.liquid`)
- `snc-main-login.liquid`: formulario de inicio de sesión
- `snc-main-register.liquid`: formulario de registro
- `snc-main-account.liquid`: panel con historial de pedidos, datos y link a direcciones

### Otras secciones
- `snc-info-hero.liquid`: hero informativo para páginas estáticas
- `snc-custom-content-text.liquid`: bloque de texto enriquecido reutilizable

---

## Lo que falta por hacer

### 1. Mega Menú (PENDIENTE — prioridad alta)
Reemplazar el drawer lateral de "Todas las Categorías" por un **mega menú desplegable al hacer hover**.

**Comportamiento esperado:**
- Hover sobre "Todas las Categorías" en `.header-bottom` → panel que se abre hacia abajo
- Panel muestra categorías principales (tipos de tela) con sus subcategorías
- Se cierra al mover el cursor fuera
- Compatible con `linklists` de Shopify
- Mobile: mantener drawer lateral o adaptarlo a acordeón

**Archivo a modificar:** `sections/header.liquid`

---

### 2. Iconos de instrucciones de cuidado (PENDIENTE)
Los SVGs ya están escritos y correctos. Falta:
- Confirmar el campo exacto del metaobjeto de cuidado (`.label`, `.name`, `.title`)
- Activar el bloque de iconos (quitar el `display:none` del `.pdp-care-items`)
- Opcionalmente, que el cliente mapee sus entradas custom ("Lavable a máquina", etc.) a los handles estándar ISO o crear entradas compatibles

---

### 3. PDP — pendientes menores
- Validar slider mobile en iOS Safari
- Testear fecha de entrega en diferentes zonas horarias
- Confirmar que swatches cargan en todos los tipos de tela (depende del handle de la colección)

---

### 4. Otras secciones por desarrollar
- **PLP** (colección): grid de productos con filtros Search & Discovery
- **Home**: hero banner, colecciones destacadas, contenido editorial
- **Footer** personalizado
- **Página de búsqueda**
- **Página de contacto**

---

## Notas técnicas importantes

### Swatches — cómo funciona
```
Título del producto → primera palabra → lowercase → + "-colores" → handle de colección
"RASO AZUL 6017"   → "RASO"          → "raso"    → "raso-colores" → collections["raso-colores"]
```
Las colecciones deben tener exactamente ese handle. Si Shopify genera handles diferentes (ej. con números o guiones adicionales), verificar en Admin → Colecciones → editar handle manualmente.

### Care instructions — MetaobjectDrop
El metacampo `shopify['care-instructions']` devuelve una lista de metaobjetos. Para obtener el texto visible, probar en orden:
```liquid
instruction.label | instruction.name | instruction.title | instruction.handle
```
El primero que devuelva texto es el campo correcto para ese metaobjeto.

### Padding section PDP
El CSS global del tema añadía `padding: 10px 47px 80px` a la section. Se sobreescribe con `!important` en `.pdp-section-{{ section.id }}`.

---

## Resultado final esperado

Una tienda Shopify de textiles con:
- **Header compacto** con mega menú de categorías al hover
- **PDP rica**: galería, swatches por colección, cuidados con iconos, entrega estimada, metros
- **Cuenta de cliente completa**: login, registro, panel de pedidos
- **Estética coherente**: colores neutros, botones redondeados, tipografía limpia
- **Totalmente editable** desde el editor de temas (settings en cada sección)
- **Mobile-first**: todas las secciones responsivas

---

## Archivos del proyecto

```
sections/
  header.liquid                  — Header con navegación y menú de categorías
  pdp.liquid                     — Página de producto (PDP)
  snc-main-login.liquid          — Login de cuenta
  snc-main-register.liquid       — Registro de cuenta
  snc-main-account.liquid        — Panel de cuenta del cliente
  snc-info-hero.liquid           — Hero informativo
  snc-custom-content-text.liquid — Bloque de texto enriquecido

templates/
  product.json                   — Template de producto con settings guardados

PROYECTO.md                      — Este documento
```
