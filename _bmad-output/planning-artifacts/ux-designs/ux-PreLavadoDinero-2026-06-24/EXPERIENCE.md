---
name: Vigía
status: final
created: 2026-06-24
updated: 2026-06-24
project: PreLavadoDinero
design: DESIGN.md
sources:
  - docs/prd_screening_pld.md
  - CONTEXT.md
  - _bmad-output/planning-artifacts/architecture.md
  - _bmad-output/planning-artifacts/ux-designs/ux-PreLavadoDinero-2026-06-24/imports/Vigia.html
---

# Vigía — Experience Spine

> Cómo se comporta el portal Vigía (PLD/FT, LFPIORPI). Web responsive, shadcn/ui sobre React+Vite+Tailwind+TanStack Query+React Router. `DESIGN.md` es la referencia de identidad visual; aquí vive el comportamiento. Las spines ganan sobre el prototipo `imports/Vigia.html` en conflicto. Vocabulario de dominio: `CONTEXT.md` (autoridad).

## Foundation

Web **responsive** (móvil incluido), una sola base de código con dos caras conectadas: **landing pública** (venta) y **panel privado** (operación). shadcn/ui aporta los primitivos accesibles (Dialog, Sheet, Popover, Toast, Table, Tabs, Command, Skeleton); este spine especifica solo el delta de comportamiento. El portal es un **cascarón delgado sobre el contrato `/v1/`**: arma nada de la decisión, solo presenta el dictamen que el motor ya decidió (consistente con la arquitectura).

**Multi-cuenta:** una **Cuenta** (despacho o sujeto obligado directo) agrupa uno o más **sujetos obligados**. El panel opera siempre "en el contexto de un sujeto obligado activo"; el despacho cambia de contexto con un selector (ver IA). El aislamiento es por sujeto obligado (ADR-0003) y se respeta en cada vista.

## Information Architecture

### Landing pública (scroll de una página)

| Sección | Propósito |
|---|---|
| Nav | Logo Vigía, secciones (Producto/Cómo funciona/Listas/Precios), CTA "Entrar al panel" |
| Hero | Urgencia factual (badge reforma + multa), CTA, stats (320+ fuentes / 24h / 10 años), tarjeta de dictamen de muestra. **Tono Agresivo** (canónico) |
| Lead magnets | Verificación de 1 nombre en vivo + calculadora de multa por giro |
| El homónimo | Diferenciador: mismo nombre, RFC+fecha distinta → dos dictámenes |
| Cómo funciona | 4 pasos (sube → coteja → dictamen → expediente) |
| Cobertura de listas | OFAC/ONU/UE/UK/PEP/SAT 69-B/LPB UIF + matching difuso |
| Precios | Esencial / Negocio / Despacho (por volumen) |
| Trust + disclaimers | "Herramienta de apoyo, no garantía"; UMA 2026/DOF |
| Footer | — |

### Panel privado (sidebar fijo + main con scroll)

| Surface | Reached from | Propósito | Estado |
|---|---|---|---|
| Selector de sujeto obligado | Sidebar (bloque superior) | Cambiar el sujeto obligado activo; despacho ve su lista | **multi-cliente** |
| Resumen consolidado (despacho) | Selector → "Todos" | KPIs y pendientes agregados de toda la cartera | **multi-cliente** |
| Resumen (dashboard) | Sidebar / app open | KPIs (padrón/hits/reales/avisos), reloj 24h en vivo, hits por banda, corridas recientes, cobertura | capa 1 |
| Subir padrón | Sidebar / "Subir padrón" | Dropzone → mapeo de columnas + validación → procesamiento → resumen | capa 1 |
| Dictámenes | Sidebar / "Dictámenes" | Tabla filtrable por banda | capa 1 |
| Detalle de dictamen | Fila de tabla / demo landing | Veredicto, fundamento+cotejos, prosa (LLM/plantilla), expediente (timeline), reloj+aviso si aplica | capa 1 |
| Configuración de rescreening | Sidebar | Cadencia y alcance; piso legal mínimo visible (ADR-0005) | capa 2 |
| Beneficiario Controlador / partes | Detalle de sujeto moral | Registrar/conservar BC y partes declarados; screening ligado al expediente | suite |
| EBR / matriz de riesgo | Sidebar | Clasificación por 5 dimensiones; exportable | suite |
| Monitoreo transaccional | Sidebar | Carga de operaciones, acumulación 6m, umbrales | **suite — bloqueado AS-4** (umbrales sin confirmar) |
| Avisos | Sidebar / reloj | Lista de relojes 24h; generar/registrar XML SPPLD | **suite — bloqueado AS-2** (XSD del SAT sin confirmar) |
| Programa PLD | Sidebar | Manual rellenado, capacitación, roles, auditoría interna | suite |
| Validación de identidad | Detalle de sujeto | RFC/CURP/69-B (nivel barato); INE/biometría por integración | suite (reforzada = proveedor a definir, AS-3) |

