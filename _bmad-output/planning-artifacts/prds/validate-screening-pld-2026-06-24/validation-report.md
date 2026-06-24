# Validation Report — Motor de Screening PLD/FT para Sujetos Obligados (LFPIORPI México)

- **PRD:** `docs/prd_screening_pld.md`
- **Rubric:** `.claude/skills/bmad-prd/assets/prd-validation-checklist.md`
- **Reviewers:** rubric walker + adversarial-general + regulatorio-lfpiorpi
- **Run at:** 2026-06-24T06:36:19Z
- **Grade:** Fair

## Overall verdict

Como **documento**, este es un PRD fuerte: tesis estratégica explícita y honesta ("el screening no es el diferenciador"), decisiones de implementación cerradas y respaldadas por 9 ADRs coherentes, y un alcance declarado con rigor. El rubric walker lo calificaría como *Good* por sí solo (una sola dimensión *thin*, cero críticos de calidad documental).

La calificación baja a **Fair** porque las revisiones adversarial y regulatoria sí encontraron hallazgos **críticos** que no son cosméticos. Dos categorías dominan: **(1) viabilidad/estrategia** — el alcance "suite completa de 10 módulos en fase 1" contradice frontalmente la recomendación de "arranca delgado" que la propia fuente (el brief) da dos veces, y la regla de homonimia estrella ("RFC + fecha mata el falso positivo") colapsa en el caso de uso central (padrón Excel sin esos datos; listas OFAC/ONU que ni siquiera traen RFC mexicano); y **(2) coherencia de contrato** — el cuerpo del PRD aún usa el modelo de decisión viejo de 3 estados con "indeterminado" (prohibido por CONTEXT.md) en lugar de la `banda de confianza` de 4 estados + matriz `(banda × tipo de lista)` que ya viven en los ADRs, contaminando el contrato público `/v1/` y todo el downstream.

Ninguna dimensión está *broken* y todos los hallazgos son direccionables. La ruta a *Good/Excellent* es concreta: re-sincronizar el vocabulario del PRD con CONTEXT.md/ADR-0004, añadir criterios de aceptación + NFR + Success Metrics, decidir un fasing interno dentro de fase 1, y cerrar las dependencias declaradas "no bloqueantes" (licencia OpenSanctions, esquema XML del SAT, proveedor de identidad) que sí bloquean la viabilidad del canal que las dispara.

## Dimension verdicts

- Decision-readiness — **strong**
- Substance over theater — **strong**
- Strategic coherence — **strong**
- Done-ness clarity — **thin**
- Scope honesty — **strong**
- Downstream usability — **adequate**
- Shape fit — **strong**

## Findings by severity

### Critical (5)

**[Adversarial]** — C1 · El alcance contradice la recomendación explícita de la propia fuente (brief) (§ PRD L3 vs Brief §10 L216, §12 L234)
El brief dice dos veces "arranca delgado; BC y avisos XML como fase 2". El PRD invierte esa recomendación metiendo 7 módulos a fase 1, justificándolo solo como "decisión del propietario tras análisis de KYC Systems" — pero ese análisis es lo que el brief consideró antes de recomendar delgado. El PRD se contradice (L188 "competir hacia arriba diluye la cuña" vs L195 "brechas cerradas vs KYC Systems").
Fix: documentar el dato nuevo que justifica invertir la recomendación, o re-evaluar un fasing interno que arranque por la cuña.

**[Adversarial]** — C2 · Suite de 10 módulos en fase 1 para greenfield de un operador, sin fasing interno (§ PRD L3, L18–20, L134–157, L199)
Fase 1 = ~10 dominios nuevos + plomería transversal (multi-tenancy, auth, billing, inmutabilidad legal), tratados como entregable atómico sin secuenciación ni MVP interno. El módulo más débil define la credibilidad de todo el producto de cumplimiento.
Fix: añadir sección "qué construimos primero dentro de fase 1" con secuenciación y MVP interno verificable.

