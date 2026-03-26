# PACTA — Branding & Sistema de Diseño
**Versión:** 2.0  
**Estado:** Producción  
**Fecha:** 2026-03-26  

---

## 1. Identidad de Marca

### 1.1 Nombre y Concepto

**PACTA** proviene del latín *pactum* — acuerdo, contrato, pacto. El nombre evoca solidez, formalidad y confianza, valores centrales en la gestión de contratos. Es corto, pronunciable en español e inglés, y memorable.

**Tagline:** *Contratos bajo control.*  
**Tagline alternativo (inglés):** *Every agreement, organized.*

### 1.2 Valores de Marca

| Valor | Expresión Visual | Expresión UX |
|---|---|---|
| **Confianza** | Azul profundo, tipografía serif en headings | Datos siempre visibles, sin surpresas |
| **Precisión** | Espaciado consistente, grid estricto | Validaciones claras, feedback inmediato |
| **Autoridad** | Paleta sobria, pesos tipográficos fuertes | Lenguaje directo, sin ambigüedad |
| **Claridad** | Alto contraste, jerarquía visual fuerte | Mínimo de clics para llegar a la información |

---

## 2. Paleta de Colores

### 2.1 Colores Primarios

```
Pacta Blue (primario)
Hex: #1B3A6B
RGB: 27, 58, 107
HSL: 216°, 60%, 26%
Uso: Navbar, sidebar activo, botones primarios, headings principales

Pacta Blue Light (acento claro)
Hex: #2D5BE3
RGB: 45, 91, 227
HSL: 221°, 76%, 53%
Uso: Links, hover states, iconos activos, badges de acciones

Slate Dark (contraste)
Hex: #0F1C33
RGB: 15, 28, 51
HSL: 218°, 55%, 13%
Uso: Texto principal sobre fondos claros, navbar texto
```

### 2.2 Colores Secundarios

```
Warm White (fondo principal)
Hex: #F8F9FC
RGB: 248, 249, 252
Uso: Fondo de la app, páginas interiores

Card White (superficies)
Hex: #FFFFFF
RGB: 255, 255, 255
Uso: Cards, modales, tablas, formularios

Border (separadores)
Hex: #E2E8F0
RGB: 226, 232, 240
Uso: Bordes de cards, líneas separadoras, bordes de inputs
```

### 2.3 Colores de Estado (Semánticos)

```
ACTIVO / Éxito
Hex: #16A34A
Light bg: #DCFCE7
Uso: Estado "Activo", mensajes de éxito, confirmaciones

PENDIENTE / Advertencia
Hex: #D97706
Light bg: #FEF3C7
Uso: Estado "Pendiente", alertas de vencimiento próximo

VENCIDO / Error
Hex: #DC2626
Light bg: #FEE2E2
Uso: Estado "Vencido", errores de validación, alertas críticas

CANCELADO / Neutral
Hex: #64748B
Light bg: #F1F5F9
Uso: Estado "Cancelado", elementos inactivos, texto secundario
```

### 2.4 Modo Oscuro (Planificado Post-MVP)

```
Dark Background: #0D1B2A
Dark Surface: #162032
Dark Border: #1E3048
Dark Text Primary: #E8EDF5
Dark Text Secondary: #8DA3BF
```

---

## 3. Tipografía

### 3.1 Sistema Tipográfico

**Display / Headings — Lora (Serif)**
```
Fuente: Lora
Peso: 600 (SemiBold), 700 (Bold)
Google Fonts: https://fonts.google.com/specimen/Lora
Uso: H1, H2 de páginas, nombre de la app, headings de sección importantes
Justificación: Serif con autoridad y elegancia. Comunica permanencia y seriedad
               sin ser rígida. Perfecta para un sistema de contratos.
```

**Cuerpo / UI — Inter**
```
Fuente: Inter (Variable)
Peso: 400 (Regular), 500 (Medium), 600 (SemiBold)
Google Fonts: https://fonts.google.com/specimen/Inter
Uso: Todo el texto de UI — labels, tablas, botones, inputs, párrafos
Justificación: Legibilidad máxima en pantalla, excelente en tamaños pequeños,
               diseñada específicamente para interfaces digitales.
```

**Monospace — JetBrains Mono**
```
Fuente: JetBrains Mono
Peso: 400, 500
Uso: Números de contrato, IDs, código, valores del audit log
Justificación: Los números de contrato y IDs son críticos — la fuente mono
               previene confusión entre caracteres similares (0/O, 1/l).
```

### 3.2 Escala Tipográfica