Sidebar colapsa a iconos en `md`; se vuelve `Sheet` (drawer) en `sm`. Módulos bloqueados (AS-2/AS-4) aparecen en la nav con estado visible "Próximamente — pendiente de configuración legal", no ocultos.

→ Referencia de composición: `imports/Vigia.html` (landing + capa-1 del panel). Surfaces de suite y responsive: spec-only hasta sus mocks. Spine gana en conflicto.

## Voice and Tone

Microcopy. La voz de marca y la postura estética viven en `DESIGN.md`. Español de México, registro técnico-legal, orientado a riesgo y consecuencia.

| Do | Don't |
|---|---|
| "Separa al criminal real del simple *homónimo*." | "¡Detecta lavado de dinero al instante! 🚀" |
| "Requiere verificación — falta RFC o fecha de nacimiento." | "Indeterminado" (término proscrito, CONTEXT.md) |
| "PEP — dispara debida diligencia reforzada (EDD), no rechazo." | "PEP — rechazar cliente" |
| "Reloj de aviso: 18h 42m restantes — cuenta desde el evento." | "Tienes tiempo de sobra para el aviso." |
| "Herramienta de apoyo, no garantía. El aviso lo presenta el sujeto obligado." | Prometer que Vigía presenta el aviso o garantiza cumplimiento |
| Cifras de multa siempre con fuente (UMA 2026 · DOF) | Cifras alarmistas sin fuente |

La prosa del dictamen la redacta el LLM en voz formal-legal, pero **debe citar literalmente la banda, el tipo de lista y la acción** del verdicto determinístico (regla de fidelidad, arquitectura H2); si no, cae a plantilla y se etiqueta `redacción: plantilla`.

## Component Patterns

Comportamiento. Specs visuales en `DESIGN.md.Components` o en shadcn.

| Componente | Uso | Reglas de comportamiento |
|---|---|---|
| Fila de dictamen (tabla) | Dictámenes | Clic en cualquier parte → Detalle. Score y banda visibles sin abrir. Hover resalta (`{colors.surface-2}`). |
| Badge de banda | Tabla, detalle, tarjetas | Color = significado (`{colors.banda-*}`); siempre con etiqueta textual (no solo color — a11y). |
| Tarjeta de dictamen | Hero, demo, detalle | Resume veredicto; "Ver el dictamen completo →" abre Detalle en el panel. |
| Reloj de aviso 24h | Dashboard, detalle, Avisos | Cuenta regresiva en vivo (tick 1s) desde el `evento origen`, no desde la lectura. Color de urgencia escala (<6h rojo, <12h ámbar). "Generar aviso XML" → flujo de avisos (bloqueado AS-2: muestra estado, no genera). |
| Verificador en vivo (landing) | Lead magnet | Input + chips de ejemplo → animación de cotejo (~1.25s) → tarjeta de resultado con banda. Sin registro. |
| Dropzone + stepper | Subir padrón | 3 pasos (Cargar/Mapear/Resultado). Acepta .xlsx/.xls/.csv, máx 10,000 sujetos/corrida (NFR-2). Mapeo de columnas editable; reporta filas inválidas/duplicadas antes de procesar. |
| Selector de sujeto obligado | Sidebar | Despacho: lista de sus sujetos obligados + "Todos" (consolidado). Sujeto directo: muestra el único, sin selector activo. Cambiar contexto recarga las vistas del panel. |
| Chips de filtro | Dictámenes | Todas/Reales/Homónimos/A verificar/Descartados, con conteo. Activo = `{colors.accent}`. |
| Switcher de tema | Flotante (opcional) | Cambia entre toga/notario/salvaguarda. Toga por defecto. |

## State Patterns