**[Adversarial]** — C3 · Cifras/artículos/plazos legales como hechos, con la verificación admitida pendiente (§ PRD L12, L56/65/113/115, L198)
Montos UMA, reforma marzo 2026, aviso 24h, ventana 6 meses, Arts. 12 Bis / 18 Fr. VII se afirman como hecho sin fuente verificable, mientras el PRD admite (L198) que "varias reglas de carácter general siguen publicándose". Se codifica lógica determinística sobre reglas no finales.
Fix: citar fuente (DOF) por afirmación; tratar reglas no finales como configuración versionada calibrable.

**[Adversarial]** — C4 · La regla de homonimia colapsa en el caso común (§ PRD L18, L138, L143; US-2 L30)
"RFC + fecha mata el falso positivo" pero US-2 admite "mínimo solo nombres". Padrón típico sin fecha/RFC limpio; listas OFAC/ONU/UE sin RFC mexicano → en el caso común todo cae en `requiere-verificación`. El diferenciador estrella se evapora en datos reales.
Fix: cuantificar la fracción de padrones reales con datos llave a ambos lados; ajustar el marketing del PRD; explorar señales secundarias de desempate (domicilio, nacionalidad, alias).

**[Regulatorio]** — C-1 · "indeterminado" como estado de decisión, prohibido por CONTEXT.md y ADR-0004 (§ PRD L138, L139, L166 vs CONTEXT L86, ADR-0004)
CONTEXT.md L86 prohíbe el término; ADR-0004 define 4 bandas sin "indeterminado". El campo `decision` es contrato público `/v1/` (L139): congelarlo con un enum prohibido fuerza un breaking change al alinear. "Indeterminado" no dice qué dato falta; `requiere-verificación` sí.
Fix: reemplazar "indeterminado" por las 4 bandas en L138/139/166; homonimia termina en `requiere-verificación` (score alto sin RFC/fecha) o `descartado-por-nombre` (score bajo). *(Misma raíz que el hallazgo high "Deriva del modelo de decisión" del rubric walker y H-1/M-1 regulatorios.)*

### High (10)

**[Done-ness clarity]** — Ninguna User Story tiene criterios de aceptación (§ User Stories L28–132)
Las 76 historias son enunciados de intención sin condición de done; la creación de stories tendrá que inventar los AC — en un producto regulatorio invita interpretaciones divergentes de la obligación legal.
Fix: añadir AC por historia/grupo, aprovechando los implícitos en Implementation/Testing Decisions; enlazar cada historia al campo del contrato / ADR que define su done.

**[Done-ness clarity]** — Umbrales reportables del monitoreo transaccional sin definir (§ US-63/64 L113–114, L151, ADR-0008)
El aviso 24h depende de "cruzar un umbral reportable" pero ni el PRD ni ADR-0008 dan el valor/regla por actividad vulnerable. Es la FR de mayor consecuencia legal y no es construible sin esa tabla.
Fix: incluir la tabla de umbrales por giro como configuración versionada, o marcar `[NOTE FOR PM: bloqueante]` y excluir la épica de monitoreo del primer corte hasta resolverla.

**[Downstream usability]** — Deriva del modelo de decisión entre PRD y glosario/ADR-0004 (§ L18, L138–139, L166 vs CONTEXT L78–86, ADR-0004)
El PRD usa "real/homónimo/indeterminado" (3 estados) y trata acción/riesgo como derivados del binario; el modelo canónico usa `banda de confianza` (4 estados) + matriz `(banda × tipo de lista)`. El campo `decision` hereda el enum viejo.
Fix: re-sincronizar Solution, regla de homonimia (L138), contrato (L139) y US-7/9/10/11 con la banda de confianza y la matriz de ADR-0004. *(Faceta downstream del crítico C-1.)*

