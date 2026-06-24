---
name: Vigía
description: Portal PLD/FT para sujetos obligados (LFPIORPI México). shadcn/ui sobre React+Vite+Tailwind v4; esta DESIGN.md especifica la capa de marca Vigía (sistema de tokens propio de 3 temas, tipografía editorial-legal y semántica de banda de confianza). Tema canónico Toga, vibe Editorial.
status: final
created: 2026-06-24
updated: 2026-06-24
project: PreLavadoDinero
sources:
  - docs/prd_screening_pld.md
  - CONTEXT.md
  - _bmad-output/planning-artifacts/architecture.md
  - _bmad-output/planning-artifacts/ux-designs/ux-PreLavadoDinero-2026-06-24/imports/Vigia.html
  - _bmad-output/planning-artifacts/ux-designs/ux-PreLavadoDinero-2026-06-24/imports/Vigia_contexto_IA.md
colors:
  # Tema canónico = Toga (azul institucional). notario/salvaguarda documentados como variantes en el cuerpo.
  bg: '#eaeff7'
  surface: '#ffffff'
  surface-2: '#f3f7fc'
  ink: '#0f1b2d'
  ink-soft: '#46566e'
  muted: '#8394ad'
  line: '#dde5f0'
  accent: '#2f6df0'
  accent-ink: '#ffffff'
  accent-soft: '#e7eefe'
  hero: '#0f1b2d'
  on-hero: '#d3deef'
  on-hero-soft: '#8ba0c0'
  gold: '#cfa13a'
  # Semántica de banda de confianza y tipo de lista (valores Toga). Cada token tiene su par -bg.
  banda-real: '#c5392c'
  banda-real-bg: '#fae8e5'
  banda-real-text: '#9e271c'        # texto del badge sobre -bg (AA ≥4.5:1); el hue vivo va en dot/borde
  banda-homonimo: '#3a64cf'
  banda-homonimo-bg: '#e8eefb'
  banda-homonimo-text: '#2a4ba3'
  banda-verificar: '#bd7d12'
  banda-verificar-bg: '#faefd6'
  banda-verificar-text: '#7a5200'   # ámbar oscurecido: el #bd7d12 reprobaba (≈3:1) sobre su -bg
  banda-descartado: '#1f8a5b'
  banda-descartado-bg: '#e1f3ea'
  banda-descartado-text: '#156944'
  pep: '#7a52c4'
  pep-bg: '#efe9fa'
  pep-text: '#5b3a9e'
typography:
  display:
    fontFamily: 'Spectral, Georgia, serif'
    fontWeight: '700'
    letterSpacing: -0.02em
    note: 'Titulares, logo, nombres de tarjeta, prosa de dictamen. Vibe Editorial (canónico). En vibe Tecnológico, display = Hanken Grotesk.'
  ui:
    fontFamily: 'Hanken Grotesk, system-ui, sans-serif'
    fontWeight: '400'
    note: 'Cuerpo, botones, nav, labels. Familia base del body. Pesos 400/500/600/700/800.'
  mono:
    fontFamily: 'IBM Plex Mono, ui-monospace, monospace'
    fontWeight: '500'
    fontVariantNumeric: 'tabular-nums'
    note: 'TODO dato: scores, RFC, folios, KPIs, precios, reloj 24h, versiones de lista, fechas técnicas. Pesos 400/500/600. font-variant-numeric: tabular-nums OBLIGATORIO (delta de rendimiento): cifras de ancho fijo para que el reloj que tickea cada segundo, scores y KPIs no salten de ancho ni provoquen micro-CLS.'
  hero-title:
    fontFamily: 'Spectral'
    fontSize: 54px
    fontWeight: '700'
    lineHeight: '1.04'
    letterSpacing: -0.02em
  h2-section:
    fontFamily: 'Spectral'
    fontSize: 36px
    fontWeight: '700'
    letterSpacing: -0.02em
  h1-panel:
    fontFamily: 'Spectral'
    fontSize: 30px
    fontWeight: '700'
    letterSpacing: -0.01em
  card-title:
    fontFamily: 'Spectral'
    fontSize: 19px
    fontWeight: '600'
  body:
    fontFamily: 'Hanken Grotesk'
    fontSize: 16px
    lineHeight: '1.65'
  body-sm:
    fontFamily: 'Hanken Grotesk'
    fontSize: 14px
  eyebrow:
    fontFamily: 'Hanken Grotesk'
    fontSize: 12px
    fontWeight: '800'
    letterSpacing: 0.1em
    note: 'uppercase. Overlines de sección.'
  data-lg:
    fontFamily: 'IBM Plex Mono'
    fontSize: 30px
    fontWeight: '600'
    note: 'Valor de KPI, precio, rango de multa.'
  data:
    fontFamily: 'IBM Plex Mono'
    fontSize: 13.5px
    fontWeight: '600'
    note: 'Score, RFC, reloj en tabla.'
  table-header:
    fontFamily: 'Hanken Grotesk'
    fontSize: 11.5px
    fontWeight: '700'
    letterSpacing: 0.05em
    note: 'uppercase, color muted.'
