# Validation Report — Vigía (UX Spines)

- **DESIGN.md:** `_bmad-output/planning-artifacts/ux-designs/ux-PreLavadoDinero-2026-06-24/DESIGN.md`
- **EXPERIENCE.md:** `…/EXPERIENCE.md`
- **Run at:** 2026-06-24T18:36:37Z
- **Reviewers:** rubric walker + accesibilidad (WCAG 2.2 AA)
- **Estado:** todos los hallazgos critical y high **resueltos en las spines** antes de cerrar (`status: final`).

## Overall verdict

El par de spines es genuinamente source-extraíble: el token de banda de confianza (recurso de marca central) está completo, hexeado y disciplinado; la herencia resuelve; los nombres de componente son consistentes; las secciones están en orden canónico; y las decisiones de dominio (matriz banda×tipo_lista, fidelidad prosa↔verdicto, aislamiento por sujeto obligado, módulos bloqueados visibles) trazan a fuente. La revisión de accesibilidad encontró que la paleta de banda **no era AA** como texto (ámbar y rojo reprobaban contraste) y faltaban deltas de movimiento, foco, aria-live y semántica de tabla móvil. **Todo ello quedó resuelto** en esta misma pasada: se añadieron tokens `banda-*-text` oscurecidos con gate de contraste ≥4.5:1, anillo de foco adaptativo, `prefers-reduced-motion`, `aria-live` acotado al cruce de umbral, refuerzo textual de score/banda, semántica de tarjeta en móvil, `lang="es"` y objetivos táctiles; y se agregó el Key Flow del aviso 24h que faltaba.

## Category verdicts (rúbrica)

- Flow coverage — thin → **resuelto** (agregado Flow 3: aviso 24h)
- Token completeness — adequate → **mejorado** (tokens `-text` + gate de contraste)
- Component coverage — strong
- State coverage — adequate
- Visual reference coverage — strong
- Bloat & overspecification — strong
- Inheritance discipline — adequate → **mejorado** (paths de `sources` normalizados a raíz)
- Shape fit — strong
- Performance floor — ausente → **agregado** (pasada UI/UX 2026-06-24): las revisiones rúbrica + accesibilidad no cubrían rendimiento (categoría HIGH). Deltas obligatorios añadidos con el mismo rango de "regla dura" que foco/contraste: `tabular-nums` en todo dato vivo/tabular (CLS del reloj que tickea), virtualización de tablas >50 filas (NFR-2, 10,000 sujetos), reserva de espacio en contenido asíncrono (skeletons dimensionados, CLS < 0.1), estrategia de fuentes (`font-display: swap` + preload selectivo, 3 familias), latencia de tap <100ms y code-splitting landing/panel. Visual en `DESIGN.md` Typography + token `mono`/`reloj-aviso`; comportamiento en `EXPERIENCE.md` §Performance Floor.

## Findings by severity

### Critical (3) — accesibilidad — RESUELTOS
**[Accesibilidad 1.4.3]** — `banda-verificar` ámbar reprueba contraste como texto en los 3 temas (Toga 3.01:1, Salvaguarda 2.79:1).
Fix aplicado: token `banda-verificar-text #7a5200` (oscurecido) para el texto del badge; el hue vivo solo en dot/borde; gate ≥4.5:1.

**[Accesibilidad 1.4.3]** — `banda-real` rojo reprueba en Toga (4.44:1).
Fix aplicado: token `banda-real-text #9e271c`; mismo patrón.

**[Accesibilidad 1.4.3 / proceso]** — sin gate de contraste; 15 combinaciones banda×tema sin medir.
Fix aplicado: gate de contraste como **regla dura** en DESIGN.md (las 15 combinaciones `-text`/`-bg` + on-hero + ink se verifican ≥4.5:1 antes de release).

### High (resueltos)
**[Rúbrica · Flow]** — ningún Key Flow cubría el aviso 24h de punta a punta. → **Flow 3 agregado** (coincidencia → reloj → aviso/acuse → expediente, con ruta de vencimiento).
**[Rúbrica · Inheritance]** — paths de `sources` mezclaban base raíz y workspace. → **normalizados** a raíz del repo en ambas spines.
**[Accesibilidad 2.3.3]** — animaciones sin `prefers-reduced-motion`. → **regla agregada** (pulso/scanline/spinner respetan reduce).
**[Accesibilidad 4.1.3]** — `aria-live` del reloj subespecificado (riesgo de narrar el tick). → contador `aria-hidden` + región `polite` solo para cruces de umbral.
**[Accesibilidad 2.4.7/2.4.11]** — foco azul invisible sobre sidebar oscuro. → **anillo adaptativo** (claro sobre hero).
**[Accesibilidad 1.4.1]** — score por color de umbral sin texto. → score muestra valor numérico + etiqueta de riesgo.
**[Accesibilidad 1.3.1]** — tabla→tarjetas en móvil podía perder asociación. → cada campo conserva etiqueta visible.

### Medium / Low (resueltos o registrados)
`muted` no es color de texto de cuerpo (nota agregada; encabezados usan `ink-soft`) · `lang="es"` · objetivos táctiles ≥24–44px · datos mono con `aria-label` · teclado en filas/chips custom · foco visible en todo interactivo. Los hallazgos menores de bloat/forma no requirieron cambio.

## Reviewer files
- `review-rubric.md`
- `review-accesibilidad.md`
