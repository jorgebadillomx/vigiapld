# Cynical Review — PRD Motor de Screening PLD/FT (LFPIORPI)

> **Veredicto:** Un PRD bien escrito y arquitectónicamente coherente que esconde su mayor riesgo a plena vista: pivotó de "cuña delgada" (lo que su propia fuente recomienda dos veces) a "suite completa de 10 módulos en fase 1" para un greenfield de un solo operador, apoyándose en cifras legales sin fuente, una regla de homonimia que se desmorona en el caso de datos común, y dependencias críticas (licencia OEM, esquema SAT, proveedor de identidad, reglas de la reforma) explícitamente sin cerrar — y declara que ninguna "bloquea el build".

---

Material revisado: `docs/prd_screening_pld.md`, `CONTEXT.md`, `docs/brief_prelavadodinero.md`, ADR-0001 a 0009.

Nota de método: el PRD es internamente consistente y los ADRs son sólidos. Las objeciones de abajo no atacan la prosa; atacan las **suposiciones de viabilidad, las afirmaciones de hecho sin fuente, y el riesgo de alcance** que el documento presenta como neutralizado.

---

## CRITICAL

### C1 — La fuente del PRD recomienda explícitamente lo contrario al alcance elegido
**Ubicación:** PRD línea 3 ("suite completa, decisión del propietario") vs. Brief §10 línea 216 y §12 línea 234.
**Objeción:** El brief —la fuente declarada del PRD— dice literalmente dos veces: *"Recomendación: arranca delgado y diferenciado por canal; BC y avisos XML como fase 2"* (§10) y *"Cerrar la decisión de alcance... Recomendado: delgada en fase 1"* (§12, paso 1). El PRD invierte esa recomendación y mete BC, avisos XML, EBR, monitoreo transaccional, partes relacionadas, programa PLD y validación de identidad **todos a fase 1**, justificándolo solo como "decisión del propietario (jun-2026) tras análisis competitivo de KYC Systems". El brief ya consideró ese análisis competitivo (la Sección 10 ES el análisis de KYC Systems) y aun así recomendó delgado.
**Por qué importa:** Una decisión que contradice frontalmente la recomendación de su propia fuente de análisis, sin nuevo dato que la justifique, no es una decisión informada — es una racionalización. El PRD presenta "brechas cerradas vs. KYC Systems" (línea 195) como si igualar features a un incumbente con +clientes fuera estrategia ganadora; el propio brief advierte que eso es "competir hacia arriba y diluye la cuña" (línea 188, contradiciéndose a sí mismo en el mismo documento). El riesgo no es que la suite sea mala idea — es que se eligió el camino que la fuente marcó como el error.

### C2 — "Suite completa en fase 1" para greenfield de un operador es un alcance que muerde más de lo que puede masticar
**Ubicación:** PRD líneas 3, 18-20, 134-157; "Greenfield" línea 199.
**Objeción:** Fase 1 incluye, en código nuevo desde cero: motor de matching + score + reglas, capa LLM, ingesta Excel con validación, resolución de homonimia, expediente event-sourced inmutable a 10 años, rescreening con Hangfire + clock inyectable, BC, partes relacionadas, monitoreo transaccional con ventanas móviles de 6 meses, generación + validación de XML contra esquema SAT, matriz EBR de 5 dimensiones, validación de identidad por integración, programa PLD (manual + capacitación + segregación de roles + auditoría interna), gateway multi-tenant con metering/rate-limits/API keys/webhooks, portal React completo, y canal API/white-label versionado. Son ~10 dominios funcionales más toda la plomería transversal (multi-tenancy, auth, billing, inmutabilidad legal). El "Greenfield" (línea 199) admite que no existe ni repo git.
**Por qué importa:** Cada módulo "suite" tiene su propio pozo de complejidad regulatoria (el XML SPPLD depende de un esquema oficial que el PRD no tiene; ver C5). Un solo operador construyendo 10 dominios a la vez no llega a fase 1 — llega a 10 medio-implementaciones donde el módulo más débil (probablemente el XML o el monitoreo de ventanas) define la credibilidad de todo el producto de cumplimiento. El PRD no incluye fasing interno, ni secuenciación, ni MVP dentro de fase 1; trata los 10 módulos como un solo entregable atómico. No hay una sección de "qué construimos primero dentro de fase 1".