rounded:
  # Normalizada de los valores inline del prototipo (no había tokens).
  sm: 8px
  md: 11px
  lg: 14px
  xl: 18px
  full: 9999px
spacing:
  # El prototipo no tiene escala de tokens; se normaliza la escala Tailwind 4-based y se fijan tokens nombrados.
  gutter: 7vw
  content-max: 1180px
  panel-max: 980px
  section-y: 80px
  sidebar-w: 248px
components:
  button-primary:
    background: '{colors.accent}'
    foreground: '{colors.accent-ink}'
    radius: '{rounded.md}'
    note: 'padding 12-15px 20-26px; font ui 700 14-15.5px.'
  button-secondary:
    background: '{colors.surface}'
    foreground: '{colors.ink-soft}'
    border: '1px solid {colors.line}'
    radius: '{rounded.md}'
  button-aviso:
    background: '{colors.banda-real}'
    foreground: '#ffffff'
    radius: '{rounded.md}'
    note: 'Acción de banda crítica: "Generar aviso XML (SPPLD)". width 100%.'
  badge-banda:
    radius: '{rounded.sm}'
    note: 'inline-flex con punto ● (hue {colors.banda-*}), fondo {colors.banda-*-bg}, texto {colors.banda-*-text} (≥4.5:1). 700 12.5px. Bloqueo-UIF invertido: fondo banda-real sólido, texto #fff.'
  dictamen-card:
    background: '{colors.surface}'
    radius: '{rounded.lg}'
    border: '1px solid {colors.line}'
  kpi-card:
    background: '{colors.surface}'
    border: '1px solid {colors.line}'
    radius: '{rounded.lg}'
    note: 'valor en {typography.data-lg}; color ink, o banda-real para reales/avisos.'
  table-dictamenes:
    background: '{colors.surface}'
    radius: '{rounded.lg}'
    note: 'grid 2fr 1.1fr .6fr 1.1fr .8fr; header surface-2; fila hover surface-2.'
  reloj-aviso:
    accent: '{colors.banda-real}'
    note: 'dot pulsante; contador en mono 600 (18px lista / 26px detalle) con tabular-nums OBLIGATORIO (no debe saltar de ancho al tickear); color urgencia <6h banda-real, <12h banda-verificar, else ink.'
  sidebar:
    background: '{colors.hero}'
    foreground: '{colors.on-hero}'
    note: 'fijo 248px; item activo color-mix(#fff 12%) + texto #fff.'
  dropzone:
    border: '2px dashed {colors.line}'
    radius: '{rounded.xl}'
    background: '{colors.surface}'
---

# Vigía — Design Spine

> Identidad visual del portal Vigía (PLD/FT, LFPIORPI). Capa de marca sobre shadcn/ui + Tailwind v4. Tema canónico **Toga**, vibe **Editorial**. Pareada con `EXPERIENCE.md`. Las spines ganan sobre el prototipo `imports/Vigia.html` en conflicto.

## Brand & Style

