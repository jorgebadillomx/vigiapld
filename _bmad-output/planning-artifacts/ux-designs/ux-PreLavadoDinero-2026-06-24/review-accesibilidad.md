# Reseña de accesibilidad — Vigía (WCAG 2.2 AA)

> Revisión adversarial de las spines `DESIGN.md` + `EXPERIENCE.md` (tema canónico Toga, vibe Editorial) contra el prototipo `imports/Vigia.html` y su contexto `imports/Vigia_contexto_IA.md`. Foco: producto regulado PLD/FT con banda de confianza color-codificada.

---

## Veredicto

**Las spines tienen un piso de accesibilidad serio y bien intencionado (banda nunca solo por color, foco visible declarado, semántica shadcn), pero NO es conforme AA todavía: la paleta de banda falla contraste de texto en los tres temas — el ámbar `banda-verificar` es el peor (3.01:1 Toga, 2.79:1 Salvaguarda) — y faltan deltas concretos para `aria-live` del reloj, `prefers-reduced-motion`, objetivos táctiles y semántica de tabla→tarjeta en móvil.**

---

## Resumen de severidad

| Sev | # | Tema dominante |
|---|---|---|
| Critical | 3 | Contraste de banda (1.4.3) en los 3 temas |
| High | 5 | Movimiento, reloj live, foco real, color en score, tabla móvil |
| Medium | 5 | muted, táctil, lang, mono/lector, Esc/teclado |
| Low | 3 | descartado verde, focus sobre hero, switcher de tema |

---

# YA CUBIERTO POR LAS SPINES (crédito donde corresponde)

Esto el prototipo NO lo tenía y las spines lo añaden correctamente como delta:

- **Banda nunca solo por color.** `EXPERIENCE.md` → Accessibility Floor + Interaction Primitives + tabla Component Patterns ("Badge de banda … siempre con etiqueta textual (no solo color — a11y)") y `DESIGN.md` → badge-banda usa **punto ● + etiqueta + fondo + texto**. Redundancia forma+texto+color: daltonismo-seguro en el badge. Bien.
- **Foco visible como delta explícito.** `DESIGN.md` → Inputs: *"falta focus visible en el prototipo → DESIGN delta obligatorio: anillo de foco 2px accent (ring shadcn)"*; reforzado en Do's/Don'ts y `EXPERIENCE.md` → Interaction Primitives ("Foco visible siempre — corrige la deuda del prototipo `outline:none`"). El prototipo en efecto trae `outline:none` (2 ocurrencias confirmadas, sin `:focus` ni `:focus-visible` declarados, 0 `tabindex`).
- **Primitivos accesibles heredados.** Dialog/Sheet/Popover/Toast/Table/Tabs/Command de shadcn aportan foco-trap, `Esc`, roles ARIA y teclado de fábrica. Es la decisión correcta vs. reescribir.
- **Intención de tabla semántica.** Floor pide "encabezados asociados" y anuncio "Dictámenes, {N} sujetos, filtro {banda}".
- **Reloj con `aria-live` cortés** mencionado: "anuncia cruces de umbral (<12h, <6h), no cada segundo".
- **`Tab` = orden de lectura; `Esc` cierra el modal/popover superior**; stacks de modal limitados a 1 nivel.
- **Mono con piso de tamaño:** "Todo dato mono mantiene tamaño legible (≥13px) y contraste AA".
- **Disclaimers permanentes** (no es a11y WCAG pero sí claridad cognitiva en producto de riesgo legal).

El problema es que varios de estos son **enunciados de intención sin el valor que los haga verdaderos** (sobre todo contraste y `aria-live`), y otros (movimiento, táctil, lang, mono) **no se mencionan en absoluto**.

---

# HUECOS A CORREGIR

## CRITICAL

### C1 — Banda `verificar` (ámbar) NO cumple contraste de texto en NINGÚN tema
- **WCAG:** 1.4.3 Contraste mínimo (AA, texto normal 4.5:1).
- **Ubicación:** `DESIGN.md` frontmatter `banda-verificar #bd7d12` sobre `banda-verificar-bg #faefd6`; badge-banda (texto 700 12.5px = texto normal, no "grande"); score ámbar de tabla; estado `requiere-verificación` del detalle. El usuario lo marcó como "el más riesgoso" — correcto.
- **Problema (ratios calculados):**
  - Toga `#bd7d12` / `#faefd6` = **3.01:1** ❌
  - Notario `#956a16` / `#f0e5cc` = **3.86:1** ❌
  - Salvaguarda `#b3851f` / `#f4ead0` = **2.79:1** ❌ (el peor de toda la paleta)
  - El badge es texto 12.5px peso 700 → **texto normal** bajo WCAG (el umbral 3:1 de "texto grande" exige ≥18.66px@700 o ≥24px). No califica; el piso es 4.5:1. Incluso si se tratara como grande, Salvaguarda 2.79 también reprueba 3:1.