### C3 — Afirmaciones legales presentadas como hechos verificados, con la verificación explícitamente pendiente
**Ubicación:** PRD línea 12 (montos UMA, artículos), líneas 56/65/113/115 (plazo 24h, ventana 6 meses), línea 198 ("varias reglas de carácter general siguen publicándose"); Brief línea 29 ("Términos exactos pendientes de reglas de carácter general").
**Objeción:** El PRD afirma como hecho: multa "$23,462 a $7,625,150 MXN (200–65,000 UMAs, 2026)", "reforma de marzo 2026", "aviso de 24 horas (Art. 7 Bis)", "acumulación en periodos de 6 meses (Art. 7)", "Art. 12 Bis", "Art. 18 Fr. VII", "Arts. 45 Bis–Quinquies". Ninguna cita una fuente verificable (no hay link al DOF, ni a la ley, ni número de publicación más allá de "DOF 27/03/2026" en el brief). Y en la misma "Further Notes" el PRD admite (línea 198) que los "términos finos de la reforma 2026 (varias reglas de carácter general siguen publicándose)" están pendientes de confirmar con especialista. El brief es aún más explícito: la obligación "ya es exigible" pero los términos exactos están pendientes.
**Por qué importa:** Todo el producto se construye sobre estos números y plazos: el reloj de 24h, la ventana de 6 meses, los umbrales reportables, la matriz EBR de 5 dimensiones "oficiales". Si la ventana es de 6 meses *naturales* vs. *móviles* (el PRD asume móviles, línea 113), o si el umbral por actividad vulnerable cambia en la regla de carácter general aún no publicada, el módulo de monitoreo transaccional —construido en fase 1— está codificando reglas que todavía no existen en forma final. Construir lógica determinística "legalmente sensible" sobre reglas que admites que siguen publicándose es construir sobre arena, y un producto de cumplimiento que cita un artículo equivocado o un monto desactualizado pierde toda su credibilidad de venta.

### C4 — La regla de homonimia "RFC + fecha de nacimiento mata el falso positivo" colapsa precisamente en el caso de uso central (Excel sin esos datos)
**Ubicación:** PRD líneas 18, 138, 143; User Story #2 (línea 30, "mínimo solo nombres"); Brief línea 120 ("Mínimo nombres; idealmente + RFC + fecha"); CONTEXT.md "Banda de confianza".
**Objeción:** La regla central de decisión es: nombre + (RFC **y** fecha iguales) → real; nombre coincide pero RFC/fecha no → homónimo; **faltan RFC/fecha → indeterminado/`requiere-verificación`**. El PRD admite en US#2 que el sistema "acepte un mínimo de solo nombres". Combinemos: un padrón Excel típico de una joyería o casa de empeño **no trae fecha de nacimiento, y muchas veces ni RFC limpio** de clientes ocasionales. En ese padrón real, la regla estrella nunca dispara su rama "mata el falso positivo" — todo cae en `requiere-verificación`. El producto que se vende como "resuelve la homonimia, tu dolor #1" devuelve, en el caso común, "no puedo resolverlo, revísalo tú". Peor: las listas de sanciones de OpenSanctions frecuentemente **tampoco** traen RFC ni fecha de nacimiento mexicana (son listas OFAC/ONU/UE), así que aun cuando el cliente tenga el dato, no hay contra qué cruzarlo. El RFC mexicano no existe en la lista SDN de OFAC.
**Por qué importa:** Esto es el corazón del "diferenciador" (líneas 194-195, 207). Si en el flujo real de "sube Excel → dictamen" la mayoría de filas salen `requiere-verificación`, el diferenciador se evapora y el producto queda como un buscador de listas más (exactamente lo que el brief advierte que ya hacen 6 competidores). El PRD presenta la regla como robusta ("mata el falso positivo", línea 143) sin cuantificar en qué fracción de los padrones reales los datos llave existen del lado del cliente **y** del lado de la lista. La banda `requiere-verificación` es honesta en el ADR-0004, pero el marketing del PRD (líneas 18, 207) la oculta tras la afirmación absoluta.

