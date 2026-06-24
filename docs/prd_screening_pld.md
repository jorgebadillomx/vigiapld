---
title: "PRD — Motor de Screening PLD/FT para Sujetos Obligados (LFPIORPI México)"
project: PreLavadoDinero
status: initial            # versión inicial cerrada bajo BMAD method
version: 1.0
created: 2026-06-16
updated: 2026-06-24
triage: ready-for-agent
glossary: CONTEXT.md       # fuente canónica del vocabulario de dominio
adrs: docs/adr/0001..0010
---

# PRD — Motor de Screening PLD/FT para Sujetos Obligados (LFPIORPI México)

> Alcance: **suite completa** (decisión del propietario, jun-2026). Incluye el motor de screening + dictamen + evidencia, y desde fase 1 también Beneficiario Controlador (BC), autogeneración de avisos XML para SPPLD, matriz EBR de 5 dimensiones, **monitoreo transaccional** (operaciones + acumulación 6 meses + umbrales), **partes relacionadas**, **programa PLD** (manual + capacitación) y **validación de identidad por integración** (decisión jun-2026 tras análisis competitivo de KYC Systems).
> **Decisión de alcance explícita (diverge del brief, deliberadamente).** El brief (§10, §12) recomienda "arranca delgado; BC y avisos XML como fase 2". El propietario **invierte esa recomendación a propósito** para entrar en fase 1 con paridad funcional frente a KYC Systems. Es una apuesta de ventana de mercado, no un descuido. La divergencia se mitiga con la **secuenciación interna de fase 1** (ver §"Secuenciación dentro de fase 1"), que define qué se construye y verifica primero para no entregar 10 medio-módulos. El riesgo de esta apuesta (esfuerzo grande, foso replicable mientras Capafy se difiere) está registrado en `.decision-log.md` y en Further Notes.
> **Canales — fase 1: portal web propio + API/white-label. La skill de Capafy se difiere a fase 2** (decisión del propietario). El motor se diseña desde fase 1 para soportar los tres canales; solo se posterga la construcción del cascarón de Capafy.
> **Glosario de dominio canónico: `CONTEXT.md`** (no el brief). Todo término de dominio —`sujeto evaluado`, `dictamen`, `banda de confianza`, `tipo de lista`, `política de corte`, `Cuenta`, `sujeto obligado`, `expediente`, `reloj de aviso`, `clasificación EBR`— se usa exactamente como lo define `CONTEXT.md` y sus `_Avoid_`. Fuente narrativa: `docs/brief_prelavadodinero.md` (algunas decisiones del brief fueron superadas por ADRs; ver Further Notes).
> Etiqueta de triage: `ready-for-agent`.

---

## Problem Statement

Un **sujeto obligado** mexicano bajo la **LFPIORPI** (joyería, inmobiliaria, notaría, casa de empeño, agente aduanal, donataria, prestador de servicios profesionales, comercializador de vehículos o activos virtuales, etc.) está legalmente obligado a identificar a sus clientes, **cruzarlos contra listas** de sanciones, PEP y listas mexicanas, conservar la documentación **10 años**, operar con **enfoque basado en riesgos**, y —tras la reforma de marzo 2026— identificar al **Beneficiario Controlador**, presentar **avisos al SAT** (incluido el **aviso de 24 horas** por intento de operación) y someterse a una **auditoría/dictamen del programa PLD (Art. 12 Bis)**. Incumplir cuesta de ~$23,462 a ~$7,625,150 MXN (200–65,000 UMAs, 2026).