| Estado | Surface | Tratamiento |
|---|---|---|
| Carga inicial | Dashboard, Dictámenes | shadcn `Skeleton` con la forma esperada (KPIs, filas). |
| Padrón vacío | Dashboard | "Aún no has subido tu padrón." + CTA "Subir padrón". Nunca pantalla en blanco. |
| Corrida procesando | Subir padrón | Spinner + log mono de listas cotejadas; resumen al terminar (reales/homónimos/a verificar/limpios). |
| Sin hits | Dictámenes | "Ningún sujeto requiere acción. Todo descartado por nombre." (verde). |
| `requiere-verificación` | Detalle | Estado ámbar + **qué dato pedir** ("Falta RFC y fecha de nacimiento para resolver"). Acción: capturar dato → re-resolver. |
| LLM caído | Detalle | Dictamen igual presente con campos completos; prosa = plantilla, badge `redacción: plantilla`; "Re-redactar" disponible cuando vuelva. |
| Lista stale | Dashboard / Detalle | Aviso: "La lista SAT 69-B no se ha refrescado desde {fecha}." (no bloquea, advierte). |
| Módulo bloqueado (AS-2/AS-4) | Avisos / Monitoreo | "Próximamente — pendiente de configuración legal (umbrales/esquema del SAT)." Explica por qué, no error. |
| Reloj <6h | Dashboard / Avisos | Contador rojo + dot pulsante; el aviso sube al tope de pendientes. |
| Sin permiso (otro sujeto obligado) | Panel | La vista nunca expone datos de otro sujeto obligado; el selector solo muestra los de la Cuenta. |
| Offline | Global | shadcn `Toast`: "Sin conexión. Reintentando." (el screening es servidor; no hay escritura local de dictámenes). |

## Interaction Primitives

- **Clic-para-actuar** en filas (toda la fila es objetivo); el dato crítico (score, banda) visible sin abrir.
- **Subir = arrastrar o elegir**; siempre hay botón "Usar archivo de ejemplo" para evaluación sin datos reales.
- **El reloj cuenta solo**, en vivo; el usuario nunca lo arranca a mano (lo dispara el motor o el canal).
- **Confirmación solo en lo irreversible**: aprobar un dictamen y generar un aviso piden confirmación; filtrar/navegar no.
- **Foco visible siempre** (anillo `{colors.accent}`) — corrige la deuda del prototipo (`outline:none`).
- **Prohibido:** scroll infinito (paginación en tablas); color como único portador de significado (siempre etiqueta textual); ocultar el disclaimer; stacks de modal > 1 nivel.

## Accessibility Floor

Comportamiento; el contraste visual vive en `DESIGN.md` (verificar AA en los 3 temas, especialmente ámbar `banda-verificar` sobre su `-bg`).

- **WCAG 2.2 AA** en toda la web responsive.
- **Banda de confianza nunca solo por color:** badge siempre con etiqueta ("Real", "Homónimo", "Requiere verificación", "Descartado") + punto. Daltonismo-seguro.
- Tablas con encabezados asociados; el lector anuncia "Dictámenes, {N} sujetos, filtro {banda}".
- Reloj 24h con `aria-live` cortés que anuncia cruces de umbral (<12h, <6h), no cada segundo.
- Orden de `Tab` = orden de lectura; `Esc` cierra el modal/popover superior.
- Todo dato mono mantiene tamaño legible (≥13px) y contraste AA; el dato sin unidad obvia (score 0–1, folio) lleva `aria-label` que lo explica al lector ("score de coincidencia 0.91").
- **Foco visible en todo interactivo**, anillo adaptativo a la superficie (DESIGN.md): azul `accent` en claro, claro `on-hero`/#fff sobre el sidebar oscuro (WCAG 2.4.7/2.4.11).
- **Reloj 24h y `aria-live`:** el contador que cambia cada segundo es `aria-hidden` (no se narra el tick); una región `aria-live="polite"` separada anuncia **solo cruces de umbral** ("Aviso 1: menos de 12 horas", "…menos de 6 horas").
- **Movimiento (WCAG 2.3.3):** dot pulsante, scanline del verificador y spinner respetan `prefers-reduced-motion: reduce` → el pulso pasa a estado estático de alto contraste, scanline/spinner a indicador no animado. Ninguna información depende del movimiento.
- **Banda y score nunca solo por color (WCAG 1.4.1):** la banda lleva etiqueta + punto; el **score lleva su valor numérico + etiqueta de riesgo textual** (Alto/Medio/Bajo), no solo el color de umbral.
- **Tabla en móvil (WCAG 1.3.1):** al pasar a tarjetas, cada campo conserva su etiqueta visible (Sujeto/Tipo de lista/Score/Banda/Riesgo) — no se pierde la asociación encabezado↔valor.
- **`lang="es"`** en el documento; fragmentos en otro idioma marcados.
- **Objetivos táctiles (WCAG 2.5.8):** ≥24px (idealmente 44px) en móvil para filas, chips, botones del drawer y "Generar aviso".
- Filas y chips personalizados son operables por teclado (rol y foco), no solo por clic (el prototipo no declaraba teclado).

## Performance Floor