- **Fix:** oscurecer el texto de banda ámbar a ~`#8a5a00`/`#7c5200` (mantiene la lectura cálida pero llega a ≥4.5:1 sobre el `-bg` crema) **o** oscurecer-saturar el fondo. Recalcular en los 3 temas. Regla de token: el par `banda-* / banda-*-bg` debe pasar 4.5:1 por definición, verificado en CI, no a ojo. El propio Floor ya señala "verificar AA … especialmente ámbar banda-verificar sobre su -bg" — esto confirma la sospecha con número.

### C2 — Banda `real` (rojo) reprueba en Toga; queda al filo en los demás
- **WCAG:** 1.4.3 (4.5:1).
- **Ubicación:** badge "Real", KPI rojo, score ≥.85 rojo. `banda-real #c5392c` / `banda-real-bg #fae8e5`.
- **Problema:** Toga = **4.44:1** ❌ (debajo de 4.5 — falla, aunque por poco). Notario 5.16 ✅, Salvaguarda 4.66 ✅. Como es la banda más importante (coincidencia confirmada, dispara aviso), no puede ir al filo.
- **Fix:** llevar el rojo Toga a `#b8342a` o más oscuro hasta ≥4.5:1; verificar que el rojo invertido de Bloqueo-UIF (texto `#fff` sobre `banda-real` sólido) mantenga ≥4.5 (hoy `#fff`/`#c5392c` = 5.26:1 ✅, ok).

### C3 — La conformidad de contraste se enuncia pero no se garantiza por token
- **WCAG:** 1.4.3 / 1.4.11 (UI 3:1), como defecto de proceso.
- **Ubicación:** `DESIGN.md` afirma "Cada token de banda … mantiene contraste sobre cada fondo"; el Floor lo delega a "verificar AA en los 3 temas". Es una promesa, no un hecho: C1/C2 (y L1) la rompen hoy.
- **Problema:** sin un gate, cada re-tinte por tema reintroduce fallas. Tres temas × ~6 pares de banda = 18 combinaciones que nadie está midiendo.
- **Fix:** añadir a la spine un **contrato de token**: todo par `fg/bg` semántico debe pasar 4.5:1 (texto) o 3:1 (borde/UI), validado automáticamente (p.ej. script de contraste en build) y documentado en la tabla de tokens. Sin esto, C1/C2 reaparecen.

---

## HIGH

### H1 — Movimiento sin `prefers-reduced-motion`
- **WCAG:** 2.3.3 Animación por interacción (AAA, recomendado) y 2.2.2 Pausar/Detener/Ocultar (AA) para el contenido en movimiento automático.
- **Ubicación:** prototipo confirma keyframes `pulse 1.5s infinite` (dot del reloj), `scanline 1.1s linear infinite` (verificador en vivo), `spin .8s linear infinite` (procesando), `fadeUp`, `grow`. **0 reglas `prefers-reduced-motion`** en el prototipo, y las spines **no lo mencionan en ninguna parte.**
- **Problema:** el dot pulsante del reloj de aviso es movimiento **persistente, automático, >5s** (vive mientras corre la cuenta de 24h) → cae directo en 2.2.2; scanline y spinner también. Usuario con sensibilidad vestibular sin escape.
- **Fix:** añadir al Floor un delta `@media (prefers-reduced-motion: reduce)` que desactive `pulse`/`scanline`/`spin`/`fadeUp` (o los reduzca a transición instantánea/estática). La urgencia del reloj debe comunicarse por **color + texto del contador**, no por la animación, para que reduzca sin perder significado.

### H2 — Reloj en vivo: el `aria-live` está enunciado pero subespecificado (riesgo de spam)
- **WCAG:** 4.1.3 Mensajes de estado (AA).
- **Ubicación:** `EXPERIENCE.md` Component Patterns ("tick 1s") + Floor ("aria-live cortés que anuncia cruces de umbral, no cada segundo").
- **Problema:** el tick visual es cada 1s. Si el `<time>` del contador es la **misma** región `aria-live`, el lector leerá el cambio **cada segundo** ("18 41 … 18 40 …"), inutilizable. La intención es correcta pero falta el patrón concreto: el contador que tickea debe ser `aria-hidden`/`aria-live="off"`, y debe existir una región `aria-live="polite"` **separada** que solo se actualice al cruzar umbral ("Reloj de aviso: menos de 6 horas restantes"). Además `polite` puede acumularse si hay varios relojes a la vez.
- **Fix:** especificar en el Floor: (1) nodo visual del contador `aria-hidden="true"` + `<time datetime>` con texto accesible redondeado ("aprox. 18 horas"); (2) región `role="status"`/`aria-live="polite"` única que anuncia solo cruces <12h y <6h; (3) si hay N relojes, anunciar un resumen, no N mensajes.

