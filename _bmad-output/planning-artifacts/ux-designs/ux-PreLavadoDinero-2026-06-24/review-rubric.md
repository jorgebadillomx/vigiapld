# Review Rubric — Spines de UX Vigía (DESIGN.md + EXPERIENCE.md)

Validación del par `DESIGN.md` + `EXPERIENCE.md` como contrato source-extraíble para consumidores downstream (arquitectura, dev de historias). Calibrada contra los ejemplos de forma shadcn, `design-md-spec.md`, y las fuentes (PRD `docs/prd_screening_pld.md`, `CONTEXT.md`, `architecture.md`, `.decision-log.md`, `imports/Vigia_contexto_IA.md`, `imports/Vigia.html`).

Fecha: 2026-06-24 · Revisor: validador de spines de UX

---

## Overall verdict

**ADEQUATE, cercano a STRONG.** El par es genuinamente source-extraíble: el token de banda de confianza —el recurso de marca más cargado— está completo, hexeado, pareado con `-bg` y disciplinado (color = significado, nunca decoración, siempre con etiqueta textual). La herencia resuelve, los nombres de componente son consistentes entre archivos, las secciones están en orden canónico, y la sección inventada (Banda de confianza) gana su lugar. Las decisiones load-bearing del dominio (matriz banda×tipo_lista, fidelidad prosa↔verdicto, multi-cliente por sujeto obligado, módulos bloqueados visibles) están comprometidas y trazan a fuente.

Lo que lo retiene de STRONG es **cobertura de flujos** (2 de ~10 módulos de la suite tienen flujo nombrado; avisos 24h, monitoreo y BC —explícitamente en alcance— no tienen protagonista ni ruta de falla propia) y **dos huecos mecánicos de verificación** que el propio texto promete pero no entrega: el contraste AA del ámbar `banda-verificar` sobre su `-bg` se nombra como "a verificar" pero no se declara ratio, y los valores de banda por-tema (notario/salvaguarda) se afirman pero no se listan sus hex. Ninguno es bloqueante; todos son cerrables con datos que ya existen o son triviales de medir.

| # | Categoría | Veredicto |
|---|---|---|
| 1 | Flow coverage | **thin** |
| 2 | Token completeness | **adequate** |
| 3 | Component coverage | **strong** |
| 4 | State coverage | **adequate** |
| 5 | Visual reference coverage | **strong** |
| 6 | Bloat & overspecification | **strong** |
| 7 | Inheritance discipline | **adequate** |
| 8 | Shape fit | **strong** |

**Conteo de hallazgos:** critical 0 · high 3 · medium 6 · low 5.

---

## 1. Flow coverage — **thin**

Extracción: EXPERIENCE.md tiene 2 Key Flows. Ambos están bien construidos —protagonista nombrado (Don Refugio, joyero; Lucía, contadora de despacho), pasos numerados, clímax marcado en negrita, ruta de falla explícita— y entre los dos cubren la cuña de capa-1 (subir padrón → dictamen → dashboard) y el caso multi-cliente + resolución de homónimo + export de expediente. La forma de cada flujo es ejemplar y cumple el patrón del ejemplo shadcn.

El problema es de **cobertura, no de calidad**: el `.decision-log.md` (2026-06-24, "Alcance de la spec de UX") fija explícitamente **portal completo (suite entera)**, y la propia IA de EXPERIENCE.md lista 9+ surfaces de panel (avisos, monitoreo, BC/partes, EBR, programa PLD, validación de identidad, rescreening, resumen consolidado). Ningún flujo tiene como protagonista los módulos que más consecuencia legal cargan.

**[high]** Ningún Key Flow cubre el **aviso de 24h de punta a punta** como protagonista (EXPERIENCE.md §Key Flows; el reloj solo aparece como clímax secundario en Flow 1 y como ruta de falla en Flow 2). El reloj de aviso es el concepto de mayor consecuencia legal del producto (PRD US-22, AC-Aviso 24h, CONTEXT.md "Reloj de aviso") y el módulo Avisos está en alcance pero bloqueado (AS-2). Un consumidor que construya la pantalla de Avisos no tiene un flujo que le diga cómo se ve atender un reloj `<6h` desde la alerta hasta "Generar aviso XML (estado bloqueado)". *Fix:* añadir Flow 3 con protagonista que reciba una `Coincidencia detectada` por rescreening, vea el reloj subir a pendientes, y choque con el estado bloqueado AS-2 al intentar generar el XML (la ruta de falla es justamente el bloqueo legal, no un error técnico).