El prototipo era una demo estática sin estrategia de rendimiento; aquí se **inventa el piso de rendimiento** como delta obligatorio, con el mismo rango de "regla dura" que el gate de contraste y el foco visible. Es load-bearing porque el panel procesa hasta **10,000 sujetos por corrida (NFR-2)** y el reloj de aviso tickea en vivo.

- **Cifras tabulares en todo dato vivo o tabular (Core Web Vitals: CLS).** El reloj de 24h, scores, KPIs, folios y precios usan `tabular-nums` (token `mono`, ver `DESIGN.md`). Regla dura: ningún número que cambie en vivo o se alinee en columna puede provocar salto de ancho. El reloj que tickea cada segundo es el caso crítico.
- **Virtualizar tablas/listas >50 filas.** La tabla de dictámenes puede traer cientos de hits de una corrida de 10,000 sujetos. Se virtualiza (render solo de filas visibles); combinado con la paginación ya exigida (no scroll infinito, ver Interaction Primitives). Aplica también a las tarjetas en móvil.
- **Reservar espacio para contenido asíncrono (CLS < 0.1).** Skeletons (ya en State Patterns) con la **forma y dimensiones** reales del contenido; la tarjeta de dictamen del hero, los KPIs y el verificador en vivo reservan su caja (`aspect-ratio`/alto fijo) para no empujar el layout al repoblarse. El dashboard que "se repuebla solo" (Flow 1, clímax) no debe saltar.
- **Estrategia de fuentes (ver `DESIGN.md` Typography):** `font-display: swap` en las tres familias; `preload` solo de las críticas above-the-fold; subset `latin`. Evita FOIT y bloqueo de render con 3 familias × múltiples pesos.
- **Latencia de interacción <100ms** para clic en fila, filtros y chips; el cotejo real es servidor (`/v1/`) y se cubre con Skeleton/spinner, pero la respuesta visual al tap (hover/active) es inmediata.
- **Code-splitting por ruta:** landing y panel son dos caras; el panel (TanStack Query, tablas, sheets) no debe cargar en la landing pública ni viceversa.

## Responsive & Platform

El prototipo era desktop-only; aquí se **inventa el piso responsive** (decisión del propietario: todo responsive).

| Breakpoint | Landing | Panel |
|---|---|---|
| `≥ lg` (1024px+) | Grids multi-columna (hero 2-col, listas/precios 3-4 col) | Sidebar fijo 248px + main; tablas completas; dashboard 2-col |
| `md` (768–1023px) | Grids colapsan a 1–2 col; `7vw` se mantiene | Sidebar → iconos; dashboard apila; tabla conserva columnas clave |
| `< md` (`sm`) | Una columna; CTA fijo; calculadora/verificador apilados | Sidebar → `Sheet` (drawer) desde top bar; **tabla de dictámenes → tarjetas** (nombre + banda + score); detalle a pantalla completa; reloj 24h como banner superior |

En móvil, el trabajo central soportado es **revisar dictámenes y atender el reloj de aviso**; subir padrón se optimiza para desktop pero no se bloquea en móvil. Web responsive, no app nativa (PWA con push es futuro, fuera de fase 1 — PRD Out of Scope).

## Banda de confianza — semántica de experiencia (sección inventada)

El concepto de dominio más cargado. Reglas de presentación, consistentes en toda la app:
- **`resuelto-real`** (rojo): veredicto fuerte; acción AML según matriz `(banda × tipo_lista)`. Sanciones → no operar/reportar; Bloqueo-UIF → bloqueo (más severo).
- **`resuelto-homónimo`** (azul): tranquiliza explícitamente ("mismo nombre, otra persona; RFC/fecha difieren"); conserva evidencia, sin acción.
- **`requiere-verificación`** (ámbar): **no es un error ni un veredicto** — es "falta dato"; siempre dice cuál y ofrece capturarlo.
- **`descartado-por-nombre`** (verde): limpio.
- **PEP** (morado, tipo de lista): recordatorio visible de que dispara **EDD, no rechazo**.
La acción y el riesgo se derivan de la matriz, nunca del binario coincidió/no — la UI nunca presenta "coincide = rechazar".

## Key Flows

### Flow 1 — Don Refugio sube su padrón por primera vez (joyería, una tarde de marzo)

Don Refugio, dueño de "Joyería La Esmeralda", supo de la reforma por un grupo de WhatsApp y entró asustado a Vigía.