### H3 — "Foco visible siempre" no define el contraste/grosor del anillo
- **WCAG:** 2.4.7 Foco visible (AA) **y 2.4.11 Apariencia del foco** (nuevo en 2.2, AA).
- **Ubicación:** `DESIGN.md` Inputs ("anillo 2px accent"); Floor ("foco visible en todo interactivo").
- **Problema:** el delta solo cubre **inputs**; debe cubrir botones, filas de tabla clicables, chips de filtro, badges-enlace, ítems de sidebar, dropzone, switcher. Y `accent #2f6df0` sobre `surface #fff` = **4.60:1** (≥3:1, ok como indicador), **pero** 2.4.11 exige que el indicador de foco tenga ≥3:1 contra **ambos** el estado sin-foco y el fondo adyacente — un anillo azul de 2px sobre fondos azulados (`accent-soft`, hero azul oscuro del sidebar) puede no separarse. El anillo `accent` azul es además **invisible sobre el sidebar `hero #0f1b2d`** (azul sobre azul tinta).
- **Fix:** subir el anillo a 2.4.11 (área mínima, contraste contra adyacentes), grosor ≥2px + offset, y un **anillo alternativo claro** (p.ej. blanco/`on-hero`) sobre superficies oscuras (sidebar, hero, botón `aviso` rojo). Declararlo para TODO interactivo, no solo inputs.

### H4 — Score en tabla codificado por color de umbral, sin refuerzo no-cromático
- **WCAG:** 1.4.1 Uso del color (A).
- **Ubicación:** `DESIGN.md` table-dictamenes / reloj ("score con color por umbral: ≥.85 rojo, ≥.5 ámbar") y KPI "color dinámico".
- **Problema:** la regla "banda nunca solo por color" está bien blindada para el **badge**, pero el **score numérico** y el **valor de KPI** cambian de color (rojo/ámbar) como señal de severidad **sin** etiqueta ni forma que lo acompañe. Eso es color como único portador para ese dato. El número en sí (0.92) es información, pero "rojo = urgente" no se transmite sin ver el color.
- **Fix:** el score siempre viaja junto al badge de banda (que sí tiene texto+forma) en la misma fila → si esa adyacencia está garantizada en todos los tamaños, el riesgo baja; documentarlo explícitamente. Para el KPI rojo (reales/avisos), añadir etiqueta/ícono ("5 reales") y no depender del rojo. Confirmar que el ámbar del score cumple C1.

### H5 — Tabla → tarjetas en móvil: la semántica de tabla puede perderse
- **WCAG:** 1.3.1 Información y relaciones (A); 1.3.2 Secuencia significativa.
- **Ubicación:** `EXPERIENCE.md` Responsive ("`< md`: tabla de dictámenes → tarjetas (nombre + banda + score)").
- **Problema:** en desktop el Floor promete encabezados asociados y anuncio del total. Al colapsar a tarjetas, si se renderiza con `<div>` se **pierde** la asociación encabezado↔celda y el anuncio "columna Score: 0.92". Además el resumen "Dictámenes, {N} sujetos" debe seguir existiendo en la vista de tarjetas.
- **Fix:** especificar que las tarjetas mantengan etiquetas visibles o `aria-label`/`<dl>` por campo ("Banda: Requiere verificación. Score: 0.71."), conserven el `role`/anuncio de conteo, y que cada tarjeta sea un único objetivo de activación con nombre accesible (el del sujeto). Decidir explícitamente si es lista semántica (`<ul>`/`role="list"`) o tabla responsive.

---

## MEDIUM