**[Adversarial]** — H1 · Licencia OpenSanctions OEM/reseller sin cerrar, "no bloquea el build" es engañoso (§ PRD L192; Brief §11 L222)
El canal API/white-label de fase 1 es precisamente el caso reseller que más probablemente cae fuera del API de €0.10/consulta. Define si el costo variable es €0.10 o una licencia plana de coste desconocido.
Fix: cerrar por escrito con ventas de OpenSanctions antes de construir el canal white-label.

**[Adversarial]** — H2 · "El LLM solo redacta" no impide fuga de criterio entre prosa y verdicto (§ PRD L18, L137, L144, L161)
Si la prosa de Claude matiza/contradice el campo `decision` y los textos no se assertan (L161), el expediente "auditable" contiene una contradicción explotable. Aísla el cálculo, no la comunicación — y la comunicación es el producto.
Fix: añadir verificación de fidelidad prosa↔campos (el dictamen debe citar literalmente la banda y la acción; test que rechace prosa contradictoria).

**[Adversarial]** — H3 · Esquema XML oficial del SAT: dependencia no controlada tratada como disponible (§ PRD US-26 L62, L149)
El módulo de avisos (fase 1) promete validar contra "el esquema oficial del SAT" sin decir que lo tenga, dónde está, ni si es estable. Si el XML rebota en SPPLD, el cliente pierde el plazo legal.
Fix: confirmar existencia/estabilidad del XSD; añadir ADR de avisos XML (ver H-2 regulatorio).

**[Adversarial]** — H4 · El diferenciador defendible es delgado/replicable; el único foso real (Capafy) se difiere (§ PRD L194–195; Brief §10 L196)
Capafy (foso vacío) está en fase 2; dictamen LLM + homonimia es frágil (C4) y replicable; "cola larga" es segmento no foso. Se gasta fase 1 igualando features commodity en vez del único canal sin competencia.
Fix: re-sopesar adelantar Capafy o concentrar fase 1 en lo no-replicable.

**[Adversarial]** — H5 · Cambio a Claude/Bedrock (20-50x costo IA) sin re-derivar la economía unitaria (§ ADR-0007; PRD L193; Brief §9)
El brief modeló DeepSeek (~$0.05/corrida); el PRD cambia a Bedrock (~20-50x) pero no rehace el cálculo; "pocos dólares" reemplaza "$0.05", apoyado en caché manual no construida y pricing no verificado.
Fix: re-derivar el margen por corrida con pricing real de Bedrock y la fracción de hits que requieren prosa LLM.

**[Regulatorio]** — H-1 · El esquema del dictamen no incorpora la separación identidad↔consecuencia de ADR-0004 (§ PRD L139, L150 vs ADR-0004, CONTEXT L79)
El PRD colapsa todo en `decision` y no nombra `banda_confianza` ni `tipo_lista` como campos; riesgo de que un PEP escale a rechazo (prohibido por ADR-0004) o EFOS-EDOS a "no operar" automático.
Fix: añadir `banda_confianza` (4 estados) y `tipo_lista` al contrato L139; declarar que `accion_sugerida`/`nivel_riesgo` salen de la matriz, no de `decision`.

**[Regulatorio]** — H-2 · Validación del XML contra el esquema oficial del SAT: promesa sin decisión de implementación (§ PRD US-26 L62, L149, L172)
Se promete validar contra el XSD oficial sin decir de dónde sale, cómo se versiona ni cómo se refresca (a diferencia de SAT 69-B). No hay ADR de avisos XML.
Fix: añadir ADR-0010 de avisos XML (fuente del esquema, versionado, refresco, plan B sin XSD oficial).

### Medium (11)

**[Strategic coherence]** — Faltan Success Metrics y counter-metrics explícitos (§ todo el PRD)
Fix: añadir 2–4 métricas que validen la tesis (precisión real/homónimo, adopción Nivel 1→2, margen por corrida) + ≥1 counter-metric (falsos negativos, dictámenes plantilla por caída de LLM).

**[Done-ness clarity]** — No hay sección NFR con bounds (§ todo el PRD)
"Tiempo real"/"síncrono", batch async, conservación 10 años, inmutabilidad sin objetivos cuantificados (latencia p95, throughput, RPO/RTO, disponibilidad, tamaño máx. de padrón).
Fix: añadir sección NFR con esos bounds; son contractuales para el canal API/ERP.