Hoy el sujeto obligado vive este proceso en Excel y a mano: no sabe distinguir una **coincidencia real** de un **homónimo** (su dolor #1 operativo, los falsos positivos), no tiene forma barata de cruzar contra OFAC/ONU/UE/UK + PEP + SAT 69-B + LPB de la UIF, no rescrea su padrón cuando las listas cambian, no genera el **expediente auditable** que la ley exige guardar 10 años, no produce el XML del aviso, y no tiene una matriz de riesgo (EBR) ni un registro de BC defendibles ante una auditoría. Los despachos y contadores que llevan el PLD de muchos clientes multiplican ese dolor por cada cartera. Las soluciones existentes o apuntan a bancos grandes (caras, lentas de implementar) o son suites genéricas que no resuelven bien la homonimia ni entregan un dictamen redactado y auditable en español encuadrado en la LFPIORPI.

## Solution

Un **único motor de screening PLD/FT (Core API)** que recibe el **padrón** del cliente (clientes, proveedores, donantes) y devuelve, por cada **sujeto evaluado**, un **dictamen auditable**: su **banda de confianza** (`resuelto-real`, `resuelto-homónimo`, `descartado-por-nombre`, `requiere-verificación`), contra qué **tipo de lista** coincidió (sanciones / bloqueados-UIF / PEP / EFOS-EDOS), con qué score, su nivel de riesgo y la **acción AML sugerida**, más el **expediente de evidencia** (PDF/Excel) que aguanta la auditoría del Art. 12 Bis durante 10 años. La decisión legalmente sensible (resolución de identidad) la toman **reglas determinísticas**; el LLM **solo redacta** el dictamen en español, sin decidir ni contradecir el verdicto.

**Honestidad sobre el dato.** La regla determinística resuelve a `resuelto-real` o `resuelto-homónimo` solo cuando el sujeto evaluado trae **RFC + fecha de nacimiento** y la lista permite cruzarlos; ese cruce es lo que elimina el falso positivo. En un padrón de solo nombres —el mínimo que el producto acepta (US-2)— o contra listas que no traen identificadores mexicanos (OFAC/ONU/UE no incluyen RFC), el motor emite con honestidad `requiere-verificación` (score alto, falta el dato para endurecer) o `descartado-por-nombre` (score bajo), nunca un falso "resuelto". El valor en ese caso no es "ya está resuelto" sino "te digo exactamente qué dato pedir para resolverlo y dejo el rastro de que lo revisé". La **tasa esperada de `requiere-verificación`** según la riqueza del padrón es una métrica de producto vigilada (ver Success Metrics) y un insumo para sugerir al sujeto obligado que capture RFC/fecha en el alta.

El mismo motor expone, además del screening: **rescreening recurrente** (que dispara el aviso 24h cuando un cliente ya registrado cae en una lista), **registro de Beneficiario Controlador y partes relacionadas** (declaradas y screeneadas), **monitoreo transaccional** (ingesta de operaciones, acumulación en ventanas de 6 meses y umbrales que disparan el aviso), **autogeneración del XML del aviso para el SPPLD**, una **matriz EBR de 5 dimensiones** documentada, **validación de identidad** (RFC/CURP/69-B y, por integración, INE/biometría/RENAPO) adjunta al expediente, y herramientas de **programa PLD** (plantilla de manual y tracking de capacitación).

**Dos niveles de uso del mismo producto (el monitoreo es opt-in).** La cara simple y la cuña de adopción es **"sube Excel → dictamen + expediente"**: un flujo puntual donde el sujeto obligado sube su padrón y recibe el dictamen para archivar (Nivel 1). Detrás vive un servicio SaaS siempre encendido que, además, permite el **uso continuo** (Nivel 2): el cliente sube su padrón **y sus operaciones** en el tiempo, y el servicio hace rescreening recurrente, acumula operaciones contra umbrales, y emite alertas y avisos. El **monitoreo transaccional es opcional** — un cliente puede vivir solo en el Nivel 1 y nunca subir una operación; el Nivel 2 es para quien tiene volumen, entra por API/ERP, o cuyo giro lo exige por la reforma 2026. El cliente elige qué tan profundo entra; el del Nivel 1 nunca ve la complejidad. (Aun en el Nivel 1, el rescreening mantiene vivo el padrón: lo revisado en enero no queda obsoleto en marzo.)

Se entrega sobre el mismo motor por canales que son cascarones delgados (el motor arma el dictamen, ellos solo lo presentan). **Fase 1:** (1) **portal web propio** (subir Excel → dashboard de hits → revisión/aprobación → export PDF) para el sujeto obligado directo, incluido el despacho/contador multi-cliente; y (2) **API / white-label** (síncrona y batch async, con API keys self-serve, rate limits y webhooks) para ERPs e integradores. **Fase 2:** (3) **skill de Capafy** para despachos/contadores/devs que revenden el screening desde su agente. El motor se diseña desde fase 1 para los tres canales; solo se posterga el cascarón de Capafy.

## User Stories

### Sujeto obligado — onboarding y carga de padrón
1. Como sujeto obligado, quiero subir mi padrón en Excel/CSV (clientes, proveedores, donantes), para cruzarlo contra listas sin capturar nombre por nombre.
2. Como sujeto obligado, quiero que el sistema acepte un mínimo de solo nombres pero aproveche RFC y fecha de nacimiento cuando los tengo, para reducir falsos positivos.
3. Como sujeto obligado, quiero ver un mapeo/validación de columnas al subir el archivo, para corregir un Excel mal formado antes de que se procese.
4. Como sujeto obligado, quiero que el sistema me avise de filas inválidas o duplicadas en mi padrón, para subir datos limpios.
5. Como sujeto obligado, quiero verificar a un solo cliente en tiempo real durante el onboarding (single-name síncrono), para decidir si lo doy de alta antes de cerrar la operación.
6. Como sujeto obligado, quiero re-subir mi padrón actualizado o agregar clientes nuevos, para que el sistema rescree solo lo que cambió.

### Sujeto obligado — dictamen y resolución de homonimia
7. Como sujeto obligado, quiero un dictamen por sujeto evaluado con su **banda de confianza** (`resuelto-real` / `resuelto-homónimo` / `descartado-por-nombre` / `requiere-verificación`), para no tratar a un homónimo como criminal ni dar por limpio lo que falta verificar.
8. Como sujeto obligado, quiero saber contra qué lista y programa coincidió cada nombre (OFAC, ONU, UE, UK, PEP, SAT 69/69-B, LPB), para entender el origen del hit.
9. Como sujeto obligado, quiero ver el score de coincidencia y el nivel de riesgo, para priorizar qué casos revisar primero.
10. Como sujeto obligado, quiero una acción AML sugerida por cada hit (p. ej. rechazar, EDD, reportar, monitorear), para saber qué hacer.
11. Como sujeto obligado, quiero ver el fundamento de la banda de confianza (qué dato coincidió o faltó: RFC, fecha de nacimiento, y qué tipo de lista), para confiar en el dictamen y defenderlo; cuando la banda es `requiere-verificación`, quiero que me diga exactamente qué dato pedir para endurecerla.
12. Como sujeto obligado, quiero poder revisar y aprobar o ajustar manualmente un dictamen, para ejercer mi criterio cuando los datos son ambiguos.
13. Como sujeto obligado, quiero que cada ajuste manual quede registrado con quién, cuándo y por qué, para que el cambio sea auditable.
14. Como sujeto obligado, quiero que el dictamen esté redactado en español claro encuadrado en la LFPIORPI, para entregarlo a mi auditor sin reescribirlo.

### Sujeto obligado — evidencia y conservación 10 años
15. Como sujeto obligado, quiero exportar el expediente de evidencia en PDF y Excel, para conservarlo y entregarlo en una auditoría.
16. Como sujeto obligado, quiero que la evidencia registre qué se revisó, cuándo, contra qué listas y qué versión de cada lista, qué coincidió y qué se decidió, para que sea prueba completa ante el Art. 12 Bis.
17. Como sujeto obligado, quiero descargar el expediente histórico de cualquier corrida pasada, para responder un requerimiento del SAT/UIF años después.
18. Como sujeto obligado, quiero que el sistema conserve el rastro durante 10 años, para cumplir la obligación de conservación sin gestionarlo yo.
19. Como sujeto obligado, quiero que el expediente sea inmutable una vez generado, para que no se pueda alegar que lo alteré después.

### Sujeto obligado — rescreening, alertas y aviso 24h
20. Como sujeto obligado, quiero que mi padrón se rescree automáticamente cuando las listas se actualizan, para detectar a un cliente que cae en una lista después de darlo de alta.
21. Como sujeto obligado, quiero recibir una alerta cuando un cliente existente se vuelve coincidencia real, para actuar a tiempo.
22. Como sujeto obligado, quiero que el sistema marque el reloj del aviso de 24 horas cuando se detecta una coincidencia o un intento de operación reportable, para no perder el plazo legal.
23. Como sujeto obligado, quiero configurar la frecuencia y el alcance del rescreening (todo el padrón, solo activos), para ajustarlo a mi operación.
24. Como sujeto obligado, quiero un historial de rescreenings, para demostrar monitoreo continuo en la auditoría.

### Sujeto obligado — avisos XML SPPLD (suite)
25. Como sujeto obligado, quiero que el sistema autogenere el XML del aviso para el SPPLD a partir del hit y los datos de la operación, para no armarlo a mano.
26. Como sujeto obligado, quiero que el XML se valide contra el esquema oficial del SAT antes de descargarlo, para no rechazarlo en el portal SPPLD.
27. Como sujeto obligado, quiero distinguir el aviso por operación reportable del aviso 24h por intento/coincidencia en listas, para presentar el correcto.
28. Como sujeto obligado, quiero registrar la fecha y el acuse de presentación del aviso, para cerrar el ciclo de cumplimiento con evidencia.
29. Como sujeto obligado, quiero acumular operaciones en periodos de 6 meses según la reforma 2026, para detectar cuándo se cruza un umbral reportable.

### Sujeto obligado — Beneficiario Controlador (suite)
30. Como sujeto obligado, quiero **registrar y conservar** al Beneficiario Controlador **que yo declaro** de mis clientes persona moral (el producto lo screenea y guarda su cadena de control; identificarlo es mi obligación legal, ver ADR-0006), para cumplir la obligación de la reforma 2026.
31. Como sujeto obligado, quiero cruzar al Beneficiario Controlador contra las mismas listas que al cliente, para detectar riesgo escondido tras la estructura societaria.
32. Como sujeto obligado, quiero conservar la evidencia del BC (cadena de control, documentos) junto al expediente, para atender un requerimiento de SAT/UIF.
33. Como sociedad mercantil sin actividad vulnerable, quiero **registrar y conservar** a mi Beneficiario Controlador declarado, para cumplir aun cuando no tenga otra obligación PLD (la identificación de la cadena de control sigue siendo mía, ver ADR-0006).

### Sujeto obligado — EBR / matriz de riesgo (suite)
34. Como sujeto obligado, quiero una matriz EBR (enfoque basado en riesgos) que clasifique a cada cliente por nivel de riesgo, para cumplir el Art. 18 Fr. VII.
35. Como sujeto obligado, quiero que la EBR se calcule sobre las cinco dimensiones oficiales (clientes/usuarios, geografía, productos/servicios, canales de distribución, perfil transaccional), para que la matriz sea defendible ante el auditor.
36. Como sujeto obligado, quiero exportar la EBR documentada, para incluirla en mi programa PLD y mostrarla en la auditoría.

### Despacho / contador (multi-cliente, vía Capafy o portal)
37. Como despacho PLD, quiero administrar el screening de varios clientes sujetos obligados desde una sola cuenta, para llevar su PLD a escala.
38. Como despacho PLD, quiero separar los padrones y expedientes por cliente final, para no mezclar información entre ellos.
39. Como despacho PLD, quiero revender el dictamen a mis clientes con mi propia marca, para ofrecerlo como mi servicio.
40. *(Fase 2 — canal Capafy)* Como contador, quiero correr el screening desde mi agente en Capafy sin salir de mi flujo, para integrarlo a la contabilidad que ya llevo.
41. Como despacho PLD, quiero un resumen consolidado de hits y pendientes de todos mis clientes, para priorizar mi trabajo del día.

### Integrador / ERP (canal API / white-label)
42. Como integrador, quiero llamar a un endpoint síncrono de screening de un solo nombre, para resolver el onboarding en tiempo real desde el ERP.
43. Como integrador, quiero enviar un padrón completo a un endpoint batch asíncrono y recibir un job id, para procesar volúmenes sin bloquear.
44. Como integrador, quiero recibir el resultado del batch por webhook o poll, para enterarme cuando el job termina.
45. Como integrador, quiero recibir un dictamen LFPIORPI terminado (69-B/LPB/RFC + resolución de homonimia) en una sola llamada, para no integrar 5 fuentes ni construir el dictamen yo.
46. Como integrador, quiero generar y administrar mis API keys self-serve, para arrancar sin esperar provisión manual.
47. Como integrador, quiero rate limits claros y errores documentados, para construir mi integración con confianza.
48. Como integrador, quiero un esquema de dictamen versionado (`/v1/`) con changelog y política de deprecación, para no romperme cuando el contrato evolucione.
49. Como integrador, quiero suscribirme a webhooks de rescreening, para enterarme cuando un cliente ya cargado cae en una lista.
50. Como integrador white-label, quiero incrustar la UI del dictamen bajo mi marca, para ofrecerlo dentro de mi producto sin construir pantallas.

### Operador del producto (Jorge / admin)
51. Como operador, quiero que las tres caras (skill, portal, API) llamen a la misma Core API, para no duplicar la lógica del dictamen por canal.
52. Como operador, quiero medir el consumo por Cuenta (metering), para cobrar por uso y recuperar el costo de OpenSanctions.
53. Como operador, quiero hospedar y refrescar automáticamente las listas SAT 69/69-B, para que el cruce siempre use la versión vigente.
54. Como operador, quiero que el cliente suba su propia copia de la LPB de la UIF, para nunca hospedar una lista confidencial.
55. Como operador, quiero una sola integración con OpenSanctions detrás del motor, para tener un solo medidor y un solo punto de control de costos.
56. Como operador, quiero alternar entre OpenSanctions por API (€0.10/consulta) y self-hosted, para optimizar el costo según el volumen.
57. Como operador, quiero inyectar las credenciales de AWS Bedrock y de la Core API desde una bóveda, para no exponerlas en los canales.
58. Como operador, quiero observabilidad de costos por corrida (OpenSanctions vs. LLM en Bedrock), para vigilar el margen.

### Auditor (consumidor de la evidencia)
59. Como auditor del Art. 12 Bis, quiero un expediente que muestre qué listas y versiones se usaron y cuándo, para verificar que el screening fue real y oportuno.
60. Como auditor, quiero ver la cadena de decisiones (automática + ajustes manuales con su justificación), para evaluar el criterio aplicado.
61. Como auditor, quiero confirmar que el rescreening fue continuo y que los avisos 24h se dispararon, para validar el monitoreo.

### Sujeto obligado — monitoreo transaccional (suite)
62. Como sujeto obligado, quiero cargar mis operaciones (Excel/CSV o por API/ERP) ligadas a un cliente, para que el sistema las monitoree.
63. Como sujeto obligado, quiero que el sistema acumule operaciones por cliente en ventanas móviles de 6 meses, para detectar cuándo se cruza un umbral reportable (reforma 2026).
64. Como sujeto obligado, quiero que al cruzar un umbral se dispare automáticamente el reloj de aviso y se arme el aviso XML, para no perder el plazo.
65. Como sujeto obligado, quiero marcar un intento de operación (aunque no se concrete), para que el sistema arranque el aviso 24h del Art. 7 Bis.
66. Como sujeto obligado, quiero que el sistema marque operaciones fuera del perfil esperado por reglas, para revisar inusualidades sin armar yo el análisis.

### Sujeto obligado — partes relacionadas (suite)
67. Como sujeto obligado, quiero declarar las partes relacionadas de un cliente (socios, avales, beneficiarios, propietarios reales), para cumplir la debida diligencia.
68. Como sujeto obligado, quiero que cada parte relacionada se screenee contra todos los tipos de lista igual que el cliente, para detectar riesgo escondido tras la estructura.
69. Como sujeto obligado, quiero que la evidencia de las partes relacionadas quede ligada al expediente del cliente, para atender un requerimiento de SAT/UIF.

### Sujeto obligado — programa PLD (suite)
70. Como sujeto obligado, quiero una plantilla de manual de cumplimiento que el sistema rellene con mi configuración, para tener el documento que la ley exige sin redactarlo desde cero.
71. Como sujeto obligado, quiero registrar y dar seguimiento a la capacitación PLD de mi personal, para demostrar el programa de capacitación en la auditoría.
72. Como sujeto obligado, quiero segregación de funciones por rol (quién screenea, quién aprueba, quién audita), para cumplir el control interno.
73. Como sujeto obligado, quiero un registro de auditoría interna de las acciones del sistema, para evidenciar la supervisión del programa.

### Sujeto obligado — validación de identidad
74. Como sujeto obligado, quiero validar el RFC, el formato de CURP y el estatus 69/69-B de un cliente, para identificarlo correctamente sin salir del flujo.
75. Como sujeto obligado, quiero validar la identidad con INE/biometría/RENAPO cuando lo necesite (vía integración), para reforzar la identificación de clientes de mayor riesgo.
76. Como sujeto obligado, quiero que el resultado de la validación de identidad quede adjunto al expediente, para que sea parte de la evidencia auditable.

## Implementation Decisions

- **Un motor, tres canales (fase 1: dos de ellos).** Se construye una **Core API (ASP.NET Core + EF Core)** como único cerebro. El portal web, la API/white-label y la skill de Capafy son cascarones delgados que invocan la Core API y **solo presentan** el dictamen; ninguno reescribe la lógica de decisión. (Decisión de arquitectura cerrada en el brief, Sección 5.) **En fase 1 se construyen portal web + API/white-label; la skill de Capafy se difiere a fase 2.** El motor y su contrato `/v1/` se diseñan desde fase 1 para los tres canales, de modo que el cascarón de Capafy no exija cambios en el motor cuando llegue.
- **Capa determinística separada de la capa de orquestación LLM.** El núcleo determinístico hace matching, score, reglas RFC/fecha, cruce 69-B/LPB, resolución de homonimia, EBR y armado del expediente. Encima, una capa de orquestación llama a **Claude Sonnet en AWS Bedrock** (ver ADR-0007) que **solo redacta** el dictamen. La decisión legalmente sensible (real vs. homónimo) **nunca** la toma el LLM. El LLM se aísla tras un puerto para poder fakearse y para que su indisponibilidad no tumbe el screening.
- **Regla de homonimia y banda de confianza (encoda la decisión central, ver ADR-0001/0004).** La regla determinística resuelve la **banda de confianza** de cuatro estados (no tres): nombre coincide + (RFC **y** fecha de nacimiento iguales) → `resuelto-real`; nombre coincide pero RFC/fecha **no** → `resuelto-homónimo`; score bajo sin coincidencia relevante → `descartado-por-nombre`; score alto pero **faltan** RFC/fecha para resolver → `requiere-verificación`. El término "indeterminado" queda **proscrito** (CONTEXT.md): no le dice al sujeto obligado qué falta. Esta regla vive en el núcleo determinístico y es la fuente del campo `fundamento` del dictamen. La conversión del score continuo de OpenSanctions a bandas la fija la **política de corte** versionada del operador (ADR-0001), cuya versión se sella en cada expediente.
- **Esquema del dictamen = contrato público versionado (alineado con ADR-0004).** El dictamen es el contrato de la API: se publica bajo `/v1/` con changelog y política de deprecación, y se congela antes de exponer el canal API. El contrato separa explícitamente **resolución de identidad** de **consecuencia AML** (ADR-0004), para que un PEP nunca escale a rechazo automático ni EFOS-EDOS a "no operar". Campos mínimos por sujeto evaluado: `nombre`, `rfc`, `fecha_nacimiento`, `tipo_persona` (física/moral), `coincidencia` (sí/no), `lista_fuente`, `tipo_lista` (`sanciones` / `bloqueados-UIF` / `PEP` / `EFOS-EDOS`), `programa`, `score`, `banda_confianza` (`resuelto-real` / `resuelto-homónimo` / `descartado-por-nombre` / `requiere-verificación`), `fundamento`, `nivel_riesgo`, `accion_sugerida`, `redaccion` (`LLM` / `plantilla`), `fecha_corte`, `version_lista`, `version_politica_corte`. **`accion_sugerida` y `nivel_riesgo` se derivan de la matriz `(banda_confianza × tipo_lista)`**, no del binario coincidió/no ni del campo de identidad. El registro de auditoría agrega `revisado_por`, `revisado_en`, `ajuste_manual`, `justificacion`. *(El campo `decision`/`indeterminado` del esquema anterior queda retirado; `banda_confianza` lo reemplaza.)*
- **Modos de invocación del screening.** Single-name **síncrono** (onboarding en tiempo real) y **batch asíncrono** (padrón → job Hangfire → resultado por webhook/poll). Ambos producen el mismo esquema de dictamen.
- **Fuentes de datos y su hospedaje.** OpenSanctions (global + PEP, vía API `/match` o self-hosted) lo integra y paga el operador; SAT 69/69-B (público) lo hospeda y refresca el operador con Hangfire; la **LPB de la UIF es confidencial → la sube el cliente** y nunca se hospeda centralmente; PEP MX se cubre vía OpenSanctions mientras la UIF no publique medio electrónico.
- **Detección de staleness de listas (no solo refresco optimista).** El SAT publica 69/69-B como archivo en un portal cuya URL/columnas/cadencia cambian sin aviso; un job Hangfire que falle en silencio dejaría al motor cruzando contra una versión vieja mientras el expediente sella "versión vigente". Por eso el refresco de cada fuente registra su `ultima_actualizacion_exitosa` y dispara **alerta de staleness** si una lista no se refresca dentro de su ventana esperada; el dictamen puede marcar advertencia si corre contra una lista marcada stale.
- **Puertos para dependencias externas.** Cada dependencia externa (OpenSanctions, el LLM en AWS Bedrock, fuente de datos SAT) se accede tras una **interfaz/puerto** inyectado en el motor, para aislar I/O, costo y red, y permitir fakes en pruebas.
- **Llamada de matching.** A OpenSanctions `/match` se envía **nombre + RFC + fecha de nacimiento**; los dos últimos son los que matan el falso positivo.
- **LLM de redacción: Claude Sonnet 4.6 en AWS Bedrock** (ver ADR-0007; **supersede** la decisión de DeepSeek del brief §5/§9). Se invoca con el cliente Bedrock de Anthropic (`AnthropicBedrockMantleClient`, paquete `Anthropic.Bedrock`, alineado con el stack ASP.NET Core) contra el endpoint Messages de Bedrock; el id de modelo en Bedrock lleva prefijo `anthropic.` → `anthropic.claude-sonnet-4-6`; la región es requerida (elegir una con el modelo disponible y la residencia de datos deseada). Un solo modelo: como las reglas deciden y el LLM solo redacta (ADR-0004), no hace falta el esquema "orquestador + razonador" de dos modelos del brief. Se usa caché de prompt (manual) de Bedrock sobre las instrucciones/plantilla del dictamen para abaratar el costo por sujeto evaluado. **El cambio a Bedrock (residencia de datos AWS, más defendible en auditoría que un API de terceros genérica) sube el costo de IA frente a DeepSeek; la economía unitaria debe re-derivarse antes de fijar tiers — ver Further Notes.**
- **Fidelidad prosa↔verdicto (el LLM redacta, no contradice).** Como el texto del LLM no es determinista, el ensamblado del dictamen **fuerza** que la prosa cite literalmente la `banda_confianza`, el `tipo_lista` y la `accion_sugerida` del verdicto determinístico; un dictamen cuya prosa contradiga o omita esos campos se rechaza y cae a la **plantilla de respaldo** (`redaccion: plantilla`). Así el expediente "auditable" no puede contener una contradicción interna entre lo que dice el campo y lo que lee el humano. Esto se prueba como seam (ver Testing Decisions).
- **Persistencia y rastro 10 años.** **PostgreSQL** (decisión del propietario, jun-2026) guarda el expediente y el log de eventos. El expediente es **inmutable** una vez generado (los ajustes manuales se registran como eventos nuevos, no como sobrescritura). El rastro guarda versión de cada lista y de la política de corte usadas.
- **Almacenamiento de archivos (expedientes PDF/Excel, documentos de BC/identidad).** Los binarios no viven en la base: se guardan en un **almacén de objetos seguro y de bajo costo**, accedido tras un puerto para no acoplar el motor a un proveedor concreto. Requisitos: cifrado en reposo, control de acceso por sujeto obligado (consistente con ADR-0003), e inmutabilidad/retención de 10 años (write-once / object-lock donde el proveedor lo soporte). La elección concreta del backend (S3, S3-compatible self-hosted tipo MinIO, etc.) la cierra la arquitectura optimizando costo; el motor solo conoce el puerto.
- **Multiplataforma (Windows + Linux; producción Linux).** Toda la arquitectura debe **compilar y correr en Windows y en Linux** (el stack .NET/ASP.NET Core lo soporta de forma nativa) aunque el **despliegue de producción será Linux**. Implicación: nada atado a APIs solo-Windows; rutas, contenedores y dependencias (PostgreSQL, almacén de objetos, Hangfire) elegidos para correr igual en ambos; contenedores Linux como artefacto de despliegue, desarrollo posible en Windows.
- **Protección de datos personales (LFPDPPP) vs. inmutabilidad PLD.** El expediente almacena datos personales —y eventualmente biométricos— de terceros (los sujetos evaluados del cliente) por 10 años. Conviven dos regímenes: la **conservación PLD** (obligación legal de 10 años, base de licitud que prevalece sobre solicitudes de supresión durante el plazo) y la **LFPDPPP / derechos ARCO**. Decisiones: el sujeto obligado es el **responsable** del tratamiento y el producto es **encargado**; los derechos ARCO de rectificación/acceso se atienden **sin romper la inmutabilidad** (la rectificación se registra como evento nuevo, no como sobrescritura); la supresión se difiere hasta vencido el plazo PLD; los datos biométricos solo se obtienen por integración y su tratamiento se acota al expediente. El detalle de aviso de privacidad y contrato de encargo es de operación/legal, no de software (ver Out of Scope), pero el diseño no puede ignorar el régimen.
- **Jobs en background (Hangfire).** (1) Refresco de listas SAT; (2) rescreening recurrente que, al detectar nueva coincidencia real, dispara la alerta y **arranca el reloj del aviso 24h** y emite webhook. El "reloj" depende de una **abstracción de tiempo (clock)** inyectable para ser determinista.
- **Gateway multi-cuenta (ver ADR-0003).** Auth + metering por **Cuenta** (la frontera de facturación), **API keys self-serve**, rate limits y webhooks viven en el gateway. El **aislamiento de datos es por sujeto obligado** —no "por tenant" plano— y se garantiza a nivel de datos: dentro de una Cuenta-despacho, la notaría nunca ve el padrón, el expediente ni la LPB de la joyería. (El término "tenant" queda proscrito, CONTEXT.md/ADR-0003.)
- **Beneficiario Controlador (suite).** Módulo que registra el BC de clientes persona moral, lo cruza contra las mismas listas que al cliente, y conserva su evidencia ligada al expediente del cliente.
- **Avisos XML SPPLD (suite, ver ADR-0010).** Módulo que genera el XML del aviso (operación reportable y aviso 24h) a partir del hit + datos de operación, soporta acumulación de operaciones en periodos de 6 meses, y registra fecha/acuse de presentación. **La validación del XML depende de un esquema oficial del SAT cuya existencia/estabilidad como XSD público está pendiente de confirmar (ADR-0010):** si hay XSD oficial estable, se valida contra él, se versiona y se refresca como las listas SAT; si no, el módulo aplica **validación estructural propia** contra una especificación derivada y la presentación final sigue siendo responsabilidad del sujeto obligado (Out of Scope). El origen, versionado y refresco del esquema se deciden en ADR-0010.
- **EBR / matriz de riesgo (suite).** La clasificación EBR se calcula sobre las **cinco dimensiones oficiales** (clientes/usuarios · geografía · productos/servicios · canales de distribución · perfil transaccional), consumiendo el nivel de riesgo del dictamen (dimensión de listas) y el perfil del módulo de operaciones. Documentada y exportable.
- **Monitoreo transaccional (suite, ver ADR-0008).** Módulo dentro del mismo motor, **sin dependencia externa nueva** (el dato lo da el cliente). Una `Operación` es un evento del log append-only (ADR-0002); una capa determinística aplica umbrales por actividad vulnerable + **acumulación en ventanas móviles de 6 meses**; un job de Hangfire recalcula las ventanas y, al cruzar umbral, dispara el reloj de aviso y el XML reusando los disparadores existentes (`Coincidencia detectada`, `Intento de operación`, `Umbral acumulado`). El LLM no decide aquí (ADR-0004). La **detección de inusualidad "con IA" queda fuera de fase 1**; en fase 1 es por reglas.
- **Partes relacionadas (suite).** Extiende el patrón de BC (ADR-0006): el cliente declara socios, avales, beneficiarios y propietarios reales; se screenean contra todos los tipos de lista y su evidencia se liga al expediente. No hay resolución recursiva de la estructura. **Honestidad del alcance:** la calidad del screening de BC/partes es tan buena como lo que el cliente declare; el producto screenea **al BC/parte declarado**, no descubre al dueño escondido. La evidencia documenta "screeneé lo declarado", no "verifiqué que esto sea el beneficiario real" — la identificación del verdadero controlador es obligación del sujeto obligado. No se debe vender como paridad de *valor antilavado* con quien resuelve cadenas, sino como paridad de *cobertura declarativa*.
- **Validación de identidad por integración (ver ADR-0009).** Tras un puerto, **no construida**: nivel barato (RFC, formato CURP, 69/69-B) en fase 1; INE/biometría/RENAPO por integración con proveedor **a definir**. El resultado se adjunta al expediente. Se fakea en pruebas como cualquier dependencia externa.
- **Programa PLD (suite).** Plantilla de manual de cumplimiento rellenada con la configuración del sujeto obligado, tracking de capacitación, segregación de funciones por rol y registro de auditoría interna. Refuerza el punto de venta ("producimos la evidencia que el auditor del 12 Bis necesita").
- **Stack frontend del portal.** Vite + React + TypeScript + shadcn/ui + Tailwind v4 + TanStack Query + React Router. Flujo: subir Excel → dashboard de hits → revisión/aprobación del dictamen → export PDF → configuración de rescreening + módulos BC/avisos/EBR.
- **Skill de Capafy (fase 2).** Corre cerrada en la nube de Capafy (evitar modo descarga para no exponer prompts/código); llama a la Core API con llaves en la bóveda de Capafy y solo presenta el dictamen. Se construye en fase 2; el motor ya estará listo para soportarla.
- **Posicionamiento legal del producto.** Herramienta de apoyo, no garantía; la responsabilidad del aviso y de la conservación es del sujeto obligado. El producto genera y deja descargable la evidencia, pero no es el responsable legal del expediente. Disclaimers explícitos en los tres canales.

## Testing Decisions

**Qué hace a un buen test aquí:** prueba **comportamiento externo observable**, no detalles de implementación. Para este motor eso significa: dado un padrón/entrada y un estado conocido de las listas (fakes), el sistema produce el **dictamen y el expediente correctos** — la decisión real/homónimo, el fundamento, el score, el nivel de riesgo, la acción y los campos del contrato. Los tests no deben acoplarse a cómo se llama una clase interna, ni hacer red real, ni gastar €/tokens; las dependencias externas se reemplazan por fakes en su puerto. La redacción exacta del LLM **no se asserta** (es no determinista); se assertan los **campos estructurados** del dictamen, que decide el núcleo determinístico.

**Seams confirmados (de mayor a menor altura):**

1. **Contrato HTTP de la Core API** — seam principal. Tests de integración contra `/v1/` (match síncrono, batch async, dictamen, BC, avisos, EBR), con los puertos externos fakeados. Cubre los tres canales de una vez porque todos llaman aquí. Se assertan el esquema del dictamen y los códigos/errores documentados.
2. **Núcleo de decisión determinístico** — tests unitarios directos, exhaustivos, de la lógica legalmente sensible: matching, score, reglas RFC/fecha, cruce 69-B/LPB, **resolución de la banda de confianza** (`resuelto-real` / `resuelto-homónimo` / `descartado-por-nombre` / `requiere-verificación`), derivación de `accion_sugerida`/`nivel_riesgo` desde la matriz `(banda × tipo_lista)`, y cálculo de EBR. Sin I/O. Tabla de casos: RFC+fecha coinciden, no coinciden, faltan (→ `requiere-verificación`), score bajo (→ `descartado-por-nombre`); homónimos; múltiples listas; **PEP no escala a rechazo**; EFOS-EDOS como factor de riesgo, no "no operar".
3. **Puertos de dependencias externas** — OpenSanctions `/match`, el LLM en AWS Bedrock y la fuente SAT 69-B se fakean en su interfaz. Tests verifican que el motor arma el verdicto correcto **independientemente** de la respuesta/redacción del LLM, y que la indisponibilidad del LLM no impide el screening determinístico.
4. **Capa de orquestación LLM** — dado un verdicto determinístico, el dictamen ensamblado contiene todos los campos del contrato; el LLM fakeado. Se prueba el ensamblado/contrato, no el texto. **Test de fidelidad prosa↔verdicto:** dado un fake de LLM que devuelve prosa que contradice u omite la `banda_confianza`/`tipo_lista`/`accion_sugerida` del verdicto, el ensamblado debe rechazarla y caer a la plantilla de respaldo (`redaccion: plantilla`); dado un fake indisponible, el dictamen se emite igual con plantilla y campos determinísticos completos.
5. **Persistencia / rastro 10 años** — con DB real (contenedor/LocalDB) o leyendo de vuelta por la API: una corrida persiste un expediente completo y recuperable, **inmutable**, con versión de listas; un ajuste manual se registra como evento nuevo sin sobrescribir.
6. **Jobs Hangfire** — seam en el método invocable del job + **abstracción de clock** inyectable para que el reloj del **aviso 24h** y el rescreening sean deterministas: al detectar nueva coincidencia real, se dispara alerta, arranca el reloj y se emite webhook.

**Módulos a probar:** núcleo determinístico (incl. homonimia y EBR de 5 dimensiones), capa de orquestación/dictamen, ingesta de padrón (validación de columnas, filas inválidas/duplicadas), persistencia/expediente, jobs de rescreening y refresco SAT, módulo BC y partes relacionadas, **monitoreo transaccional** (reglas de umbral + acumulación en ventanas de 6 meses, deterministas vía clock inyectable; el cruce de umbral dispara el reloj de aviso), módulo de avisos XML (validación contra esquema oficial SAT), **validación de identidad** (puerto fakeado), programa PLD, gateway (auth/metering/rate limits/API keys/webhooks), y export PDF/Excel.

**Prior art:** no hay código aún (proyecto greenfield). Se establece la convención desde cero: tests de integración por contrato HTTP de la Core API como patrón de referencia para features nuevas, y tests unitarios de tabla para el núcleo determinístico. Esta convención queda como prior art para los PRDs siguientes.

## Success Metrics

Métricas que validan la **tesis** del producto (resolvemos la homonimia y producimos evidencia auditable mejor que el screening genérico), no actividad. A calibrar con datos reales tras el primer trimestre de operación.

- **SM-1 · Precisión de resolución.** % de sujetos evaluados que el motor resuelve a `resuelto-real` o `resuelto-homónimo` (vs. quedar en `requiere-verificación`) en padrones con RFC+fecha presentes. Mide que la regla central funciona cuando el dato existe. *Objetivo inicial: a fijar tras baseline.*
- **SM-2 · Reducción de falsos positivos.** % de hits de OpenSanctions que el motor reclasifica como `resuelto-homónimo` (falsos positivos eliminados sin trabajo manual del cliente). Es el dolor #1 que el producto promete resolver.
- **SM-3 · Adopción Nivel 1 → Nivel 2.** % de Cuentas que, habiendo entrado por "sube Excel → dictamen", activan monitoreo continuo (operaciones + rescreening). Mide la cuña de adopción.
- **SM-4 · Margen por corrida.** Ingreso por corrida − (costo OpenSanctions + costo IA Bedrock). Vigila la regla de oro del negocio (precio por volumen, margen delgado).
- **SM-5 · Oportunidad del aviso.** % de relojes de aviso 24h cerrados (aviso generado) dentro del plazo, sobre el total disparado. Mide la utilidad de cumplimiento real.

**Counter-metrics (vigilar el daño colateral):**

- **CM-1 · Falsos negativos.** Coincidencias reales que el motor dejó pasar como `descartado-por-nombre` (detectadas en auditoría o re-corrida). El error más caro legalmente; debe tender a cero aun a costa de más `requiere-verificación`.
- **CM-2 · Carga manual residual.** % del padrón que cae en `requiere-verificación` y exige revisión humana — si crece, la promesa "sube Excel y listo" se erosiona (ligado a la calidad del dato del padrón).
- **CM-3 · Dictámenes en plantilla por caída de LLM.** % de dictámenes emitidos con `redaccion: plantilla` por indisponibilidad de Bedrock — señal de degradación de experiencia (no de corrección: la decisión nunca depende del LLM).

## Non-Functional Requirements

Bounds objetivo para fase 1 (a confirmar contra el pricing/SLA real de Bedrock y OpenSanctions; son contractuales para el canal API/ERP).

- **NFR-1 · Latencia single-name síncrono (US-5/US-42).** p95 < 3 s extremo a extremo (incluye `/match` de OpenSanctions + redacción o plantilla). El onboarding en tiempo real no puede bloquear la operación.
- **NFR-2 · Throughput batch (US-43).** Padrón de referencia de 10,000 sujetos evaluados procesado en < 30 min; job async con `job id` y resultado por webhook/poll. Tamaño máximo de padrón por corrida: a fijar (límite duro documentado para evitar costos descontrolados de OpenSanctions).
- **NFR-3 · Disponibilidad.** Core API 99.5% mensual objetivo; la **indisponibilidad del LLM no degrada la disponibilidad del screening** (cae a plantilla, NFR de diseño ya garantizado por el puerto).
- **NFR-4 · Conservación e inmutabilidad (US-18/19).** Expediente recuperable durante 10 años; inmutable (append-only, ADR-0002); RPO ≤ 24 h, RTO ≤ 8 h para el almacén de expedientes.
- **NFR-5 · Seguridad y datos.** Credenciales (Bedrock, Core API, OpenSanctions) desde bóveda, nunca en los canales; cifrado en reposo y en tránsito; aislamiento por sujeto obligado verificable (ADR-0003). Postura de seguridad alineada a ISO 27001 como meta de canal (ver Further Notes).
- **NFR-6 · Observabilidad de costo.** Costo por corrida desglosado (OpenSanctions vs. IA) por Cuenta, para vigilar SM-4 en tiempo real.

## Acceptance Criteria

Criterios de aceptación a **nivel de capacidad** para las FR de mayor consecuencia. El detalle por historia lo derivará `bmad-create-epics-and-stories`; aquí se fija el "done" de lo legalmente sensible. La regla general de done-ness del dictamen es **"todos los campos del contrato presentes y correctos según el verdicto determinístico"**, no la calidad de la prosa (que no se asserta).

- **AC-Dictamen.** Dado un sujeto evaluado y un estado conocido de listas (fakes), el dictamen contiene todos los campos del contrato `/v1/` con la `banda_confianza` correcta según la tabla de homonimia, `tipo_lista` correcto, y `accion_sugerida`/`nivel_riesgo` derivados de la matriz `(banda × tipo_lista)`. PEP nunca produce `accion_sugerida = rechazar`.
- **AC-Banda de confianza.** RFC+fecha coinciden → `resuelto-real`; nombre coincide y RFC/fecha difieren → `resuelto-homónimo`; faltan RFC/fecha con score alto → `requiere-verificación` (con el campo `fundamento` indicando qué dato pedir); score bajo → `descartado-por-nombre`.
- **AC-Aviso 24h.** Al ocurrir una `Coincidencia detectada`, un `Intento de operación` (entrado por canal) o un `Umbral acumulado`, el `Reloj de aviso` arranca sellando `evento_origen` y `marca_de_tiempo_origen`, cuenta desde el evento (no desde la lectura), y el XML del aviso se arma. Determinista vía clock inyectable.
- **AC-Expediente inmutable.** Una corrida persiste un expediente recuperable con la versión de cada lista y de la política de corte; un ajuste manual se registra como evento nuevo (con `revisado_por`/`revisado_en`/`justificacion`) sin sobrescribir; ninguna operación puede mutar un evento pasado.
- **AC-Avisos XML.** El XML se genera a partir del hit/operación; si hay XSD oficial (ADR-0010) valida contra él, si no aplica validación estructural propia; se registra fecha/acuse de presentación.
- **AC-Aislamiento.** Una Cuenta-despacho con dos sujetos obligados no puede leer, a través de ningún canal, el padrón/expediente/LPB del otro.
- **AC-Ingesta.** Un Excel/CSV con columnas mal mapeadas, filas inválidas (sin nombre) o duplicadas (misma clave RFC, o nombre+fecha cuando no hay RFC) se reporta al usuario antes de procesar; el usuario corrige y re-sube.

## Secuenciación dentro de fase 1

La decisión de alcance es "suite completa en fase 1", pero **no es un entregable atómico**: se construye y verifica por capas, de modo que el motor sea correcto antes de apilarle módulos. Esto mitiga el riesgo de "10 medio-módulos" (el motor más débil define la credibilidad de un producto de cumplimiento). Orden propuesto (refinable por el equipo; no cambia el alcance, solo su orden):

1. **Núcleo verificable (cuña):** ingesta de padrón → núcleo determinístico (banda de confianza + matriz `banda × tipo_lista`) → contrato `/v1/` del dictamen → expediente inmutable (ADR-0002) → export PDF/Excel. Es el flujo "sube Excel → dictamen + expediente" (Nivel 1) y el seam de prueba de referencia.
2. **Vigencia y continuidad:** refresco SAT 69-B + staleness, rescreening recurrente (Hangfire + clock), alertas y reloj de aviso 24h.
3. **Canales:** portal web → API/white-label (keys self-serve, rate limits, webhooks, gateway por Cuenta).
4. **Módulos de suite sobre el núcleo:** BC + partes relacionadas, EBR de 5 dimensiones, monitoreo transaccional (umbrales + ventanas 6 meses), avisos XML (ADR-0010), validación de identidad (nivel barato; integración tras puerto), programa PLD.

Cada capa entra cuando la anterior pasa sus tests de contrato. Las dependencias externas no cerradas (licencia OpenSanctions, XSD del SAT, proveedor de identidad — ver Assumptions) condicionan **sus** módulos, no las capas 1–2.

> **Directiva para `bmad-create-epics-and-stories`:** cortar a detalle (con stories) solo las **capas 1–3** (núcleo verificable, vigencia/continuidad, canales). Las épicas de **monitoreo transaccional** (bloqueada por **AS-4**, umbrales) y **avisos XML** (bloqueada por **AS-2**, esquema del SAT) se crean como **épicas marcadas "bloqueadas", sin stories detalladas**, hasta confirmar el dato con el especialista PLD. No inferir umbrales ni formato de esquema: inventar una regla legal es un riesgo, no un avance.

## Assumptions / Pendientes

Índice consolidado de supuestos y pendientes, con impacto y dónde se resuelven. (Antes dispersos en Further Notes y ADRs.)

| ID | Supuesto / pendiente | Impacto | Bloquea | Se resuelve en |
|----|----------------------|---------|---------|----------------|
| AS-1 | Licencia OpenSanctions: ¿el API hospedado (€0.10/consulta) permite revender dictámenes vía white-label/Capafy, o exige licencia OEM/reseller? | Mayor swing del ROI; define costo variable del canal API/white-label | **Viabilidad del canal white-label** (no el build del motor, que va tras puerto) | Confirmar por escrito con ventas de OpenSanctions **antes de comercializar** el canal white-label |
| AS-2 | Esquema oficial XML del SPPLD: ¿existe un XSD público y estable? | Si no, el XML "validado" se rechaza en el portal y el cliente pierde plazo | Épica de avisos XML (capa 4) | ADR-0010 + confirmación con especialista/portal SAT |
| AS-3 | Proveedor de validación de identidad INE/biometría/RENAPO (US-75/76) | Sin proveedor contratado, la integración de identidad reforzada no se entrega | Solo el nivel reforzado de identidad (el nivel barato RFC/CURP/69-B sí entra) | Selección de proveedor (cobertura, precio, SLA); puerto ya diseñado (ADR-0009) |
| AS-4 | Umbrales reportables del monitoreo por actividad vulnerable (reforma 2026) | El disparo del aviso por `Umbral acumulado` no es construible sin la tabla | Épica de monitoreo transaccional (capa 4) | Tabla de configuración versionada + especialista PLD |
| AS-5 | Valor del piso legal de cadencia de rescreening (ADR-0005) | Piso mal calibrado deja descubierto al cliente de plan básico | Definición de tiers comerciales | Especialista PLD |
| AS-6 | Pricing regional real de Bedrock para Claude Sonnet 4.6 + ahorro real de la caché manual | La economía unitaria (SM-4) y los tiers dependen de esto | Definición de precios (no el build) | Verificar pricing Bedrock + medir caché en pruebas |
| AS-7 | Términos finos de la reforma 2026 (reglas de carácter general aún publicándose): artículos, montos UMA, plazos, alcance BC/PEP | Lógica legalmente sensible codificada sobre reglas no finales | Riesgo de credibilidad si una cita queda desactualizada | Especialista PLD/abogado; tratar reglas no finales como **config versionada**, no como constantes en código |
| AS-8 | ISO/IEC 27001:2022 | Due-diligence de ERPs/integradores suele exigirla (canal de fase 1) | **Comercialización** del canal API/ERP serio (no el build) | Calendarizar certificación contra el lanzamiento del canal; whitepaper + responsabilidad compartida AWS como puente |

> Las afirmaciones legales (AS-7 y las citas de artículos/plazos/montos del Problem Statement) son **internamente consistentes** entre PRD, CONTEXT y ADRs, pero **requieren confirmación de un especialista PLD externo** antes de tratarse como definitivas. Ninguna bloquea el diseño del motor; varias bloquean la *comercialización* o *módulos específicos*, como indica la columna "Bloquea".

## Out of Scope

- **Skill de Capafy (diferida a fase 2).** No se construye en esta fase; el motor sí queda listo para soportarla. La distribución agente-nativa en Capafy sigue siendo el diferenciador de canal, solo que se entrega después del portal web y la API/white-label.
- **PWA instalable con push** para alertas de rescreening (futuro; bajo ROI para B2B documental, sobre el mismo motor más adelante).
- **Sustituir la auditoría del Art. 12 Bis**: el producto genera la evidencia que el auditor necesita, no reemplaza la auditoría ni la certifica autoridad alguna.
- **Presentar avisos directamente en el portal SPPLD del SAT en nombre del cliente**: el producto genera y valida el XML y registra el acuse, pero la presentación la hace el sujeto obligado.
- **Figura de oficial de cumplimiento certificado con examen**: aplica solo al sector financiero, no a las actividades vulnerables objetivo.
- **Hospedar la LPB de la UIF**: confidencial; siempre la sube el cliente.
- **Consultoría/asesoría legal PLD**: el producto es herramienta de apoyo, no garantía ni dictamen legal.
- **Cobranza, facturación y contratos de Capafy/OpenSanctions**: pertenecen al modelo de negocio/operación, no al alcance de software de este PRD.
- **Construir motor propio de identidad/biometría** (OCR, conexiones RENAPO/IMSS/SEP): se **integra**, no se construye (ADR-0009). El nivel barato (RFC/CURP/69-B) sí entra en fase 1.
- **Detección de patrones inusuales "con IA"** en monitoreo transaccional: fase 2; en fase 1 la inusualidad es por reglas (ADR-0008).
- **Sector financiero / CNBV / reportes SITI (TXT)**: no-goal deliberado. El foco es la cola larga de sujetos obligados (actividades vulnerables, SPPLD XML); cubrir ambos regímenes —como KYC Systems/ArmorAML— es competir hacia arriba y diluye la cuña.

## Further Notes

- **Decisión de licencia OpenSanctions [CRÍTICO — ver AS-1].** No bloquea el *build del motor* (va tras un puerto), pero **sí bloquea la comercialización del canal API/white-label**: ese canal es justo el caso reseller que más probablemente cae fuera del API hospedado de €0.10/consulta y exige licencia OEM/reseller (coste desconocido). Distinguir "no bloquea el build" de "no bloquea el negocio": cerrar por escrito con ventas **antes de comercializar** el white-label, no después de construirlo. El motor alterna API (€0.10/consulta, ideal <~30k consultas/mes ≈ 60 clientes) y self-hosted (licencia plana, jugada de escala) tras el puerto.
- **El costo real sigue siendo OpenSanctions, no la IA — pero la economía unitaria debe re-derivarse [ver AS-6].** Una corrida de 500 nombres ≈ $55 (OpenSanctions, ~99% del variable). El brief modeló la IA con DeepSeek (~$0.05/corrida); el cambio a Claude Sonnet 4.6 en Bedrock (~$3.00 input / $15.00 output por 1M tokens, **pricing regional a verificar**) sube el costo de IA ~20–50× frente a DeepSeek. Sigue siendo menor que OpenSanctions porque solo se redacta prosa para los hits (los `descartado-por-nombre` usan plantilla sin LLM) y con caché de prompt manual — **pero "pocos dólares" no es un número**: hay que re-derivar el margen por corrida (SM-4) con el pricing real de Bedrock y la fracción medida de dictámenes que requieren LLM antes de fijar tiers. Implica **precio por volumen / tiers por tamaño de padrón** desde el día 1; una suscripción plana barata va a números rojos.
- **El screening en sí no es el diferenciador.** Al menos 6 jugadores MX (ArmorAML, KYC Systems, Regcheq, Artu, etc.) ya cruzan OFAC/ONU/PEP/69-B. El foso es: distribución agente-nativa en Capafy (nicho abierto, sin skill PLD hoy), cola larga de sujetos obligados chicos, dictamen LLM + resolución de homonimia, API mexicana "lista para LFPIORPI", y precio por volumen. No vender el screening como capacidad nueva.
- **Tensión a vigilar: el foso más limpio (Capafy) se difiere a fase 2.** El único diferenciador que ningún incumbente ocupa hoy es el canal Capafy, y está fuera de fase 1; las demás ventajas (dictamen LLM + homonimia, API LFPIORPI-ready, precio) son **replicables** por cualquiera con una API LLM —el propio brief advierte "un competidor podría envolver cualquiera de estas APIs igual que tú"— y la homonimia depende de la riqueza del dato del padrón (CM-2). Decisión consciente del propietario: construir la paridad de suite en fase 1 y la cabeza diferenciada (Capafy) en fase 2. Re-sopesar si conviene adelantar el cascarón de Capafy una vez que el núcleo (capa 1) esté verificado.
- **Brechas cerradas vs. KYC Systems (análisis jun-2026).** Para competir de frente se incorporaron: monitoreo transaccional + acumulación 6 meses (#1), EBR de 5 dimensiones (#2), partes relacionadas (#3), programa PLD —manual + capacitación— (#4) y validación de identidad por integración (#5). Lo que **no** se persigue: sector financiero/CNBV/SITI (no-goal) y la detección de inusualidad con IA (fase 2). Ventajas que ningún incumbente iguala: Capafy, dictamen LLM + homonimia, self-serve "sube Excel → dictamen", y API/white-label LFPIORPI-ready.
- **ISO/IEC 27001:2022 [ver AS-8].** Certifica el SGSI de la *empresa* (alcance "SaaS PLD"), no el código; **no es requisito legal** para operar, pero es señal de confianza decisiva en el canal API/white-label y ERPs (su due-diligence suele pedirla; KYC Systems la tiene). **Tensión de tiempo:** el canal que la exige (API/ERP) está en fase 1, así que se lanzaría antes de tener el sello. Mitigación: AWS/Bedrock ya están certificados → se heredan controles de infraestructura y se reduce el alcance; arrancar con whitepaper de seguridad + postura de responsabilidad compartida de AWS como **puente, no sustituto** (para un ERP serio no reemplaza el certificado). Calendarizar la certificación contra el lanzamiento comercial del canal ERP. Costo: tiempo y dinero (auditor, ~6–12 meses, vigilancia anual).
- **Canales de alerta WhatsApp/SMS (mejora ligera).** Sumar WhatsApp y SMS a las alertas (hoy webhooks/correo); alto valor cultural en el SMB mexicano. Bajo esfuerzo.
- **Pendientes a confirmar con especialista PLD:** términos finos de la reforma 2026 (varias reglas de carácter general siguen publicándose), pricing regional de AWS Bedrock para Claude Sonnet 4.6, y monto del Platform Sandbox Fee de Capafy (relevante en fase 2).
- **Greenfield:** no existe repo git ni issue tracker configurado; este PRD se publica como archivo local en `docs/` por decisión del propietario. Convertir a issues (con `ready-for-agent`) cuando el repo se inicialice.