**Vigía** = "el que vigila desde lo alto": vigilancia + protección. Es una herramienta de cumplimiento legal con multas millonarias de fondo, así que la postura visual es **despacho legal moderno, no acartonado**: gravitas sin polvo. La marca tiene dos caras conectadas por un flujo: la **landing pública** (venta, urgencia factual de la multa y la reforma 2026) y el **panel privado** (trabajo documental, que prioriza claridad y usabilidad por encima del drama).

El recurso de marca central es la **semántica de banda de confianza color-codificada** (rojo=real, azul=homónimo, ámbar=verificar, verde=descartado, morado=PEP). Es funcional antes que decorativa: el color *es* el dictamen de un vistazo. Logo: cuadro redondeado con **"V" en Spectral serif** sobre el acento. Badge `PLD/FT` (acrónimo correcto; nunca "DLP"/"PLD" mal escrito).

Vigía hereda shadcn/ui como capa de primitivos de componente, pero la capa de marca es sustancial: sistema de tokens propio de tres temas, tipografía editorial y la semántica de banda. shadcn aporta el comportamiento accesible de los primitivos (Dialog, Sheet, Popover, Toast, Table, Tabs, Command); Vigía los reviste con sus tokens.

## Colors

Sistema de **tres temas** con los mismos nombres de token; **Toga es el canónico** (frontmatter). Los otros dos son variantes seleccionables (el prototipo conserva un switcher flotante):

- **Toga (canónico)** — azul institucional `accent #2f6df0` sobre claro frío. Lee legal-tech serio y confiable.
- **Notario** — papel cálido `bg #efe9dc` + oxblood `accent #9c3a2a` + `gold #a9802a`. Variante tradicional/notarial.
- **Salvaguarda** — verde `accent #0b6b43` + dorado `#c2992f` sobre verde claro. Variante "seguridad/protección".

Reglas de uso del tema canónico:
- **`accent` (azul)** — CTAs primarios, nav activo, enlaces, foco. No para estado ni decoración.
- **`hero` (azul tinta `#0f1b2d`)** — fondos oscuros: hero de landing, sidebar del panel, footer. Texto sobre él usa `on-hero`/`on-hero-soft`.
- **`gold`** — acentos editoriales puntuales (eyebrows, cifras destacadas en tono Equilibrado/Sobrio). Nunca como color de acción.

**Semántica de banda de confianza (el token más importante, y NO decorativo):**
- `banda-real` rojo → `resuelto-real` (coincidencia confirmada).
- `banda-homonimo` azul → `resuelto-homónimo` (mismo nombre, otra persona).
- `banda-verificar` ámbar → `requiere-verificación` (falta dato).
- `banda-descartado` verde → `descartado-por-nombre` (limpio).
- `pep` morado → tipo de lista PEP (recordatorio visual: PEP dispara EDD, **no** rechazo).
- Tipo de lista **Bloqueo-UIF** es el único badge invertido (fondo `banda-real` sólido + texto `#fff`): es la consecuencia más severa.

Cada token de banda tiene tres roles separados para garantizar contraste (corrige el hallazgo de accesibilidad: el ámbar y el rojo reprobaban como texto sobre su `-bg`):
- **Hue vivo** (`banda-*`) → punto ●, `border-left 4px`, barras — elementos no textuales (umbral 3:1 UI).
- **Fondo tintado** (`banda-*-bg`) → relleno del badge/tarjeta.
- **Texto** (`banda-*-text`, oscurecido) → la etiqueta del badge sobre `-bg`, garantizado **≥4.5:1**. Nunca usar `banda-*` (el hue vivo) como color de texto sobre `-bg`.