1. Desde la landing (hero rojo: "Una multa de hasta $7,625,150…") prueba el **verificador gratis** con su propio nombre; ve la animación de cotejo y un dictamen `descartado-por-nombre` verde. Se convence.
2. "Entrar al panel" → dashboard con padrón vacío: "Aún no has subido tu padrón." Pulsa **Subir padrón**.
3. Arrastra su Excel de 248 clientes. Vigía mapea columnas, marca 3 filas sin nombre y 2 duplicadas; él corrige y procesa.
4. Spinner + log ("Cotejando contra OFAC · ONU · UE · UK…"). Termina: 5 reales · 3 homónimos · 3 a verificar · 237 limpios.
5. **Clímax:** el dashboard se repuebla solo — KPIs en mono, **5 coincidencias reales en rojo** y un **reloj de aviso 24h corriendo** sobre una de ellas. Don Refugio entiende de un vistazo qué es urgente y abre el primer dictamen real. El miedo difuso se volvió una lista de acciones concretas.

Falla: el Excel viene con columnas irreconocibles → el mapeo no autodetecta; Vigía pide asignar "Nombre" y "RFC" a mano antes de permitir procesar. No procesa basura.

### Flow 2 — Lucía resuelve un homónimo y defiende el criterio (contadora de despacho, multi-cliente)

Lucía lleva el PLD de 14 sujetos obligados desde un despacho.

1. En el sidebar usa el **selector de sujeto obligado** y entra al contexto de "Inmobiliaria del Bajío".
2. Va a **Dictámenes**, filtra por **A verificar** (ámbar): aparece "María González Pérez", score alto, sin RFC.
3. Abre el Detalle: estado ámbar, "Falta RFC y fecha de nacimiento para resolver". Captura el RFC que el cliente le envió.
4. Vigía re-resuelve: el RFC **no** coincide con el registro de la lista → **`resuelto-homónimo`** azul. La prosa (LLM) explica "el RFC y la fecha difieren; se resuelve como homónimo; no requiere acción AML más allá de conservar la evidencia".
5. **Clímax:** Lucía exporta el expediente (PDF) con el cotejo y el fundamento por escrito. Tiene la prueba de que revisó y por qué no actuó — exactamente lo que el auditor del Art. 12 Bis puede pedirle. Cambia de contexto a su siguiente cliente sin mezclar nada.

Falla: el RFC capturado **sí** coincide → banda pasa a `resuelto-real` rojo y arranca el reloj de aviso 24h; Vigía lo sube a pendientes y Lucía avisa al sujeto obligado responsable de presentar el aviso (Vigía no lo presenta por él).

### Flow 3 — El reloj de aviso de 24 horas (Don Refugio, una coincidencia real entra de noche)

El flujo de mayor consecuencia legal: del disparo del reloj a la evidencia de que se atendió a tiempo.

1. Un rescreening nocturno detecta que un cliente ya en el padrón cayó en la lista OFAC SDN (`CoincidenciaDetectada`). El motor arranca el **reloj de aviso 24h**, sellando `evento origen` y `marca de tiempo origen` (cuenta desde el evento, no desde que Don Refugio lo vea).
2. En la mañana, Don Refugio abre el dashboard: arriba, **un reloj rojo "18h 42m restantes"** con dot pulsante; el lector de pantalla anuncia "Aviso activo, menos de 24 horas". Es lo primero que ve.
3. Abre el Detalle: banda `resuelto-real`, tipo de lista `sanciones`, acción sugerida "no concretar la operación y presentar el aviso dentro de 24 horas", y la tarjeta del reloj con el botón **"Generar aviso XML (SPPLD)"**.
4. **Si el módulo de avisos está disponible:** genera el XML, Vigía lo valida (contra el XSD oficial o por validación estructural propia, ADR-0010), Don Refugio lo descarga, lo presenta él mismo en el portal SPPLD del SAT, y **registra el acuse** de vuelta en Vigía. **Si está bloqueado (AS-2, XSD sin confirmar):** el detalle muestra "Avisos XML: próximamente — pendiente de configuración legal" y, mientras tanto, Vigía deja la evidencia del hit y del reloj para que el sujeto obligado presente el aviso por su cuenta.
5. **Clímax:** al registrar el acuse (o la evidencia), el reloj se cierra y el expediente del sujeto evaluado suma el evento "aviso presentado · {fecha} · acuse {folio}". Meses después, ante un requerimiento del SAT, Don Refugio descarga el expediente y ahí está la cadena completa: coincidencia detectada → reloj → aviso → acuse, con sus tiempos. Cumplió, y puede probarlo.

Falla: las 24h se agotan antes de presentar → el reloj marca "Vencido" en rojo y lo registra como tal en el expediente (Vigía no oculta el incumplimiento; deja el rastro honesto). El producto alerta y cuenta; presentar a tiempo es responsabilidad del sujeto obligado.