---

## HIGH

### H1 — La economía depende de una licencia OpenSanctions que el modelo de negocio (Capafy/white-label = revender a muchos) probablemente viola, y se declara "no bloquea el build"
**Ubicación:** PRD línea 192 ("[CRÍTICO, pendiente, no bloquea el build]"); Brief §11 riesgo 1 (línea 222, "Es el mayor swing del ROI").
**Objeción:** El propio PRD marca esto CRÍTICO y dice que es "el mayor swing del ROI", y aun así lo etiqueta "no bloquea el build". La pregunta sin resolver: ¿revender dictámenes a muchos clientes finales vía white-label/Capafy cabe en los términos del API hospedado (€0.10/consulta) o exige licencia OEM/reseller (cotización, "más caro", brief línea 160)? El canal API/white-label —que el PRD pone en fase 1— es **precisamente** el caso de uso reseller que más probablemente cae fuera de los términos del API simple. No es un detalle que se resuelve después: define si el costo variable es €0.10/consulta o una licencia plana reseller de coste desconocido.
**Por qué importa:** "No bloquea el build" es cierto en sentido literal (puedes escribir código tras un puerto sin la respuesta) pero engañoso en sentido de negocio: puedes construir todo el canal white-label y descubrir que su modelo de monetización es contractualmente inviable con tu proveedor de datos, o que el margen es negativo a la licencia reseller. Diferir la pregunta de costo de la dependencia más cara hasta después de construir el canal que la dispara es invertir el orden del riesgo.

### H2 — "El LLM solo redacta" no aísla el riesgo legal tan limpiamente como se afirma — hay fuga de criterio por la prosa
**Ubicación:** PRD líneas 18, 137, 144; ADR-0004; ADR-0007; CONTEXT.md "Dictamen".
**Objeción:** El PRD repite como mantra que la decisión real/homónimo la toman las reglas y "el LLM **solo redacta**" (líneas 18, 137), por lo tanto el riesgo legal queda aislado. Dos fugas: (1) El dictamen es el entregable que el sujeto obligado lleva a su auditor; si el LLM redacta prosa que **matiza, contradice o sobre-interpreta** el campo determinístico `decision` (p. ej. el campo dice `requiere-verificación` pero la prosa de Claude dice "no se encontró coincidencia relevante"), el texto que el humano lee y archiva diverge del verdicto estructurado. El PRD dice que no se assertan los textos del LLM en tests (línea 161) — entonces nada garantiza que la prosa sea fiel al campo. (2) El campo `accion_sugerida` (rechazar/EDD/reportar) es criterio legal incrustado en una matriz; presentarlo en prosa redactada por LLM lo hace ver como recomendación profesional. El disclaimer "herramienta de apoyo, no garantía" (línea 157) es la única defensa, y los disclaimers rara vez aguantan cuando un cliente actuó sobre una "acción sugerida" redactada en español autoritativo.
**Por qué importa:** El producto entero se vende sobre "evidencia auditable que aguanta el 12 Bis". Si la prosa del LLM puede divergir del verdicto determinístico y nadie lo testea, el expediente "auditable" contiene una contradicción interna que un auditor adversarial (o un litigante) puede explotar. "El LLM solo redacta" aísla el *cálculo*, no la *comunicación* — y la comunicación es el producto.