```css
/* Tailwind custom config */
--text-xs:   0.75rem  / 12px  (metadatos, timestamps)
--text-sm:   0.875rem / 14px  (labels, texto secundario, table cells)
--text-base: 1rem     / 16px  (texto de párrafo, body principal)
--text-lg:   1.125rem / 18px  (subtítulos de sección)
--text-xl:   1.25rem  / 20px  (títulos de card, headings menores)
--text-2xl:  1.5rem   / 24px  (heading de página — H2)
--text-3xl:  1.875rem / 30px  (heading principal de página — H1)
--text-4xl:  2.25rem  / 36px  (números KPI en dashboard)
```

---

## 4. Iconografía

**Librería:** Heroicons (MIT License)  
**Fuente:** https://heroicons.com  
**Estilo:** Outline para iconos de UI, Solid para iconos de acción/estado  
**Tamaño estándar:** 20x20px (interfaz), 24x24px (headings)

### Iconos por módulo:

```
Contratos     → document-text
Clientes      → building-office
Proveedores   → building-storefront
Firmantes     → identification
Suplementos   → document-plus
Documentos    → paper-clip
Reportes      → chart-bar
Notificaciones→ bell
Usuarios      → users
Auditoría     → clock (history)
Dashboard     → home
Configuración → cog-6-tooth

Acciones:
Crear         → plus
Editar        → pencil-square
Eliminar      → trash
Ver detalle   → eye
Buscar        → magnifying-glass
Filtrar       → funnel
Exportar      → arrow-down-tray
Subir         → arrow-up-tray
Cerrar        → x-mark
Confirmar     → check
```

---

## 5. Componentes de UI

### 5.1 Botones

```
Primary (Pacta Blue)
  Fondo: #1B3A6B | Texto: white | Hover: #162F58 | Radius: 6px | Padding: 8px 16px

Secondary (Outline)
  Fondo: white | Borde: #1B3A6B | Texto: #1B3A6B | Hover: #F0F4FF

Danger (Destructive)
  Fondo: #DC2626 | Texto: white | Hover: #B91C1C

Ghost (Subtle)
  Fondo: transparent | Texto: #475569 | Hover bg: #F1F5F9

Disabled
  Opacity: 0.5 | Cursor: not-allowed
```

### 5.2 Badges de Estado

```
ACTIVO:    Fondo #DCFCE7, Texto #166534, Dot #16A34A
PENDIENTE: Fondo #FEF3C7, Texto #92400E, Dot #D97706
VENCIDO:   Fondo #FEE2E2, Texto #991B1B, Dot #DC2626
CANCELADO: Fondo #F1F5F9, Texto #475569, Dot #64748B

Borrador (Suplemento):  Fondo #EFF6FF, Texto #1E40AF
Aprobado (Suplemento):  Fondo #F0FDF4, Texto #166534
```

### 5.3 Cards

```
Fondo: #FFFFFF
Borde: 1px solid #E2E8F0
Radius: 8px
Sombra: 0 1px 3px rgba(0,0,0,0.07), 0 1px 2px rgba(0,0,0,0.04)
Sombra hover: 0 4px 6px rgba(0,0,0,0.08)
Padding: 24px
```

### 5.4 Tablas

```
Header: Fondo #F8F9FC, Texto #374151, Font-weight 600, Font-size 12px, Uppercase
Row: Fondo white, Borde-bottom 1px solid #E2E8F0
Row hover: Fondo #F8F9FC
Padding de celda: 12px 16px
Texto: #1F2937 (valores principales), #6B7280 (secundarios)
```

### 5.5 Inputs y Forms

```
Input base:
  Borde: 1px solid #CBD5E1
  Radius: 6px
  Padding: 8px 12px
  Font-size: 14px
  Fondo: white

Input focus:
  Borde: 2px solid #2D5BE3
  Ring: 0 0 0 3px rgba(45, 91, 227, 0.1)

Input error:
  Borde: 1px solid #DC2626
  Mensaje de error: Texto #DC2626, Font-size 12px, margin-top 4px

Label: Font-size 14px, Font-weight 500, Color #374151, Margin-bottom 4px
Placeholder: Color #9CA3AF
```

### 5.6 Sidebar

```
Ancho: 240px (expandido), 64px (colapsado)
Fondo: #1B3A6B (Pacta Blue)
Texto: rgba(255,255,255,0.8)
Texto activo: white
Item activo fondo: rgba(255,255,255,0.12)
Item hover fondo: rgba(255,255,255,0.07)
Separadores: rgba(255,255,255,0.12)
Logo / Brand: Lora Bold 18px, blanco
```

### 5.7 Navbar Top

```
Altura: 56px
Fondo: white
Borde inferior: 1px solid #E2E8F0
Sombra: 0 1px 0 #E2E8F0
Contenido: Breadcrumb izquierda, Notificaciones + Avatar derecha
```