**[Done-ness clarity]** — Adjetivos no testables en el dictamen y la validación (§ US-14 L44, Solution L14)
Fix: re-expresar el done del dictamen como "campos del contrato presentes y correctos" + plantilla de respaldo; bajar la prosa a no-objetivo de test.

**[Scope honesty]** — "Proveedor a definir" y umbrales "a calibrar" no indexados como pendientes (§ L153; ADR-0001/0005/0008)
Fix: consolidar un "Assumptions / Pendientes" con cada `a definir`/`a calibrar`, su impacto (bloquea épica X / no) y dónde se resuelve.

**[Downstream usability]** — "Por nombre" vs sujeto evaluado; "tenant"/"cliente" vs Cuenta/Sujeto obligado (§ L139, L143, L147, L52, L98 vs CONTEXT L7–22)
Fix: "por nombre"→"por sujeto evaluado", "tenant"→"Cuenta"/"Sujeto obligado" según corresponda, "cliente"→"Sujeto obligado".

**[Downstream usability]** — El PRD no contiene su propio glosario ni enlaza CONTEXT.md (§ encabezado L5)
El encabezado apunta al glosario "del brief" (vocabulario más viejo), no al canónico de CONTEXT.md.
Fix: apuntar el glosario explícitamente a `CONTEXT.md` como fuente canónica; retirar la lista del brief.

**[Adversarial]** — M2 · Inmutabilidad 10 años vs protección de datos (LFPDPPP/ARCO) no analizada (§ PRD L19, L51, L145; ADR-0002)
Un segundo régimen legal (protección de datos personales sensibles de terceros por 10 años) ignorado por completo.
Fix: añadir análisis de protección de datos (base de licitud, cómo se atienden ARCO sin romper inmutabilidad, retención).

**[Adversarial]** — M3 · ISO 27001 diferida, pero el canal de fase 1 (ERPs) la exige (§ PRD L196)
El canal de mayor leverage puede quedar muerto al arranque sin el sello que su comprador exige.
Fix: calendarizar la certificación contra el lanzamiento del canal ERP; whitepaper + AWS como puente, no sustituto.

**[Adversarial]** — M4 · Refresco automático de SAT 69-B asume fuente estable; falla en silencio (§ PRD L53, L99, L141)
Un cambio de URL/columnas rompe el refresco en silencio y el producto cruza contra versión vieja sellando "versión vigente".
Fix: añadir detección de staleness (alerta si no se refresca en N días).

**[Adversarial]** — M5 · BC/partes "declarados" = paridad de UI, no de valor antilavado (§ PRD L148, L152; ADR-0006)
"El cliente lo declara" → se screenea al testaferro declarado y se emite evidencia "limpia" de algo no verificado (falsa tranquilidad).
Fix: ser explícito sobre cuánto reduce el valor el "declarado, no resuelto"; no venderlo como paridad de valor antilavado.

**[Regulatorio]** — M-3 · US-30/US-33 dicen "identificar" el BC, que ADR-0006 deja fuera por diseño (§ PRD US-30/33 vs ADR-0006)
Fix: reformular a "registrar y conservar el BC que yo declaro".

*(También a nivel medium, fusionados con criticos/high por raíz compartida: Regulatorio M-2 "tenant" prohibido [→ ver M Downstream "tenant/cliente"]; Regulatorio M-1 "indeterminado" en seams de prueba [→ faceta del crítico C-1].)*

### Low (8)

**[Decision-readiness]** — Sin callouts `[NOTE FOR PM]` ni `[ASSUMPTION]` formales (§ todo el PRD) — Fix: convertir los "[CRÍTICO, pendiente]" y "a definir" en tags indexados.

**[Done-ness clarity]** — "Filas inválidas o duplicadas" sin regla (§ US-4 L32) — Fix: definir clave de deduplicación y validaciones mínimas.