### H3 — El esquema XML oficial del SAT/SPPLD es una dependencia externa no controlada tratada como si estuviera disponible
**Ubicación:** PRD líneas 26 (US#26 "validar contra el esquema oficial del SAT"), 62, 149 ("valida contra el esquema oficial del SAT").
**Objeción:** El módulo de avisos XML (fase 1) promete generar y **validar contra el esquema oficial del SAT** antes de descargar. El PRD nunca dice que tenga ese esquema, dónde está, qué versión, ni si es público y estable. El SPPLD/SITI son portales de la UIF/SAT con esquemas que cambian y que históricamente han sido mal documentados. Construir un generador-validador de XML contra un esquema que no tienes en mano es un compromiso de fase 1 apoyado en un artefacto externo de disponibilidad no confirmada — y el PRD lo lista entre los "pendientes" solo de refilón (no aparece en Further Notes como riesgo nombrado, a diferencia de OpenSanctions).
**Por qué importa:** Si el XML rebota en el portal SPPLD por una discrepancia de esquema, el cliente pierde el plazo legal (el aviso 24h o el reportable) y la culpa percibida es del producto. Es un módulo de alto riesgo regulatorio y baja controlabilidad metido en fase 1 sin que su dependencia clave esté asegurada.

### H4 — El "diferenciador" defendible es delgado y el propio análisis admite que es replicable
**Ubicación:** PRD líneas 194-195; Brief §10 línea 196 ("un competidor podría envolver cualquiera de estas APIs igual que tú").
**Objeción:** Los diferenciadores listados: (a) distribución Capafy — **diferida a fase 2**, o sea ausente del producto que se construye; (b) dictamen LLM + homonimia — frágil en datos reales (ver C4) y replicable por cualquiera con una API LLM; (c) "cola larga de sujetos obligados chicos" — un segmento, no un foso; (d) "API mexicana lista para LFPIORPI" — el brief mismo dice (línea 196) que "un competidor podría envolver cualquiera de estas APIs igual que tú; por eso el dato no defiende nada", y lo mismo aplica al wrapper de dictamen; (e) precio por volumen — los competidores también pueden bajar precio. El único foso genuinamente vacío hoy (Capafy, brief línea 202) es justo el que el PRD saca de fase 1.
**Por qué importa:** El PRD construye una suite completa "para competir de frente con KYC Systems" (líneas 195) pero su ventaja real más limpia (canal Capafy) no está en fase 1, y las demás son igualables. Estás gastando el esfuerzo de fase 1 en igualar features commodity (suite) en lugar de en el único canal sin competencia. Es la estrategia invertida: construyes el cuerpo del me-too y difieres la cabeza diferenciada.

### H5 — Lock-in con AWS Bedrock + cambio de LLM (DeepSeek → Claude) infla el costo de IA 10-50x sin recalcular la economía unitaria
**Ubicación:** ADR-0007; PRD línea 193; Brief §9 líneas 145-153.
**Objeción:** El brief modeló la economía con DeepSeek (Flash $0.14/$0.28, Pro ~$1.74/$3.48, corrida de 500 = ~$0.05 de IA). El PRD cambia a Claude Sonnet 4.6 en Bedrock (~$3/$15 por 1M, línea 193) — eso es ~20x el input y ~50x el output vs. DeepSeek Flash. El PRD afirma que "sigue muy por debajo de OpenSanctions" y "queda en pocos dólares", pero **no rehace el cálculo unitario** que el brief sí hacía; solo asegura cualitativamente que es menor que OpenSanctions. Además admite (líneas 144, 196, 198) que el pricing regional de Bedrock está sin verificar y que la caché automática no existe en Bedrock (hay que hacerla manual, línea 144/ADR-0007), lo que reduce el ahorro asumido.
**Por qué importa:** La "regla de oro" del negocio es precio por volumen porque el margen es delgado. Cambiar el componente de IA por uno 20-50x más caro sin re-derivar el margen por corrida, y apoyándose en una caché manual no construida y un pricing no verificado, deja la economía unitaria —la justificación central del modelo de precios— sin actualizar. "Pocos dólares" no es un número; el brief tenía "$0.05".

---

## MEDIUM

### M1 — Validación de identidad con "proveedor a definir" en fase 1 es un compromiso sin contraparte
**Ubicación:** PRD línea 153; ADR-0009 ("proveedor a definir"); User Stories 74-76.
**Objeción:** El nivel barato (RFC/CURP/69-B) entra en fase 1, bien. Pero las US 75-76 y la línea 153 prometen INE/biometría/RENAPO "por integración con proveedor a definir". Comprometer una user story de fase 1 a un proveedor que no existe aún (cobertura, precio, contrato, SLA desconocidos) es prometer una integración contra una caja vacía. El puerto está bien diseñado, pero un puerto sin implementación no entrega la user story.
**Por qué importa:** Si un cliente de mayor riesgo necesita la validación INE/biometría y no hay proveedor contratado, la promesa de fase 1 no se cumple. Debería estar inequívocamente en fase 2 o marcado como "puerto sin implementación en fase 1".

### M2 — La inmutabilidad "a 10 años" choca con derechos de borrado y rectificación (LFPDPPP)
**Ubicación:** PRD líneas 19, 51, 145; ADR-0002 (log append-only).
**Objeción:** El PRD diseña el expediente como inmutable append-only por 10 años (correcto para auditoría). Pero almacena datos personales sensibles de los sujetos evaluados (nombre, RFC, fecha de nacimiento, resultado de listas, biometría eventual). La ley mexicana de protección de datos (LFPDPPP) y derechos ARCO pueden chocar con "nada se borra nunca". El PRD no menciona en ningún lado cómo concilia inmutabilidad regulatoria PLD con obligaciones de protección de datos. No aparece la palabra "datos personales" ni "ARCO" ni "LFPDPPP".
**Por qué importa:** Un producto que almacena datos personales sensibles de terceros (los clientes de sus clientes) por 10 años sin un análisis de protección de datos es un riesgo de cumplimiento de un *segundo* régimen legal que el PRD ignora por completo. Para un SaaS que quiere vender ISO 27001 como señal de confianza (línea 196), omitir el régimen de datos personales es una laguna grande.

### M3 — ISO 27001 tratada como "no bloquea" pero el canal de fase 1 (API/white-label/ERPs) es justo el que la exige
**Ubicación:** PRD línea 196.
**Objeción:** El PRD dice que ISO 27001 "no es requisito legal" y se persigue "cuando el canal API/ERP la exija". Pero el canal API/white-label está en **fase 1**, y el propio párrafo admite que "su due-diligence suele pedirla; KYC Systems la tiene". O sea: el canal que requiere la certificación se lanza antes de tenerla.
**Por qué importa:** El gancho del canal de mayor apalancamiento (ERPs como Contpaqi/Aspel/SAP B1) es justo donde la due-diligence de seguridad bloquea la venta sin certificación. Lanzar el canal sin el sello que su comprador exige puede dejar el canal de mayor leverage muerto al arranque. La mitigación (heredar controles de AWS + whitepaper) es razonable pero el PRD subestima que para un ERP serio eso no sustituye el certificado.

### M4 — Hospedar y refrescar SAT 69-B "automáticamente" asume una fuente estable que históricamente no lo es
**Ubicación:** PRD líneas 99, 141, 53 (US#53); Brief línea 100 ("~trimestral").
**Objeción:** El operador promete hospedar y refrescar automáticamente con Hangfire la lista SAT 69/69-B. El SAT publica esa lista como Excel/CSV en un portal de datos abiertos cuya estructura, URL y cadencia ("~trimestral") cambian sin aviso. Un job automático contra un Excel publicado a mano por una dependencia gubernamental es frágil: un cambio de columnas o de URL rompe el refresco silenciosamente, y el producto sigue cruzando contra una versión vieja sin que nadie lo note.
**Por qué importa:** El expediente sella la versión de lista (bien), pero si el refresco automático falla en silencio, el sello dice "versión vigente" cuando no lo es, y el cumplimiento del cliente queda comprometido sin alerta. Necesita detección de staleness, no solo un cron job optimista.

### M5 — Partes relacionadas y BC "declarados, no resueltos" trasladan el trabajo difícil al cliente y debilitan la promesa
**Ubicación:** PRD líneas 148, 152; ADR-0006; CONTEXT.md "Partes relacionadas".
**Objeción:** Decisión defendible técnicamente (no hay fuente abierta del registro mercantil MX), pero el PRD la presenta como feature de paridad con KYC Systems sin reconocer que "el cliente lo declara" significa que la calidad del screening de BC/partes relacionadas es tan buena como lo que el cliente teclee. Si el sujeto obligado no sabe o no declara al verdadero beneficiario controlador (que es justo el caso que la regulación quiere atrapar — el dueño escondido), el producto screenea a un testaferro declarado y emite evidencia "limpia" de algo que no se verificó.
**Por qué importa:** Es paridad de feature en la UI, no paridad de valor antilavado. La evidencia generada puede dar falsa tranquilidad: "screeneamos al BC" cuando en realidad "screeneamos a quien el cliente dijo que era el BC". El límite de alcance es correcto, pero el PRD no es honesto sobre cuánto reduce el valor del módulo.

---

## LOW

### L1 — "Tres canales, el motor diseñado para los tres desde fase 1" añade costo de diseño por un canal diferido
**Ubicación:** PRD líneas 4, 24, 136.
**Objeción:** Se insiste en diseñar el contrato `/v1/` para soportar Capafy desde fase 1 aunque Capafy sea fase 2. Diseñar para un canal que no se construye es especulación arquitectónica; el contrato podría sobre-generalizarse para un consumidor (Capafy) cuyos requisitos reales no se conocen hasta construirlo.
**Por qué importa:** Riesgo bajo (el contrato API es útil de todos modos), pero es trabajo extra justificado por un canal diferido. YAGNI parcial.

### L2 — "Mínimo solo nombres" + matching difuso multiplica el volumen de `requiere-verificación` y por tanto el trabajo manual del cliente
**Ubicación:** PRD US#2 (línea 30), ADR-0001 (zona gris).
**Objeción:** La zona gris (`requiere-verificación`) es buena práctica, pero combinada con padrones de solo-nombre (C4) puede producir un volumen alto de casos que el cliente debe revisar a mano — exactamente el trabajo manual que el producto promete eliminar. El PRD no estima la tasa esperada de zona gris.
**Por qué importa:** Si el 40% de un padrón cae en `requiere-verificación`, la propuesta de valor "sube Excel y listo" se erosiona en práctica. Bajo porque es ajustable vía política de corte, pero no está dimensionado.

### L3 — Caché de prompt "manual" en Bedrock es trabajo no trivial asumido como dado
**Ubicación:** PRD línea 144; ADR-0007.
**Objeción:** El ahorro de costo de IA depende de caché de prompt manual de Bedrock; el PRD lo menciona como si fuera gratis. La caché manual de Bedrock tiene reglas (tamaño mínimo de prefijo, TTL, puntos de quiebre) que requieren ingeniería y no siempre aplican a prompts cortos de redacción.
**Por qué importa:** Menor, pero contribuye a H5: el ahorro asumido para justificar el LLM caro no es automático.

---

## Resumen de severidad

| # | Severidad | Hallazgo |
|---|-----------|----------|
| C1 | Critical | El alcance contradice la recomendación explícita de la propia fuente (brief) |
| C2 | Critical | Suite de 10 módulos en fase 1 para greenfield de un operador, sin fasing interno |
| C3 | Critical | Cifras/artículos/plazos legales como hechos, con la verificación admitida como pendiente |
| C4 | Critical | La regla de homonimia colapsa en el caso común (Excel sin RFC/fecha; listas sin RFC MX) |
| H1 | High | Licencia OpenSanctions OEM/reseller sin cerrar, "no bloquea el build" es engañoso |
| H2 | High | "El LLM solo redacta" no impide fuga de criterio entre prosa y verdicto |
| H3 | High | Esquema XML oficial del SAT: dependencia no controlada tratada como disponible |
| H4 | High | El diferenciador defendible es delgado/replicable; el único foso real (Capafy) se difiere |
| H5 | High | Cambio a Claude/Bedrock (20-50x costo IA) sin re-derivar la economía unitaria |
| M1 | Medium | Validación de identidad fase 1 con "proveedor a definir" |
| M2 | Medium | Inmutabilidad 10 años vs. protección de datos (LFPDPPP/ARCO) no analizada |
| M3 | Medium | ISO 27001 diferida pero el canal de fase 1 (ERPs) la exige |
| M4 | Medium | Refresco automático de SAT 69-B asume fuente estable; falla en silencio |
| M5 | Medium | BC/partes "declarados" = paridad de UI, no de valor antilavado |
| L1 | Low | Diseñar el motor para Capafy (fase 2) desde fase 1 |
| L2 | Low | Volumen de `requiere-verificación` no dimensionado |
| L3 | Low | Caché de prompt manual de Bedrock asumida como gratis |
