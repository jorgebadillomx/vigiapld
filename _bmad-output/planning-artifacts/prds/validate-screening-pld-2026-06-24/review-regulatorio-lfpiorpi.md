# Reseña de coherencia regulatoria PLD/FT (LFPIORPI) — PRD Motor de Screening

**Fecha:** 2026-06-24
**Revisor:** Especialista en cumplimiento PLD/FT (revisión de coherencia interna; NO es opinión legal vinculante)
**Documentos revisados:** PRD `docs/prd_screening_pld.md`, glosario `CONTEXT.md`, los 9 ADRs (`docs/adr/0001`–`0009`), brief `docs/brief_prelavadodinero.md`.

---

## Veredicto

El PRD es regulatoriamente sólido en su posicionamiento ("herramienta de apoyo, no garantía") y mayormente coherente con los 9 ADRs, pero arrastra **deriva terminológica con CONTEXT/ADRs en la regla de homonimia ("indeterminado" prohibido)** y **una promesa no respaldada por diseño (validación de XML contra el esquema oficial del SAT sin decisión de implementación que la sustente)** que deben corregirse antes de pasar a build.

---

## A. Incoherencias internas confirmadas (no requieren especialista externo)

### CRITICAL

#### C-1. El PRD usa "indeterminado" como estado de decisión, contra la prohibición explícita de CONTEXT.md y ADR-0004
- **Ubicación:** PRD §Implementation Decisions, líneas 138 ("→ **indeterminado / requiere revisión**") y 139 (campo `decision` (real/homónimo/**indeterminado**)); también §Testing Decisions línea 166 ("real / homónimo / indeterminado").
- **Incoherencia:** CONTEXT.md (línea 86, entrada "Banda de confianza") prohíbe explícitamente el término: *"_Avoid_: Indeterminado (demasiado vago; usar `requiere-verificación` o `descartado-por-nombre` según el score)"*. ADR-0004 (línea 5) define la banda de confianza como `real / homónimo / requiere-verificación / descartado-por-nombre` — cuatro estados, sin "indeterminado". ADR-0001 introduce la zona gris que produce `requiere-verificación`. El PRD encoda el vocabulario viejo (tres estados con "indeterminado") que los ADRs/CONTEXT ya superaron.
- **Por qué importa:** el campo `decision` es parte del contrato público versionado `/v1/` (PRD línea 139). Congelar el contrato con un valor de enum prohibido por el glosario obliga a un cambio de contrato (breaking) en cuanto se alinee con los ADRs. Además debilita la defensa en auditoría: "indeterminado" no le dice al sujeto obligado qué dato falta, mientras `requiere-verificación` sí.
- **Fix:** reemplazar "indeterminado" por las cuatro bandas de ADR-0004/CONTEXT en las líneas 138, 139 y 166. La regla de homonimia (línea 138) debe terminar en `requiere-verificación` cuando faltan RFC/fecha con score alto, y `descartado-por-nombre` cuando el score es bajo (no en un único "indeterminado").

### HIGH

#### H-1. El PRD no incorpora la separación identidad↔consecuencia de ADR-0004 en el esquema del dictamen
- **Ubicación:** PRD línea 139 (campos del dictamen: `decision` (real/homónimo/indeterminado), `nivel_riesgo`, `accion_sugerida`); §EBR línea 150 ("consumiendo el nivel de riesgo del dictamen").
- **Incoherencia:** ADR-0004 separa explícitamente dos decisiones que "el PRD trataba como una sola": (1) **resolución de identidad** → produce la *banda de confianza*; (2) **consecuencia AML** → `accion_sugerida` y `nivel_riesgo` derivados de la matriz `(banda de confianza × tipo de lista)`. El PRD nunca usa el término "banda de confianza" (lo usa CONTEXT y ADR-0004); colapsa todo en un campo `decision` y trata `nivel_riesgo`/`accion_sugerida` como atributos sueltos sin nombrar la matriz. CONTEXT.md ("Tipo de lista", línea 79) es explícito: la acción y el riesgo "se derivan de la matriz `(banda de confianza × tipo de lista)`".
- **Por qué importa:** sin nombrar la matriz ni el `tipo_de_lista` como campo del contrato, el PRD deja implícito el punto regulatorio más delicado de ADR-0004: que un PEP dispara EDD, no rechazo; que EFOS-EDOS es factor de riesgo, no "no operar" automático. Un implementador podría derivar `accion_sugerida` del binario coincidió/no — exactamente el error que ADR-0004 prohíbe.
- **Fix:** alinear el esquema de la línea 139 con ADR-0004/CONTEXT: renombrar/añadir `banda_confianza` (4 estados) y `tipo_lista` (sanciones/bloqueados-UIF/PEP/EFOS-EDOS) como campos del contrato; declarar que `accion_sugerida` y `nivel_riesgo` salen de la matriz `(banda × tipo de lista)`, no de `decision`.

#### H-2. Promesa regulatoria sin decisión de implementación que la respalde: validación del XML contra el esquema oficial del SAT
- **Ubicación:** promesa en User Story 26 (línea 62: "validado contra el esquema oficial del SAT antes de descargarlo") y §Implementation Decisions línea 149 ("lo **valida contra el esquema oficial del SAT**"); §Testing Decisions línea 172 ("validación contra esquema oficial SAT").
- **Hueco de cobertura:** el PRD promete validación contra el esquema XSD/oficial del SPPLD pero **no hay decisión de implementación que diga de dónde sale ese esquema, cómo se versiona ni cómo se actualiza cuando el SAT lo cambia.** Compárese con SAT 69/69-B, para la cual sí hay decisión explícita de hospedaje + refresco con Hangfire (líneas 141, 145-146, 99). El esquema del aviso no tiene equivalente. No hay ADR sobre avisos XML (los 9 ADRs no cubren el módulo de avisos).
- **Por qué importa:** es una promesa regulatoria concreta ("no rechazarlo en el portal SPPLD", US-26) que el diseño actual no garantiza cumplir. Si el esquema oficial cambia (frecuente en reformas; el propio PRD admite que "varias reglas de carácter general siguen publicándose", línea 198) y no hay mecanismo de actualización, el XML "validado" se rechazará en el portal.
- **Fix:** añadir decisión de implementación (idealmente un ADR-0010 de avisos XML) que especifique: fuente del esquema oficial, su versionado, mecanismo de refresco, y qué pasa cuando el SAT no publica XSD (¿validación estructural propia? ¿la presentación sigue siendo responsabilidad del cliente — coherente con Out of Scope línea 181?). Marcar la dependencia del esquema como riesgo si no se puede obtener un XSD oficial estable.

### MEDIUM

#### M-1. Etiqueta de "indeterminado" también contamina los seams de prueba
- **Ubicación:** PRD §Testing Decisions línea 166 ("resolución de homonimia (real / homónimo / indeterminado)").
- **Incoherencia:** misma deriva que C-1, propagada a la especificación de pruebas. Si los tests se escriben asertando un estado "indeterminado", quedarán acoplados a un vocabulario que los ADRs prohíben.
- **Fix:** alinear con C-1; los casos de tabla deben asertar `requiere-verificación` y `descartado-por-nombre`, no "indeterminado".

#### M-2. El término "tenant" aparece en el PRD pese a estar prohibido por CONTEXT/ADR-0003
- **Ubicación:** PRD líneas 52, 98, 147 ("por tenant", "metering por tenant", "separación de datos por tenant"); ADR-0001 línea 3 ("no el cliente/tenant").
- **Incoherencia:** CONTEXT.md prohíbe "Tenant" en las entradas "Cuenta" (línea 9) y "Sujeto obligado" (línea 13), en favor de **Cuenta** (frontera de metering/facturación) y **Sujeto obligado** (frontera de aislamiento). ADR-0003 establece la jerarquía Cuenta → Sujeto obligado precisamente para no usar "tenant" como concepto plano. El PRD mezcla "tenant" con "cliente final" sin distinguir las dos fronteras de ADR-0003.
- **Por qué importa (regulatorio):** el aislamiento de la LPB confidencial y del expediente es **por sujeto obligado**, no "por tenant" (ADR-0003 línea 7). Usar "tenant" difumina dónde vive la frontera legal de aislamiento — relevante porque la LPB es confidencial y la notaría no debe ver la de la joyería de la misma cuenta-despacho.
- **Fix:** sustituir "tenant" por "Cuenta" (cuando se habla de metering/auth/facturación, líneas 52, 98) y por "Sujeto obligado" (cuando se habla de aislamiento de datos/padrón/LPB, línea 147), conforme a ADR-0003 y CONTEXT.

#### M-3. "Beneficiario controlador" en User Story 33 promete cobertura de sociedad sin actividad vulnerable que el módulo no resuelve
- **Ubicación:** US-33 (línea 71) "Como sociedad mercantil sin actividad vulnerable, quiero identificar y conservar a mi Beneficiario Controlador…"; cruzado con ADR-0006 (BC declarado, sin resolución recursiva) y §Implementation Decisions línea 148.
- **Coherencia parcial / riesgo de expectativa:** ADR-0006 es explícito en que el producto **solo screenea al BC declarado** y conserva la cadena de control como evidencia; **no resuelve la cadena hasta la persona física última**. US-33 está alineada con eso si se lee como "registrar/conservar", pero el enunciado "identificar… a mi Beneficiario Controlador" puede leerse como que el producto *identifica* (resuelve) el BC, que es justo lo que ADR-0006 deja fuera por diseño. La obligación legal de *identificar* al BC es del sujeto obligado (ADR-0006 línea 5).
- **Fix:** reformular US-33 y US-30 para que digan "registrar y conservar el BC que yo declaro" (no "identificar"), dejando explícito —como ya hace el posicionamiento legal— que la identificación de la cadena es responsabilidad del sujeto obligado, coherente con ADR-0006.

### LOW

#### L-1. El brief sigue describiendo DeepSeek en toda la arquitectura, costos y stack; el PRD/ADR-0007 lo revirtieron a Claude Sonnet en Bedrock
- **Ubicación:** brief §5 (líneas 68, 71-72, 78, 80), §6 (líneas 95, 103, 113), §9 (líneas 145-146, 152), §12 (líneas 237, 240); vs. PRD línea 144 y ADR-0007 (Claude Sonnet 4.6 en AWS Bedrock).
- **Incoherencia:** el brief —fuente declarada del PRD (línea 5)— describe DeepSeek (Flash + Pro, "doble IA", `api.deepseek.com`, key en bóveda de Capafy) como el LLM de redacción. ADR-0007 supersede esa decisión y el PRD la sigue. Es deriva esperable (un ADR supersede al brief), pero **el brief no está anotado como obsoleto en este punto**, y el PRD cita el brief como fuente sin advertir la divergencia. Tampoco hay impacto regulatorio directo, pero sí de trazabilidad documental.
- **Fix:** anotar en el brief (o en el PRD §Further Notes) que la decisión de LLM fue superseded por ADR-0007 (Claude Sonnet en Bedrock), para que el lector no asuma que DeepSeek sigue vigente. El argumento de ADR-0007 (residencia de datos AWS más defendible ante auditoría que API de terceros genérica) es además un punto regulatorio a favor del cambio — vale la pena conservarlo visible.

#### L-2. Falta de número de piso legal del rescreening (coherente entre docs, pero pendiente declarado)
- **Ubicación:** ADR-0005 línea 12 ("El valor numérico del piso se calibra con el especialista PLD (pendiente)"); CONTEXT "Cadencia de rescreening" línea 46 ("piso legal mínimo"); PRD US-23/US-20 y §Implementation línea 146.
- **Observación:** los documentos son internamente consistentes (todos remiten el valor numérico al especialista). No es incoherencia; se marca para que el especialista PLD lo cierre antes de fijar tiers comerciales, porque un piso mal calibrado deja descubierto a un cliente de plan básico (exposición legal que el propio ADR-0005 reconoce).
- **Fix:** ninguno en el PRD; cerrar el número con el especialista (ver Sección B).

---

## B. Afirmaciones que requieren confirmación de especialista PLD externo

Estas NO son incoherencias internas; el PRD las trata como hechos legales que necesitan validación de un especialista PLD/abogado (varias el propio PRD ya marca como pendientes, líneas 198, 138 del brief).

1. **Citas de artículos y su contenido.** El PRD atribuye: Art. 12 Bis = dictamen/auditoría del programa PLD; Art. 7 Bis = aviso 24h por intento de operación; Art. 18 Fr. VII = EBR (5 dimensiones); Art. 7 = acumulación en periodos de 6 meses; Arts. 45 Bis–Quinquies = capítulo PEP (solo en el brief, línea 24). Internamente son **consistentes** entre PRD, CONTEXT y ADRs (mismas atribuciones en todos los docs). **Requiere confirmación externa** de que cada artículo dice lo que se le atribuye tras la reforma de marzo 2026 — el PRD mismo advierte que "varias reglas de carácter general siguen publicándose" (línea 198).

2. **Montos en UMAs y su conversión a MXN 2026.** PRD línea 12 y brief línea 22 coinciden: 200–65,000 UMAs ≈ $23,462–$7,625,150 MXN, con UMA 2026 = $117.31 diarios (solo el brief explicita el valor diario). La aritmética interna es consistente (200 × 117.31 = $23,462; 65,000 × 117.31 = $7,625,150). **Requiere confirmación** del valor de la UMA 2026 y de que el rango sancionatorio aplicable a las actividades vulnerables objetivo es efectivamente 200–65,000 UMAs.

3. **Plazos:** conservación 10 años, ventana de acumulación 6 meses, aviso 24h. Consistentes en todos los documentos (PRD líneas 12, 50, 56, 65, 113; CONTEXT "Reloj de aviso", "Operación"; ADR-0002, ADR-0008). **Requiere confirmación** de los plazos exactos y de los disparadores precisos del aviso 24h tras la reforma.

4. **Alcance de la reforma marzo 2026 (DOF 27/03/2026):** Beneficiario Controlador para toda sociedad mercantil, obligatoriedad de PEP para actividades vulnerables, avisos al SAT. El brief (líneas 23-29) da la fecha del DOF y el detalle; el PRD lo asume. **Requiere confirmación** de la fecha del DOF, vigencia y términos finos (el brief ya lo marca como pendiente, línea 29).

5. **Esquema oficial del XML del SPPLD/SAT** (ver H-2): además del hueco de diseño, **requiere confirmación** de que existe un XSD/esquema oficial publicado y estable contra el cual validar, o si la validación debe ser estructural propia.

6. **Figura de oficial de cumplimiento certificado:** el PRD (Out of Scope línea 182) y el brief (línea 137) afirman que NO aplica a actividades vulnerables (solo sector financiero). Consistente entre docs; **requiere confirmación** de que la reforma 2026 no extendió alguna figura equivalente a ciertas actividades vulnerables.

7. **Valor del piso legal de rescreening** (ver L-2): cerrar el número con el especialista.

---

## C. Aspectos coherentes verificados (sin hallazgo)

- **Posicionamiento legal "herramienta de apoyo, no garantía"** — consistente en los tres módulos sensibles: avisos (PRD línea 157, 181; CONTEXT "Aviso"/"Reloj de aviso" líneas 49, 59), validación de identidad (ADR-0009, PRD línea 153, Out of Scope línea 186) y dictamen/12 Bis (Out of Scope línea 180; brief §8 línea 136). El producto alerta/cuenta/genera evidencia; la presentación y la responsabilidad legal son del sujeto obligado. **Coherente.**
- **El LLM solo redacta, nunca decide lo legalmente sensible** — consistente entre PRD (líneas 18, 137, 144, 151), CONTEXT ("Dictamen" línea 27) y ADR-0004/ADR-0007/ADR-0008. **Coherente.**
- **LPB confidencial, la sube el cliente, nunca se hospeda** — consistente en PRD (líneas 100, 141, 183), CONTEXT (ADR-0003), ADR-0003 línea 7, ADR-0005, brief §5/§6. **Coherente.**
- **Rastro event-sourced inmutable / conservación 10 años / sello de versión de lista** — PRD (líneas 145, 169) ↔ ADR-0002 ↔ ADR-0005 (sello de versión) ↔ CONTEXT ("Expediente", "Constancia de corrida"). **Coherente.**
- **Monitoreo transaccional determinístico, IA fuera de fase 1** — PRD (línea 151, Out of Scope 187) ↔ ADR-0008 ↔ CONTEXT ("Operación"). **Coherente.**
- **Detalle técnico del LLM (Claude Sonnet 4.6, id `anthropic.claude-sonnet-4-6`, cliente `AnthropicBedrockMantleClient` / paquete `Anthropic.Bedrock`, endpoint Messages)** — PRD línea 144 ↔ ADR-0007. Verificado contra la referencia oficial de Bedrock: el prefijo `anthropic.` y el cliente Mantle son correctos para el endpoint Messages de Bedrock. **Coherente y técnicamente correcto.** (Nota menor: el id de modelo no debe llevar sufijo de fecha en Bedrock — el PRD/ADR ya lo escriben sin sufijo, correcto.)