**[Downstream usability]** — IDs de User Story sin prefijo de tipo (§ L28–132) — Fix: adoptar IDs estables con prefijo (`UJ-SO-01`) o confirmar la numeración corrida como ID contractual.

**[Adversarial]** — L1 · Diseñar el motor para Capafy (fase 2) desde fase 1 — YAGNI parcial (§ L4, L24, L136) — Fix: no sobre-generalizar el contrato `/v1/` por un canal diferido.

**[Adversarial]** — L2 · Volumen de `requiere-verificación` no dimensionado (§ US-2 L30; ADR-0001) — Fix: dimensionar la tasa de zona gris vía política de corte; instrumentarla.

**[Adversarial]** — L3 · Caché de prompt manual de Bedrock asumida como gratis (§ L144; ADR-0007) — Fix: validar el ahorro real antes de apoyar la economía en él.

**[Regulatorio]** — L-1 · El brief sigue describiendo DeepSeek; superseded por ADR-0007 sin anotar (§ Brief vs ADR-0007) — Fix: anotar en brief/Further Notes que la decisión de LLM fue superseded.

**[Regulatorio]** — L-2 · Valor del piso legal de rescreening pendiente (coherente entre docs) (§ ADR-0005, CONTEXT L46) — Fix: cerrar el número con el especialista PLD antes de fijar tiers comerciales.

## Mechanical notes

- **Glossary drift (la más material):** el cuerpo del PRD usa términos que CONTEXT.md marca en `_Avoid_` (`indeterminado`, `tenant`, "por nombre", "cliente" para el obligado) y nunca usa los términos canónicos refinados (`banda de confianza`, `sujeto evaluado`, `política de corte`, `constancia de corrida`, `tipo de lista`, `reloj de aviso`). Todo el modelo refinado vive solo en CONTEXT.md/ADRs.
- **ID continuity:** User Stories 1–76 sin gaps/duplicados pero sin prefijo de tipo; grupos "monitoreo" (62–66) y "partes relacionadas" (67–69) rompen el agrupamiento temático. Implementation Decisions son viñetas sin ID; no hay IDs de Success Metrics (no existen).
- **Cross-refs:** ADR-0001…0009 resuelven todas. "brief, Sección 5" no reverificado línea a línea. US-40 marca correctamente "(Fase 2 — canal Capafy)".
- **Assumptions Index roundtrip:** no existe sección consolidada ni tags `[ASSUMPTION]` inline; los pendientes están dispersos. No verificable porque el índice no existe.
- **Secciones requeridas:** presentes Problem/Solution/User Stories/Implementation/Testing/Out of Scope/Further Notes. Ausentes y esperables para chain-top de alto riesgo: Success Metrics (+counter-metrics), NFR con bounds, glosario embebido o enlace canónico a CONTEXT.md.

## Afirmaciones que requieren confirmación de especialista PLD externo

(Internamente consistentes entre PRD/CONTEXT/ADRs, pero no verificables solo con coherencia interna.)

1. Citas de artículos y su contenido (Art. 12 Bis, 7 Bis, 18 Fr. VII, 7, 45 Bis–Quinquies) tras la reforma marzo 2026.
2. Montos en UMAs y conversión a MXN 2026 (200–65,000 UMAs; UMA 2026 = $117.31; aritmética interna correcta).
3. Plazos: conservación 10 años, ventana 6 meses, aviso 24h y sus disparadores exactos.
4. Alcance de la reforma DOF 27/03/2026 (BC para toda sociedad, PEP para actividades vulnerables, avisos al SAT).
5. Existencia/estabilidad de un XSD oficial del SPPLD contra el cual validar.
6. Que la figura de oficial de cumplimiento certificado no se extendió a actividades vulnerables en la reforma.
7. Valor del piso legal de rescreening.

## Reviewer files

- `review-rubric.md`
- `review-adversarial-general.md`
- `review-regulatorio-lfpiorpi.md`