**Gate de contraste (regla dura, no aspiración):** las **15 combinaciones banda×tema** (5 bandas × 3 temas) de `banda-*-text` sobre `banda-*-bg` se verifican ≥4.5:1 antes de release; igual `on-hero`/`on-hero-soft` sobre `hero` y `ink`/`ink-soft` sobre `surface`/`bg`. Los valores `-text` del frontmatter son para Toga; notario/salvaguarda derivan su `-text` con el mismo gate. **`muted` (#8394ad) NO es color de texto de cuerpo ni de dato informativo** (reprueba AA): solo para sub-labels decorativos de tamaño grande o sobre alto contraste; los encabezados de tabla usan `ink-soft`, no `muted`.

**Dark mode:** no en fase 1; los tres temas son claros. Si se añade, se hará con tokens `-dark` separados, no invirtiendo a mano.

Evitar: usar el rojo de banda como color de error genérico de UI (es semántica de dictamen); usar `banda-*` como texto sobre `-bg` (usar `banda-*-text`); introducir colores fuera del sistema de tokens; gradientes decorativos (el único gradiente permitido es el overlay radial del hero).

## Typography

Tres familias con roles estrictos:
- **Spectral (serif)** → `display`: titulares, logo, nombres de tarjeta, prosa del dictamen. Da la gravitas legal. (Vibe canónico Editorial. En vibe Tecnológico, `display` cambia a Hanken Grotesk — única diferencia del vibe.)
- **Hanken Grotesk (sans)** → `ui`/`body`: todo lo operativo, legible.
- **IBM Plex Mono** → **todo dato**: scores, RFC, folios, KPIs, precios, el reloj 24h, versiones de lista. Regla dura: un número que importa va en mono.

Ramp (ver tokens): hero 54 · h2 sección 36 · h1 panel 30 · card-title 19 · body 16 · body-sm 14 · eyebrow 12 (uppercase, `ls .1em`, 800) · data-lg 30 · data 13.5 · table-header 11.5 (uppercase). Titulares con `letter-spacing` negativo (−.01 a −.02em); eyebrows/overlines con `ls` positivo y uppercase. La cursiva Spectral marca conceptos clave ("*homónimo*", "*tocayo*").

**Cifras tabulares (delta de rendimiento, regla dura):** todo lo que va en `mono` usa `font-variant-numeric: tabular-nums`. El motivo es load-bearing en este producto: el reloj de aviso tickea cada segundo y los scores/KPIs se repueblan al cargar; con cifras proporcionales el contador cambia de ancho en cada tick y produce jitter/micro-CLS justo en el elemento de mayor consecuencia legal. Las cifras de ancho fijo lo eliminan sin coste visual.

**Carga de fuentes (delta de rendimiento):** tres familias (Spectral, Hanken Grotesk en 400/500/600/700/800, IBM Plex Mono en 400/500/600) son mucho peso. Reglas: `font-display: swap` en todas (evita FOIT — texto invisible mientras carga); `preload` **solo** de las críticas above-the-fold (Spectral del hero/titular + Hanken Grotesk 400); el resto de pesos cargan sin preload. Subsetear a `latin`/`latin-ext` (es-MX no necesita más). El prototipo no declaraba ninguna estrategia → es deuda a cerrar como el `outline:none`.

## Layout & Spacing

El prototipo no tenía escala de tokens (todo inline); aquí se **normaliza** a la escala Tailwind 4-based (4/8/12/16/20/24/32/40/48/64) más tokens nombrados: `gutter 7vw` (padding horizontal de sección en landing), `content-max 1180px`, `panel-max 980px`, `section-y 80px`, `sidebar-w 248px`. Contenedor de landing centrado a `content-max`; vistas de panel densas (upload, detalle) a `panel-max`.

Grid: landing por secciones full-bleed con contenido centrado; panel = `grid 248px 1fr` (sidebar fijo + main con scroll propio). Breakpoints y colapsos: ver `EXPERIENCE.md` → Responsive & Platform (el responsive se inventa; el prototipo era desktop-only).

## Elevation & Depth

Sin tokens de sombra en el prototipo; se normaliza una escala de 5 niveles, color base `rgba(15,27,45,·)` (tono `ink` de Toga):
- nivel-0 plano (la mayoría de superficies, solo `border 1px line`).
- nivel-1 `0 2px 14px /.04` (tiers no destacados).
- nivel-2 `0 4px 24px /.05` (tarjeta de verificador).
- nivel-3 `0 20px 50px /.10` (tier destacado, popovers).
- nivel-4 `0 26px 60px rgba(0,0,0,.34)` (tarjeta de dictamen sobre el hero oscuro — única que usa negro puro por contraste).

La elevación **no** es el dispositivo de jerarquía principal: el borde `1px line` y el color de superficie (`surface` vs `surface-2`) hacen la mayor parte. Sombra solo para flotantes reales.

## Shapes

Radios normalizados: `sm 8` (badges, botones chicos, score box) · `md 11` (botones, inputs, filas de aviso) · `lg 14` (tarjetas, paneles, KPIs) · `xl 18` (tarjetas grandes: lead magnets, pricing) · `full 9999` (píldoras: chips, filtros, badges de hero, switcher de tema). El acento de banda se expresa además como **`border-left: 4px` del color de banda** en el resultado del verificador y las filas del reloj. Dropzone usa `2px dashed line`.

## Components

shadcn aporta los primitivos accesibles; Vigía especifica el revestimiento. Specs clave (ver tokens `components`):

- **Button** — primario (`accent`/`accent-ink`, `rounded.md`), secundario (`surface`+`border line`), ghost-sobre-hero (`color-mix #fff 10%` fondo + borde `#fff 32%`), y **aviso** (`banda-real` sólido, full-width, "Generar aviso XML (SPPLD)").
- **Badge de banda** — inline-flex con punto ●, `banda-*-bg`/`banda-*`, `rounded.sm`; agranda en el detalle del dictamen. Tipo de lista en `rounded.sm` 12px; Bloqueo-UIF invertido.
- **Tarjeta de dictamen** — `surface`, `rounded.lg`, header con título display + folio mono; nombre 700 17px; badge de banda; mini-cajas Lista/Score en `surface-2`.
- **KPI card** — valor en `data-lg`, color dinámico (ink, o `banda-real` para reales/avisos).
- **Tabla de dictámenes** — grid `2fr 1.1fr .6fr 1.1fr .8fr`; header `surface-2` uppercase; fila clic→detalle, hover `surface-2`; score con color por umbral (≥.85 rojo, ≥.5 ámbar).
- **Reloj de aviso 24h** — dot pulsante (`banda-real`, `pulse 1.5s`), contador mono con color de urgencia (<6h rojo, <12h ámbar); dos tamaños (lista 18 / detalle 26).
- **Sidebar** — `hero` de fondo, item activo `color-mix #fff 12%` + texto `#fff`; logo "V"; bloque de sujeto obligado activo (selector multi-cliente, ver EXPERIENCE.md).
- **Dropzone** — `2px dashed line`, `rounded.xl`, icono ⬆ + stepper de 3 pasos arriba.
- **Inputs** — `surface-2` + `border line`, `rounded.md`; **falta focus visible en el prototipo → DESIGN delta obligatorio: anillo de foco visible en TODO interactivo.** El anillo **se adapta a la superficie**: `2px accent` sobre superficies claras; sobre el sidebar/hero oscuro (donde `accent` azul desaparece) usa anillo claro `2px on-hero`/`#fff`. Verificado ≥3:1 contra su fondo (WCAG 2.4.11).

## Do's and Don'ts

| Do | Don't |
|---|---|
| Color de banda = significado (rojo/azul/ámbar/verde/morado) | Usar el rojo de banda como error genérico de UI |
| Todo dato numérico en IBM Plex Mono | Poner scores/RFC/reloj en sans |
| Spectral serif para titulares/prosa (vibe Editorial) | Titular operativo largo en serif que dañe legibilidad |
| Heredar primitivos shadcn (Dialog, Sheet, Table, Command) | Reescribir primitivos accesibles desde cero |
| Mantener foco visible (anillo `accent`) en todo interactivo | Dejar `outline:none` sin reemplazo (deuda del prototipo) |
| Toga como cara por defecto; notario/salvaguarda como variantes | Mezclar tokens de dos temas en una vista |
| PEP en morado, recordando que dispara EDD | Pintar PEP como "rechazo" (rojo) |
| Disclaimers "herramienta de apoyo, no garantía" presentes | Prometer inmunidad ante multas en el copy |