---

## 6. Layout y Espaciado

### 6.1 Grid del Sistema

```
Layout principal: Sidebar (240px fijo) + Contenido (fluid)
Contenido máximo: 1280px (centrado en pantallas grandes)
Gutter: 24px (padding lateral del área de contenido)
```

### 6.2 Espaciado

```
Tailwind spacing scale (base 4px):
  space-1  = 4px
  space-2  = 8px
  space-3  = 12px
  space-4  = 16px
  space-6  = 24px (gap estándar entre cards)
  space-8  = 32px (separación de secciones)
  space-12 = 48px (separación mayor)
```

### 6.3 Breakpoints (Responsive)

```
Mobile:  < 640px  — Sidebar se oculta, menú hamburguesa
Tablet:  640-1024px — Sidebar colapsado (solo iconos)
Desktop: > 1024px — Sidebar expandido (iconos + texto)
Wide:    > 1280px — Contenido centrado con máx-width
```

---

## 7. Logo y Marca Gráfica

### 7.1 Concepto del Logo

El logo de PACTA combina:
- **Un sello / cierres** — evoca la formalización de un acuerdo, la autenticidad notarial
- **Tipografía "PACTA"** en Lora Bold — autoridad y elegancia

### 7.2 Variaciones del Logo

```
Principal:   Isotipo (sello) + "PACTA" (Lora Bold, #0F1C33)
Solo marca:  "PACTA" tipográfico
Negativo:    Logo blanco sobre Pacta Blue (#1B3A6B)
Favicon:     Isotipo solo, cuadrado, 32x32px
App icon:    Isotipo en círculo Pacta Blue, 512x512px
```

### 7.3 Espaciado de Protección del Logo

El logo debe tener siempre un espacio limpio alrededor equivalente a la altura de la letra "A" del logotipo.

---

## 8. Tono de Voz (UX Writing)

### 8.1 Principios

- **Directo:** "Contrato creado" no "Su contrato ha sido creado exitosamente"
- **Activo:** "Crear contrato" no "Creación de contrato"
- **Específico en errores:** "La fecha de fin debe ser posterior a la fecha de inicio" no "Fecha inválida"
- **Amigable pero profesional:** Sin jerga técnica, sin ser condescendiente

### 8.2 Mensajes Clave

```
Acciones exitosas:
  ✓ "Contrato guardado"
  ✓ "Cliente eliminado"
  ✓ "Suplemento aprobado"

Errores:
  ✗ "El número de contrato ya existe"
  ✗ "Este campo es requerido"
  ✗ "No puedes eliminar un contrato activo"

Confirmaciones destructivas:
  "¿Cancelar este contrato? Esta acción no puede deshacerse."
  [Cancelar contrato] [No, volver]

Estados vacíos:
  Contratos: "No hay contratos todavía. Crea el primero."
  Resultados de búsqueda: "Sin resultados para '{término}'. Intenta con otro filtro."
  Notificaciones: "Todo al día. No hay notificaciones pendientes."
```

---

## 9. Animaciones y Transiciones

```
Duración estándar: 150ms (hover, focus) / 200ms (panel aparece) / 300ms (modal)
Easing: ease-out (aparece), ease-in (desaparece)

Transiciones específicas:
  - Sidebar collapse/expand: 250ms ease-in-out (width)
  - Modal aparece: 200ms ease-out (opacity + scale 0.95 → 1)
  - Toast/Notificación: 300ms ease-out (translateY + opacity)
  - Hover en rows de tabla: 100ms ease background-color
  - Dropdown menu: 150ms ease-out
  - Badge de notificaciones: pulse animation (1.5s, infinite) cuando hay no leídas urgentes
```

---

## 10. Configuración Tailwind

```javascript
// tailwind.config.js
module.exports = {
  content: ['./frontend/app/templates/**/*.html'],
  theme: {
    extend: {
      colors: {
        pacta: {
          50:  '#EEF2FB',
          100: '#D5DFF5',
          200: '#A8BCEC',
          300: '#7A99E3',
          400: '#4D76DA',
          500: '#2D5BE3',  // Pacta Blue Light
          600: '#234AC5',
          700: '#1B3A6B',  // Pacta Blue (primario)
          800: '#162F58',
          900: '#0F1C33',  // Slate Dark
        },
      },
      fontFamily: {
        display: ['Lora', 'Georgia', 'serif'],
        sans:    ['Inter', 'system-ui', 'sans-serif'],
        mono:    ['JetBrains Mono', 'monospace'],
      },
      borderRadius: {
        DEFAULT: '6px',
        lg: '8px',
        xl: '12px',
      },
    },
  },
}
```