**[medium]** El **monitoreo transaccional** (suite, bloqueado AS-4) y el **Beneficiario Controlador / partes relacionadas** (suite, en alcance fase 1) no tienen flujo (EXPERIENCE.md §IA los lista como surfaces, §Key Flows no los toca). BC es especialmente notable porque es una brecha cerrada vs. KYC Systems (PRD Further Notes) y tiene un patrón de UX no trivial: declarar cadena de control + screening ligado al expediente. *Fix:* al menos un flujo corto de BC (registrar un BC declarado de un sujeto moral → screening → evidencia al expediente); para monitoreo basta un flujo spec-only que muestre el estado bloqueado, ya que AS-4 impide construir la regla.

**[low]** Los dos flujos existentes solo viven en el **panel privado**; no hay flujo que arranque en la landing y cruce a conversión→registro→panel como una sola narrativa (Flow 1 lo roza al mencionar el verificador gratis, pero el salto "Entrar al panel" no es un paso instrumentado). Para un producto cuya tesis de distribución es la landing de urgencia (`plan_distribucion.md`, tono Agresivo), un flujo de conversión nombrado reforzaría el contrato. *Fix:* opcional — extender Flow 1 o añadir un mini-flujo de conversión pública.

---

## 2. Token completeness — **adequate**

Extracción de frontmatter (DESIGN.md): colores 25 tokens con hex (bg, surface, surface-2, ink, ink-soft, muted, line, accent, accent-ink, accent-soft, hero, on-hero, on-hero-soft, gold, banda-real/-bg, banda-homonimo/-bg, banda-verificar/-bg, banda-descartado/-bg, pep/-bg). Typography 13 roles. Rounded 5 niveles. Spacing 5 tokens nombrados. Components 11 entradas. **Toda referencia `{path.to.token}` en prosa resuelve:** `{colors.accent}`, `{colors.surface}`, `{colors.surface-2}`, `{colors.banda-real}`, `{colors.line}`, `{colors.accent-ink}`, `{colors.hero}`, `{rounded.md}`, `{rounded.sm}`, `{rounded.lg}`, `{rounded.xl}`, `{typography.data-lg}` — todas tienen definición en frontmatter. En EXPERIENCE.md las referencias `{colors.surface-2}`, `{colors.banda-*}`, `{colors.accent}` también resuelven a DESIGN.md. **Cero colores sin hex en el tema canónico Toga.** Esto es fuerte.