### M1 — `muted` reprueba contraste de texto en (casi) todos los temas
- **WCAG:** 1.4.3 (4.5:1 texto) — aceptable solo si es texto desactivado o decorativo (exento).
- **Ubicación:** `muted #8394ad` para table-header (11.5px uppercase), metadatos, placeholders, eyebrows secundarios.
- **Problema:** Toga `muted`/`surface` = **3.09:1**, /`bg` = **2.67:1**, /`surface-2` = **2.87:1** — todos ❌ para texto normal. Notario 3.56 ❌, Salvaguarda 4.00 ❌ (al filo). Los **encabezados de tabla** (`table-header`, color muted) son contenido informativo, no decorativo → deben cumplir. A 11.5px tampoco califican como texto grande.
- **Fix:** usar `ink-soft` (`#46566e`/`surface` = **7.46:1** ✅) para encabezados de tabla y cualquier texto muted que sea informativo. Reservar `muted` solo para elementos genuinamente decorativos/deshabilitados. Reverificar en Notario/Salvaguarda.

### M2 — Objetivos táctiles sin tamaño mínimo especificado
- **WCAG:** 2.5.8 Tamaño del objetivo (mínimo) (AA, nuevo en 2.2 → 24×24px).
- **Ubicación:** móvil `sm`: chips de filtro (píldoras `rounded.full`), badges, ícono de drawer en top bar, "Ver el dictamen completo →", switcher de tema flotante, controles de mapeo de columnas. El prototipo usa botones chicos con padding inline; las spines no fijan tamaño táctil.
- **Problema:** el usuario pidió ≥24–44px. 2.5.8 exige 24×24 (o spacing equivalente); 44×44 es la mejor práctica (Apple/2.5.5 AAA). Chips densos y badges-enlace fácilmente caen por debajo.
- **Fix:** añadir al Floor/Responsive: objetivos interactivos ≥24×24 CSS px (meta 44×44 en móvil para acciones primarias), con separación suficiente entre chips de filtro adyacentes. La fila/tarjeta completa como objetivo (ya es patrón) ayuda; documentarlo.

### M3 — `lang` del documento no declarado en las spines
- **WCAG:** 3.1.1 Idioma de la página (A); 3.1.2 Idioma de las partes (AA).
- **Ubicación:** todo el contenido es español de México; el prototipo NO tiene `lang=` en `<html>` (confirmado: 0 ocurrencias). Las spines no lo mencionan.
- **Problema:** sin `<html lang="es">` (o `es-MX`), el lector de pantalla puede leer el español con fonética inglesa. Términos extranjeros embebidos (OFAC, nombres de lista en inglés) idealmente `lang="en"`.
- **Fix:** delta de Floor: `<html lang="es">` (preferible `es-MX`); marcar fragmentos en otro idioma con `lang`. Trivial pero obligatorio (nivel A).

### M4 — Datos en mono (RFC, score, folio, reloj) sin tratamiento para lector de pantalla
- **WCAG:** 1.3.1 (A); 1.4.4 Redimensionar texto (AA).
- **Ubicación:** `data` 13.5px, `data-lg` 30px; RFC, CURP, folios, scores, reloj, versiones de lista. La regla "todo dato en IBM Plex Mono" es visual; no dice cómo se anuncia.
- **Problema:** (1) un RFC leído carácter-a-carácter o como palabra es ininteligible si no se etiqueta; un score "0.92" debería anunciarse con contexto ("Score 0.92 de 1"). (2) "≥13px" del Floor es un **mínimo en px** — debe permitir zoom de texto al 200% (1.4.4) sin recorte; mono a 13.5px en celdas estrechas es frágil al zoom. (3) Confirmar que `data` 13.5px cumple su propia promesa de "contraste AA" — si va en color muted/ámbar, hereda C1/M1.
- **Fix:** etiquetar datos con `aria-label` contextual (RFC, folio, score sobre 1), no depender solo de la columna; permitir reflow/zoom 200% (1.4.10/1.4.4); asegurar que ningún dato mono use color que reprueba contraste.

### M5 — `Esc` y teclado: enunciado para modales, no para popovers/Command/drawer/dropzone
- **WCAG:** 2.1.1 Teclado (A); 2.1.2 Sin trampa de teclado (A).
- **Ubicación:** Floor ("Esc cierra el modal/popover superior"); prototipo usa `onclick`/`onClick` (88) y `cursor:pointer` (48) — varios elementos clicables que pueden no ser `<button>` nativos (filas, chips, switcher); 0 `tabindex`.
- **Problema:** la fila de dictamen "toda la fila es objetivo de clic" debe ser activable por teclado (Enter/Space) y enfocable — si se implementa como `<div onclick>` sin `role="button"`+`tabindex`, es inalcanzable por teclado. El selector de sujeto obligado (Command), el switcher de tema flotante, los chips de ejemplo del verificador y el drawer `Sheet` necesitan operación de teclado y `Esc` documentada. shadcn lo da si se usan sus primitivos; el riesgo es el markup custom de fila/chip.
- **Fix:** regla en Floor: todo objetivo de clic = elemento nativo interactivo (o `role`+`tabindex`+manejo Enter/Space); `Esc` cierra Sheet/Popover/Command/Dialog; foco se devuelve al disparador al cerrar (2.4.3 Orden del foco). Verificar la fila clicable y el switcher flotante en particular.