**[high]** El **contraste AA del ámbar `banda-verificar` (#bd7d12) sobre su `-bg` (#faefd6)** no está declarado, pese a que tanto DESIGN.md §Components/Inputs como EXPERIENCE.md §Accessibility Floor lo nombran explícitamente como la combinación a vigilar ("especialmente ámbar `banda-verificar` sobre su `-bg`"). El prompt lo marca como load-bearing. Cálculo: #bd7d12 sobre #faefd6 da un ratio ≈ 3.9:1 — **pasa AA para texto grande/bold (≥18.66px o ≥14px bold) pero queda por debajo de 4.5:1 para texto normal**. El badge de banda usa 700 12.5px (bold pero pequeño), lo que lo deja en zona fronteriza. *Fix:* declarar el ratio medido de cada par banda-color/banda-bg en DESIGN.md §Colors o §Accessibility, y si `banda-verificar` sobre `-bg` no llega a 4.5:1 en el tamaño real del badge, oscurecer el texto a ~#9a6610 (o subir peso/tamaño) y anotarlo como delta obligatorio igual que se hizo con el anillo de foco.

**[medium]** Los **valores hex de banda por tema notario y salvaguarda no están listados.** DESIGN.md §Colors afirma "Cada token de banda se redefine sutilmente por tema (mantiene contraste sobre cada fondo)" y el `.decision-log.md` confirma "Tokens de banda por tema (no globales)", pero el frontmatter solo trae los valores Toga y el cuerpo solo da `accent`/`bg`/`gold` de los otros dos temas. Un consumidor que implemente el switcher de tema (componente declarado, ver §3) no tiene los hex de banda de notario/salvaguarda. *Fix:* listar los valores de banda por tema (al menos como tabla en §Colors) o declarar explícitamente que se derivan en build con una regla nombrada; sin esto, el "mantiene contraste sobre cada fondo" no es verificable downstream.

**[low]** `colors.gold` (#cfa13a) está definido en frontmatter pero su uso en prosa es marginal ("acentos editoriales puntuales… Nunca como color de acción"). No es huérfano (se usa), pero raya en token sin trabajo suficiente. *Fix:* aceptable como está; si en una pasada futura no gana uso concreto en un componente, considerar retirarlo.

**[low]** `spacing` se declara con tokens nombrados (`gutter`, `content-max`, etc.) pero el comentario dice "se normaliza la escala Tailwind 4-based" sin enumerarla en frontmatter (solo en prosa §Layout: "4/8/12/16/20/24/32/40/48/64"). Es consistente con el patrón shadcn de heredar la escala, pero un consumidor estricto la querría como tokens. *Fix:* aceptable (hereda Tailwind); opcionalmente referenciar la escala Tailwind por nombre como hace el ejemplo Drift.

---

## 3. Component coverage — **strong**

Extracción cruzada. Cada componente usado aparece en DESIGN.md.Components (visual) Y EXPERIENCE.md.Component Patterns (comportamiento), con nombres consistentes:

| Componente | DESIGN.md (visual) | EXPERIENCE.md (comportamiento) |
|---|---|---|
| Button (primario/secundario/aviso) | ✅ §Components + tokens | ✅ (implícito en flujos/primitivas; aviso en Reloj) |
| Badge de banda | ✅ `badge-banda` | ✅ "Badge de banda" |
| Tarjeta de dictamen | ✅ `dictamen-card` | ✅ "Tarjeta de dictamen" |
| KPI card | ✅ `kpi-card` | ✅ (Dashboard KPIs) |
| Tabla de dictámenes / Fila | ✅ `table-dictamenes` | ✅ "Fila de dictamen" |
| Reloj de aviso 24h | ✅ `reloj-aviso` | ✅ "Reloj de aviso 24h" |
| Sidebar | ✅ `sidebar` | ✅ (IA panel, selector) |
| Dropzone + stepper | ✅ `dropzone` | ✅ "Dropzone + stepper" |
| Selector de sujeto obligado | ✅ (note en `sidebar`) | ✅ "Selector de sujeto obligado" |
| Chips de filtro | — (no fila propia) | ✅ "Chips de filtro" |
| Switcher de tema | ✅ (Shapes: "switcher de tema") | ✅ "Switcher de tema" |
| Verificador en vivo (landing) | — (no fila propia) | ✅ "Verificador en vivo" |

La cobertura es sólida y los nombres son idénticos entre archivos (disciplina de herencia correcta).

**[medium]** **Chips de filtro** y **Verificador en vivo** tienen comportamiento en EXPERIENCE.md.Component Patterns pero no tienen fila de spec visual propia en DESIGN.md.Components. Chips de filtro se cubre indirectamente ("píldoras: chips, filtros" en §Shapes y "Activo = `{colors.accent}`" en EXPERIENCE) y el verificador hereda tarjeta de dictamen + dropzone, así que ningún consumidor queda totalmente a ciegas — pero la regla del prompt ("cada componente usado tiene fila en DESIGN.md.Components Y en EXPERIENCE.md") no se cumple al pie para estos dos. *Fix:* añadir dos filas cortas a DESIGN.md.Components: `chip-filtro` (estado activo `accent`/`accent-ink`, inactivo `surface-2`/`ink-soft`, `rounded.full`, conteo en mono) y `verificador-card` (o una nota de que reusa `dictamen-card` + `dropzone`).

---

## 4. State coverage — **adequate**

Extracción de §State Patterns (EXPERIENCE.md), recorriendo surfaces de la IA. Estados cubiertos: carga inicial (Skeleton), padrón vacío, corrida procesando, sin hits, requiere-verificación, LLM caído (→plantilla, alineado con NFR-3/CM-3 del PRD), lista stale (alineado con la "detección de staleness" de arquitectura), módulo bloqueado AS-2/AS-4, reloj <6h, sin permiso (otro sujeto obligado, alineado con ADR-0003 aislamiento), offline. Es una cobertura amplia y bien anclada a fuente —notablemente el estado "módulo bloqueado" y "LLM caído→plantilla" capturan decisiones de arquitectura que muchos specs olvidarían.

**[medium]** **Falta el estado de error de ingesta como fila de State Patterns.** La ruta de falla de Flow 1 ("Excel con columnas irreconocibles → pide asignar Nombre/RFC a mano antes de procesar; no procesa basura") y el AC-Ingesta del PRD describen un comportamiento de error rico en Subir padrón, pero §State Patterns no tiene una fila "archivo inválido / mapeo fallido / supera límite de 10,000 (NFR-2)". El comportamiento existe disperso en el flujo y el componente Dropzone, pero no como estado de surface enumerado. *Fix:* añadir fila(s) a §State Patterns: "Archivo no mapeable / filas inválidas / >10,000 sujetos (NFR-2)" en Subir padrón, con el tratamiento (bloquear procesamiento, reportar antes, ProblemDetails 413/422 según arquitectura §Edge Conventions).

**[medium]** **No hay estado de error de red/servidor genérico (5xx) ni de timeout** para el screening síncrono. El portal es cascarón sobre `/v1/`; la arquitectura define ProblemDetails (RFC 7807) y degradación (circuit breaker → "sanciones pendientes" cuando OpenSanctions cae). EXPERIENCE.md cubre "LLM caído" y "Offline" pero no "OpenSanctions caído → dictamen con marca *sanciones pendientes*", que es un estado de degradación parcial explícito en arquitectura (SPOF de sanciones/PEP). *Fix:* añadir fila "Sanciones pendientes (OpenSanctions no disponible)" — el dictamen se emite con tipos de lista locales y una marca visible, consistente con architecture.md §API Patterns.

**[low]** El estado **"sin sesión / no autenticado"** y la transición landing→panel no tienen tratamiento (auth es ASP.NET Core Identity por arquitectura). Probablemente fuera del alcance "UX ligera del portal", pero el contrato lo deja implícito. *Fix:* una línea en §State o §IA confirmando que el gate de auth es estándar shadcn/Identity y no recibe diseño de marca propio.

---

## 5. Visual reference coverage — **strong**

Ambas spines enlazan a `imports/Vigia.html` en la sección relevante y dicen qué ilustra:
- DESIGN.md header: "Las spines ganan sobre el prototipo `imports/Vigia.html` en conflicto." ✅ spines-win-on-conflict declarado.
- EXPERIENCE.md header: igual. ✅ Y en §IA: "→ Referencia de composición: `imports/Vigia.html` (landing + capa-1 del panel). Surfaces de suite y responsive: spec-only hasta sus mocks. Spine gana en conflicto." — esto es ejemplar: enlaza, dice qué cubre el mock (landing + capa-1), qué NO cubre (suite + responsive son spec-only), y declara la regla de conflicto.

Verificado en disco: `imports/Vigia.html` y `imports/Vigia_contexto_IA.md` **existen** en la carpeta de la spine (no son enlaces rotos). El ejemplo shadcn enlaza a múltiples mocks por surface (`mockups/today.html`, etc.); aquí hay un solo HTML monolítico, lo cual es honesto dado que el prototipo es un único Design Component de ~1MB. **Sin huérfanos.**

**[low]** El prototipo solo ilustra landing + capa-1; las surfaces de suite (avisos, monitoreo, BC, EBR, programa PLD) y todo el responsive **no tienen referencia visual** —y la spine lo declara honestamente como "spec-only hasta sus mocks". No es un huérfano (es una ausencia declarada), pero significa que el consumidor de esos módulos extrae sin ancla visual. *Fix:* ninguno requerido; la declaración es correcta. Anotar para una pasada futura que esos módulos ganarían un mock antes de su dev de historias.

---

## 6. Bloat & overspecification — **strong**

DESIGN.md ejerce voz editorial (permitida): "gravitas sin polvo", "el color *es* el dictamen de un vistazo" — apropiado para la sección Brand & Style. EXPERIENCE.md se mantiene operativo y sin voz de marca (correcto: la voz vive en DESIGN.md, y EXPERIENCE.md lo dice). Las tablas se usan donde corresponde (IA, Component Patterns, State Patterns, Voice/Tone, Responsive). Los px aparecen casi siempre como **valor de token en frontmatter** (correcto) y no se restatan en prosa salvo el ramp tipográfico resumido (útil, no bloat).

**[low]** Algún px suelto en notes de componente que podría ser token: `button-primary` note "padding 12-15px 20-26px; font ui 700 14-15.5px" y `reloj-aviso` "18px lista / 26px detalle". Son rangos de prototipo normalizados, no tokens, así que el px crudo es defendible aquí (no existe token de padding de botón ni de tamaño de reloj), pero rozan la sobre-especificación. *Fix:* aceptable; si se quisiera purista, crear tokens `reloj-size-lista`/`reloj-size-detalle`. No es bloqueante.

**[low]** §Layout & Spacing restata la escala Tailwind ("4/8/12/16/20/24/32/40/48/64") que ya se dice heredada — leve restatement de fuente. *Fix:* trivial; referenciar "escala Tailwind 4-based" por nombre basta, como hace Drift.

Sin restatement significativo del PRD: las spines referencian decisiones (matriz banda×tipo_lista, fidelidad prosa↔verdicto) en una línea y apuntan a la fuente en vez de re-explicarlas. Disciplina correcta.

---

## 7. Inheritance discipline — **adequate**

`sources` (ambos archivos): `docs/prd_screening_pld.md` ✅, `CONTEXT.md` ✅, `_bmad-output/planning-artifacts/architecture.md` ✅, `imports/Vigia.html` ✅, `imports/Vigia_contexto_IA.md` ✅ (DESIGN.md) — todos resuelven en disco. EXPERIENCE.md `design: DESIGN.md` ✅ resuelve. Glosario: los términos de dominio usados (banda de confianza y sus 4 estados, tipo de lista, sujeto obligado, Cuenta, padrón, dictamen, reloj de aviso, expediente, Beneficiario Controlador, EBR) son **idénticos a CONTEXT.md**, incluyendo el respeto de los `_Avoid_`: EXPERIENCE.md §Voice/Tone proscribe "Indeterminado" citando CONTEXT.md, y usa "sujeto obligado" no "tenant/cliente". Nombres de componente idénticos entre los dos archivos (ver §3). Referencias de token de EXPERIENCE resuelven a DESIGN (ver §2).

**[high]** **Inconsistencia de base de path en `sources` entre los dos archivos, y mezcla de bases dentro de cada lista.** Las spines viven en `_bmad-output/planning-artifacts/ux-designs/ux-PreLavadoDinero-2026-06-24/`. La lista `sources` mezcla paths relativos a la **raíz del repo** (`docs/prd_screening_pld.md`, `CONTEXT.md`, `_bmad-output/planning-artifacts/architecture.md`) con paths relativos a la **carpeta de la spine** (`imports/Vigia.html`, `imports/Vigia_contexto_IA.md`, y `design: DESIGN.md`). Ambos grupos resuelven *si* el consumidor sabe qué base aplicar a cada uno, pero un source-extractor mecánico que asuma una sola base romperá la mitad de las referencias. *Fix:* normalizar a una sola convención — o todo relativo a la raíz del repo (`_bmad-output/.../imports/Vigia.html`) o anotar explícitamente en una línea de header que `imports/` y `DESIGN.md` son relativos a la carpeta de la spine y el resto a la raíz. Recomendado: paths desde la raíz del repo para todo.

**[medium]** EXPERIENCE.md cita reglas de arquitectura por etiqueta interna sin que la etiqueta sea resoluble: "regla de fidelidad, **arquitectura H2**" (§Voice/Tone). `architecture.md` no usa una numeración "H2" — la regla de fidelidad prosa↔verdicto existe en architecture.md §Process Patterns y en el PRD ("Fidelidad prosa↔verdicto"), pero el ancla "H2" no resuelve a ninguna sección/heading del documento de arquitectura. *Fix:* reemplazar "arquitectura H2" por la referencia resoluble (p. ej. "architecture.md §Process Patterns / PRD 'Fidelidad prosa↔verdicto'").

**[low]** Las referencias a ADRs (`ADR-0003`, `ADR-0005`) en EXPERIENCE.md resuelven conceptualmente (existen `docs/adr/0003`, `0005` según frontmatter de architecture.md) pero `docs/adr/` no está en `sources`. *Fix:* opcional — añadir `docs/adr/` a `sources` o confiar en que architecture.md ya los encadena (aceptable).

---

## 8. Shape fit — **strong**

**DESIGN.md — orden canónico de secciones** (vs. design-md-spec.md): Brand & Style → Colors → Typography → Layout & Spacing → Elevation & Depth → Shapes → Components → Do's and Don'ts. **Exacto y completo**, ninguna omitida, orden bloqueado respetado. Frontmatter usa los keys correctos (`name`, `description`, `colors` flat kebab-case con hex, `typography` nested, `rounded`, `spacing`, `components` con `{path.to.token}`). El uso de `note` en typography/components para convenciones semánticas (vibe Tecnológico, urgencia del reloj) sigue el patrón recomendado del spec. Light/dark: declara "dark no en fase 1, tokens `-dark` separados si se añade" — alineado con el patrón del spec.

**EXPERIENCE.md — defaults presentes:** Foundation, Information Architecture, Voice and Tone, Component Patterns, State Patterns, Interaction Primitives, Accessibility Floor, Responsive & Platform, (Inspiration & Anti-patterns está ausente — ver abajo), Key Flows. Sigue el orden y los headers del ejemplo shadcn.

**Sección inventada "Banda de confianza — semántica de experiencia":** **gana su lugar limpiamente.** Es el concepto de dominio más cargado (CONTEXT.md lo define como entidad central), no es derivable de los defaults, y codifica la regla load-bearing: "La acción y el riesgo se derivan de la matriz `(banda × tipo_lista)`, nunca del binario coincidió/no — la UI nunca presenta 'coincide = rechazar'." Esto compromete directamente la decisión de ADR-0004 / PRD AC-Dictamen ("PEP nunca produce accion_sugerida = rechazar"). Inventar esta sección es exactamente lo que el spec permite cuando una sección gana su lugar.

**[medium]** EXPERIENCE.md **omite la sección "Inspiration & Anti-patterns"** que el ejemplo shadcn trae (lifted-from / rejected). No es un default obligatorio del spec de EXPERIENCE (el spec canónico es design-md; EXPERIENCE sigue el ejemplo), pero su ausencia significa que las **decisiones rechazadas** (qué deliberadamente NO se hace) no quedan registradas en la spine —p. ej. "rechazado: detección de inusualidad con IA en fase 1" (PRD Out of Scope), "rechazado: presentar el aviso en nombre del cliente", "rechazado: el binario coincide=rechazar". Algunas viven en Do's/Don'ts de DESIGN.md y en la sección de banda, pero no hay un lugar único de anti-patterns de experiencia. *Fix:* añadir una sección corta "Inspiration & Anti-patterns" a EXPERIENCE.md consolidando los rechazos ya dispersos (no presenta el aviso; no infiere intento de operación; no detección IA en fase 1; no "coincide=rechazar"; PEP≠rechazo) — varios ya están como Don'ts, esto los eleva a decisión rechazada explícita y trazable.

---

## Mechanical notes

- **Archivos verificados en disco:** `DESIGN.md`, `EXPERIENCE.md`, `.decision-log.md`, `imports/Vigia.html`, `imports/Vigia_contexto_IA.md` (todos en la carpeta de la spine); `CONTEXT.md`, `docs/prd_screening_pld.md`, `_bmad-output/planning-artifacts/architecture.md` (raíz/docs). Todas las rutas físicas existen.
- **Nota de path:** `imports/Vigia_contexto_IA.md` se referencia en el prompt como `imports/Vigia_contexto_IA.md` relativo a la raíz, pero solo existe dentro de la carpeta de la spine (`.../ux-PreLavadoDinero-2026-06-24/imports/`). No hay copia en la raíz del repo. Ver hallazgo 7-[high].
- **Token de banda — recuento:** los 5 colores de banda + PEP, cada uno con par `-bg`, están todos hexeados en Toga. Cero faltantes en el tema canónico.
- **Referencias `{path.to.token}` rotas:** ninguna detectada. Todas resuelven al frontmatter de DESIGN.md.
- **Términos proscritos (CONTEXT.md `_Avoid_`):** no aparece "indeterminado", "tenant", "cliente final" en las spines. EXPERIENCE.md proscribe activamente "Indeterminado" en §Voice/Tone. Disciplina de glosario correcta.
- **Contraste pendiente de declarar:** `banda-verificar #bd7d12` / `banda-verificar-bg #faefd6` ≈ 3.9:1 (estimado) — fronterizo para texto bold pequeño del badge. Ver 2-[high]. Los demás pares de banda Toga deben declararse también pero a ojo superan 4.5:1 (rojo, azul, verde, morado sobre sus bg claros).
- **Ancla no resoluble:** "arquitectura H2" en EXPERIENCE.md §Voice/Tone (ver 7-[medium]).
- **Cobertura de alcance vs. decision-log:** el log fija "portal completo (suite entera)"; las spines especifican la IA completa de la suite (todas las surfaces listadas con estado) pero solo 2/~10 módulos tienen Key Flow. Brecha de flujos, no de IA.