---

## LOW

### L1 — Banda `descartado` (verde) reprueba contraste de texto en Toga
- **WCAG:** 1.4.3 (4.5:1).
- **Ubicación:** badge "Descartado", estado "Sin hits" verde. `banda-descartado #1f8a5b`/`#e1f3ea`.
- **Problema:** Toga = **3.76:1** ❌. Notario 4.27 ✅(filo), Salvaguarda 4.27 ✅(filo). Es la banda de menor consecuencia (limpio) pero el texto del badge igual debe leerse.
- **Fix:** oscurecer el verde Toga a ≥4.5:1 (`#1a7a50` aprox.); incluir en el gate de C3. Low solo porque su severidad de dominio es baja, no porque cumpla.

### L2 — Foco con anillo `accent` invisible sobre superficies oscuras (sub-caso de H3)
- **WCAG:** 2.4.11 (AA).
- **Ubicación:** sidebar `hero #0f1b2d`, hero de landing, botón `aviso` rojo, ghost-sobre-hero.
- **Problema:** anillo azul `accent` sobre azul tinta del sidebar → contraste insuficiente del indicador.
- **Fix:** anillo claro alternativo en contexto oscuro (cubierto por el fix de H3; se lista aparte por ser fácil de pasar por alto).

### L3 — Switcher de tema flotante: foco, teclado y anuncio
- **WCAG:** 2.1.1 (A); 4.1.2 Nombre/Rol/Valor (A); 2.4.11 (AA).
- **Ubicación:** `DESIGN.md`/`EXPERIENCE.md` switcher flotante (toga/notario/salvaguarda).
- **Problema:** control flotante (posición fija) suele quedar fuera del orden de Tab o sin nombre accesible; al cambiar de tema, el cambio de color no debe ser la única confirmación (1.4.1). Además, cambiar de tema **no debe romper** ninguno de los contrastes — y hoy varios fallan en los tres (C1/L1).
- **Fix:** que sea un grupo de radios/toggle accesible (nombre "Tema", estado seleccionado anunciado), enfocable en orden lógico, con foco visible; y que los 3 temas pasen el gate de contraste (C3) para que cambiar de tema nunca degrade AA.

---

## Apéndice — Tabla de contraste calculada (sRGB, fórmula WCAG)

Marcados ❌ los < 4.5:1 (texto normal). UI/borde usa 3:1.

| Combinación | Toga | Notario | Salvaguarda |
|---|---|---|---|
| ink / surface | 17.28 ✅ | 15.91 ✅ | 16.35 ✅ |
| ink / bg | 14.97 ✅ | 13.95 ✅ | 14.55 ✅ |
| ink-soft / surface | 7.46 ✅ | — | — |
| **muted / surface** | **3.09 ❌** | **3.56 ❌** | **4.00 ❌** |
| muted / bg (Toga) | 2.67 ❌ | — | — |
| on-hero / hero | 12.73 ✅ | 13.15 ✅ | 13.27 ✅ |
| on-hero-soft / hero | 6.49 ✅ | 6.10 ✅ | 6.01 ✅ |
| accent / surface | 4.60 ✅* | 6.49 ✅ | 6.57 ✅ |
| **banda-real / -bg** | **4.44 ❌** | 5.16 ✅ | 4.66 ✅ |
| banda-homonimo / -bg | 4.62 ✅ | 5.13 ✅ | 4.46 ❌(filo) |
| **banda-verificar / -bg** | **3.01 ❌** | **3.86 ❌** | **2.79 ❌** |
| **banda-descartado / -bg** | **3.76 ❌** | 4.27 ✅(filo) | 4.27 ✅(filo) |
| pep / -bg | 4.62 ✅ | 4.78 ✅ | 4.70 ✅ |
| Bloqueo-UIF #fff / banda-real | 5.26 ✅ | — | — |

\* `accent` 4.60 pasa texto normal por poco; como color de **foco/UI** (3:1) sobra, pero ver H3/L2 sobre fondos oscuros.

**Conclusión de la tabla:** el ámbar `verificar` falla en los 3 temas (peor: Salvaguarda 2.79). `muted` falla como texto informativo en los 3. `banda-real` falla en Toga. `banda-descartado` falla en Toga. La promesa "mantiene contraste sobre cada fondo" del DESIGN.md no se cumple hoy → C3.
