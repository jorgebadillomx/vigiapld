---
stepsCompleted: [1, 2, 3, 4]
status: 'complete'
completedAt: '2026-06-24'
inputDocuments:
  - docs/prd_screening_pld.md
  - _bmad-output/planning-artifacts/architecture.md
  - _bmad-output/planning-artifacts/ux-designs/ux-PreLavadoDinero-2026-06-24/DESIGN.md
  - _bmad-output/planning-artifacts/ux-designs/ux-PreLavadoDinero-2026-06-24/EXPERIENCE.md
  - CONTEXT.md
  - docs/adr/0001-umbral-matching-fijo-versionado.md
  - docs/adr/0002-rastro-event-sourced.md
  - docs/adr/0003-tenancy-cuenta-sujeto-obligado.md
  - docs/adr/0004-resolucion-identidad-vs-consecuencia-aml.md
  - docs/adr/0005-cadencia-rescreening-con-piso-legal.md
  - docs/adr/0006-beneficiario-controlador-declarado-sin-resolucion-recursiva.md
  - docs/adr/0007-llm-claude-sonnet-en-aws-bedrock.md
  - docs/adr/0008-monitoreo-transaccional-deterministico-event-sourced.md
  - docs/adr/0009-validacion-identidad-por-integracion.md
  - docs/adr/0010-avisos-xml-sppld-esquema-y-validacion.md
project_name: 'PreLavadoDinero'
user_name: 'Jorge'
date: '2026-06-24'
---

# PreLavadoDinero - Epic Breakdown

## Overview

Este documento descompone los requisitos del PRD (Motor de Screening PLD/FT para Sujetos Obligados, LFPIORPI México), el UX Design (portal **Vigía**) y la Arquitectura (.NET 10 / Clean Architecture / Marten / PostgreSQL / AWS) en épicas e historias implementables.

**Vocabulario de dominio:** la autoridad canónica es `CONTEXT.md`. Términos proscritos (`indeterminado`, `tenant`) no aparecen en historias ni código.

**Directiva de corte (PRD §Secuenciación dentro de fase 1):** se cortan a detalle (con stories) solo las **capas 1–3** (núcleo verificable, vigencia/continuidad, canales). Las épicas de **monitoreo transaccional** (bloqueada por **AS-4**, umbrales) y **avisos XML SPPLD** (bloqueada por **AS-2**, esquema del SAT) se crean como **épicas marcadas "bloqueadas", sin stories detalladas**, hasta confirmar el dato con el especialista PLD.

## Requirements Inventory

### Functional Requirements

> Los requisitos funcionales son las **76 user stories** del PRD, identificadas con su ID canónico **US-N** (usado en PRD y Arquitectura). Se conservan agrupadas por actor/módulo.

**Sujeto obligado — onboarding y carga de padrón**
- **FR-US-1:** Subir el padrón en Excel/CSV (clientes, proveedores, donantes) para cruzarlo contra listas sin capturar nombre por nombre.
- **FR-US-2:** Aceptar un mínimo de solo nombres pero aprovechar RFC y fecha de nacimiento cuando existan, para reducir falsos positivos.
- **FR-US-3:** Mostrar un mapeo/validación de columnas al subir el archivo, para corregir un Excel mal formado antes de procesarlo.
- **FR-US-4:** Avisar de filas inválidas o duplicadas en el padrón, para subir datos limpios.
- **FR-US-5:** Verificar a un solo cliente en tiempo real durante el onboarding (single-name síncrono).
- **FR-US-6:** Re-subir el padrón actualizado o agregar clientes nuevos, rescreando solo lo que cambió.

**Sujeto obligado — dictamen y resolución de homonimia**
- **FR-US-7:** Dictamen por sujeto evaluado con su **banda de confianza** (`resuelto-real` / `resuelto-homónimo` / `descartado-por-nombre` / `requiere-verificación`).
- **FR-US-8:** Indicar contra qué lista y programa coincidió cada nombre (OFAC, ONU, UE, UK, PEP, SAT 69/69-B, LPB).
- **FR-US-9:** Mostrar el score de coincidencia y el nivel de riesgo, para priorizar la revisión.
- **FR-US-10:** Acción AML sugerida por cada hit (rechazar, EDD, reportar, monitorear).
- **FR-US-11:** Mostrar el fundamento de la banda (qué dato coincidió o faltó); en `requiere-verificación`, decir exactamente qué dato pedir para endurecerla.
- **FR-US-12:** Revisar y aprobar o ajustar manualmente un dictamen.
- **FR-US-13:** Registrar cada ajuste manual con quién, cuándo y por qué (auditable).
- **FR-US-14:** Dictamen redactado en español claro encuadrado en la LFPIORPI, entregable al auditor sin reescritura.

**Sujeto obligado — evidencia y conservación 10 años**
- **FR-US-15:** Exportar el expediente de evidencia en PDF y Excel.
- **FR-US-16:** Registrar en la evidencia qué se revisó, cuándo, contra qué listas y versión de cada lista, qué coincidió y qué se decidió (prueba Art. 12 Bis).
- **FR-US-17:** Descargar el expediente histórico de cualquier corrida pasada.
- **FR-US-18:** Conservar el rastro durante 10 años.
- **FR-US-19:** Expediente inmutable una vez generado.

**Sujeto obligado — rescreening, alertas y aviso 24h**
- **FR-US-20:** Rescreening automático del padrón cuando las listas se actualizan.
- **FR-US-21:** Alerta cuando un cliente existente se vuelve coincidencia real.
- **FR-US-22:** Marcar el reloj del aviso de 24 horas al detectar coincidencia o intento de operación reportable.
- **FR-US-23:** Configurar frecuencia y alcance del rescreening (todo el padrón, solo activos) — con piso legal (ADR-0005).
- **FR-US-24:** Historial de rescreenings, para demostrar monitoreo continuo.

**Sujeto obligado — avisos XML SPPLD (suite) [BLOQUEADO AS-2]**
- **FR-US-25:** Autogenerar el XML del aviso para el SPPLD a partir del hit y datos de la operación.
- **FR-US-26:** Validar el XML contra el esquema oficial del SAT antes de descargarlo.
- **FR-US-27:** Distinguir el aviso por operación reportable del aviso 24h por intento/coincidencia.
- **FR-US-28:** Registrar fecha y acuse de presentación del aviso.
- **FR-US-29:** Acumular operaciones en periodos de 6 meses (reforma 2026) para detectar umbral reportable.

**Sujeto obligado — Beneficiario Controlador (suite)**
- **FR-US-30:** Registrar y conservar al BC declarado de clientes persona moral; el producto lo screenea y guarda su cadena de control (ADR-0006).
- **FR-US-31:** Cruzar al BC contra las mismas listas que al cliente.
- **FR-US-32:** Conservar la evidencia del BC (cadena de control, documentos) junto al expediente.
- **FR-US-33:** Sociedad mercantil sin actividad vulnerable puede registrar y conservar su BC declarado (ADR-0006).

**Sujeto obligado — EBR / matriz de riesgo (suite)**
- **FR-US-34:** Matriz EBR que clasifique a cada cliente por nivel de riesgo (Art. 18 Fr. VII).
- **FR-US-35:** EBR calculada sobre las cinco dimensiones oficiales (clientes/usuarios, geografía, productos/servicios, canales de distribución, perfil transaccional).
- **FR-US-36:** Exportar la EBR documentada.

**Despacho / contador (multi-cliente)**
- **FR-US-37:** Administrar el screening de varios sujetos obligados desde una sola cuenta.
- **FR-US-38:** Separar padrones y expedientes por cliente final (aislamiento por sujeto obligado).
- **FR-US-39:** Revender el dictamen a clientes con marca propia (white-label).
- **FR-US-40:** *(Fase 2 — canal Capafy)* Correr el screening desde el agente en Capafy.
- **FR-US-41:** Resumen consolidado de hits y pendientes de todos los clientes.

**Integrador / ERP (canal API / white-label)**
- **FR-US-42:** Endpoint síncrono de screening de un solo nombre (onboarding en tiempo real).
- **FR-US-43:** Endpoint batch asíncrono que recibe un padrón completo y devuelve un job id.
- **FR-US-44:** Recibir el resultado del batch por webhook o poll.
- **FR-US-45:** Recibir un dictamen LFPIORPI terminado en una sola llamada.
- **FR-US-46:** Generar y administrar API keys self-serve.
- **FR-US-47:** Rate limits claros y errores documentados.
- **FR-US-48:** Esquema de dictamen versionado (`/v1/`) con changelog y política de deprecación.
- **FR-US-49:** Suscribirse a webhooks de rescreening.
- **FR-US-50:** Incrustar la UI del dictamen bajo marca propia (white-label).

**Operador del producto (Jorge / admin)**
- **FR-US-51:** Que las tres caras (skill, portal, API) llamen a la misma Core API.
- **FR-US-52:** Medir el consumo por Cuenta (metering) para cobrar por uso.
- **FR-US-53:** Hospedar y refrescar automáticamente las listas SAT 69/69-B.
- **FR-US-54:** Que el cliente suba su propia copia de la LPB de la UIF (nunca hospedada centralmente).
- **FR-US-55:** Una sola integración con OpenSanctions detrás del motor.
- **FR-US-56:** Alternar entre OpenSanctions por API y self-hosted (tras puerto).
- **FR-US-57:** Inyectar credenciales (Bedrock, Core API) desde una bóveda.
- **FR-US-58:** Observabilidad de costos por corrida (OpenSanctions vs. LLM en Bedrock).

**Auditor (consumidor de la evidencia)**
- **FR-US-59:** Expediente que muestre qué listas y versiones se usaron y cuándo.
- **FR-US-60:** Cadena de decisiones (automática + ajustes manuales con justificación).
- **FR-US-61:** Confirmar que el rescreening fue continuo y que los avisos 24h se dispararon.

**Sujeto obligado — monitoreo transaccional (suite) [BLOQUEADO AS-4]**
- **FR-US-62:** Cargar operaciones (Excel/CSV o por API/ERP) ligadas a un cliente.
- **FR-US-63:** Acumular operaciones por cliente en ventanas móviles de 6 meses.
- **FR-US-64:** Al cruzar un umbral, disparar el reloj de aviso y armar el aviso XML.
- **FR-US-65:** Marcar un intento de operación (aunque no se concrete) para arrancar el aviso 24h (Art. 7 Bis).
- **FR-US-66:** Marcar operaciones fuera del perfil esperado por reglas.

**Sujeto obligado — partes relacionadas (suite)**
- **FR-US-67:** Declarar las partes relacionadas de un cliente (socios, avales, beneficiarios, propietarios reales).
- **FR-US-68:** Screenear cada parte relacionada contra todos los tipos de lista igual que el cliente.
- **FR-US-69:** Ligar la evidencia de las partes relacionadas al expediente del cliente.

**Sujeto obligado — programa PLD (suite)**
- **FR-US-70:** Plantilla de manual de cumplimiento rellenada con la configuración del sujeto obligado.
- **FR-US-71:** Registrar y dar seguimiento a la capacitación PLD del personal.
- **FR-US-72:** Segregación de funciones por rol (quién screenea, quién aprueba, quién audita).
- **FR-US-73:** Registro de auditoría interna de las acciones del sistema.

**Sujeto obligado — validación de identidad**
- **FR-US-74:** Validar RFC, formato CURP y estatus 69/69-B (nivel barato, fase 1).
- **FR-US-75:** Validar identidad con INE/biometría/RENAPO por integración (nivel reforzado) [proveedor a definir, AS-3].
- **FR-US-76:** Adjuntar el resultado de la validación de identidad al expediente.

### NonFunctional Requirements

- **NFR-1 · Latencia single-name síncrono (US-5/US-42):** p95 < 3 s extremo a extremo (incluye `/match` de OpenSanctions + redacción o plantilla).
- **NFR-2 · Throughput batch (US-43):** padrón de referencia de 10,000 sujetos procesado en < 30 min; job async con `job id` y resultado por webhook/poll; tamaño máx. de padrón acotado (límite duro documentado).
- **NFR-3 · Disponibilidad:** Core API 99.5% mensual objetivo; la indisponibilidad del LLM **no** degrada el screening (cae a plantilla).
- **NFR-4 · Conservación e inmutabilidad (US-18/19):** expediente recuperable 10 años; inmutable (append-only, ADR-0002); RPO ≤ 24 h, RTO ≤ 8 h.
- **NFR-5 · Seguridad y datos:** credenciales desde bóveda, nunca en los canales; cifrado en reposo y en tránsito; aislamiento por sujeto obligado verificable (ADR-0003); postura ISO 27001 como meta de canal.
- **NFR-6 · Observabilidad de costo:** costo por corrida desglosado (OpenSanctions vs. IA) por Cuenta, para vigilar SM-4.

### Additional Requirements

> Requisitos técnicos derivados de la Arquitectura y los ADRs que condicionan la implementación. Marcados como **AR-N**.

**Inicialización (impacta Épica 1, Historia 1 — Starter Template)**
- **AR-1 · Starter backend:** Jason Taylor Clean Architecture (`dotnet new ca-sln --client-framework None --database PostgreSQL`) con **dos ajustes desde el día 1**: (a) sustituir **MediatR** por el source-generator `Mediator` (Martin Othamar, MIT) o handlers propios; (b) montar **Marten** (event store OSS sobre Postgres) en Infrastructure.
- **AR-2 · Starter frontend:** Vite + React-TS + Tailwind v4 + shadcn/ui + TanStack Query + React Router (desacoplado, cascarón sobre `/v1/`).
- **AR-3 · .NET 10 LTS + contenedores Linux:** compila y corre en Windows y Linux; producción Linux; sin APIs solo-Windows.

**Datos y persistencia**
- **AR-4 · Event store Marten 8.x:** rastro append-only (ADR-0002); proyecciones **inline** al inicio; async daemon + snapshots cuando el volumen lo pida (mitiga R1). EF Core para CRUD/catálogos/metering.
- **AR-5 · Sello de reproducibilidad:** cada expediente sella versión de lista, de política de corte y **del matcher de OpenSanctions** (mitiga R2; en modo API se sella la versión de dataset que devuelve la respuesta).
- **AR-6 · Convenciones de datos:** `snake_case` en PostgreSQL; Marten gestiona sus propias tablas de eventos (no tocar); migraciones EF Core + esquema Marten.

**Aislamiento multi-tenant y seguridad**
- **AR-7 · Aislamiento por sujeto obligado:** Row-Level Security de Postgres **+** filtro obligatorio en repositorio por `sujeto_obligado_id` tomado del **principal autenticado, nunca del input**; test adversarial de fuga cruzada (IDOR sweep).
- **AR-8 · WORM/inmutabilidad:** **S3 Object Lock (modo compliance, retención 10 años)** para expedientes; **verificado por test set-at-creation** antes de persistir cualquier expediente real (R3). El evento de dictamen persiste en DB aunque el binario del expediente falle (desacoplar).
- **AR-9 · Secretos:** AWS Secrets Manager + IAM role; cero credenciales en contenedor/logs. LPB confidencial cifrada por sujeto obligado, no persistida más de lo necesario.
- **AR-10 · Protección de datos (LFPDPPP/ARCO):** base de licitud PLD documentada; rectificación como evento nuevo; supresión diferida al vencer el plazo de 10 años.
- **AR-11 · RBAC:** segregación de funciones screener / aprobador / auditor (US-72).

**Puertos a dependencias externas**
- **AR-12 · Puertos:** toda I/O externa tras una interfaz `I<Dependencia>Port` en Application, implementada en Infrastructure, fakeable en test: `IOpenSanctionsPort`, `ILlmRedactorPort`, `ISatListaPort`, `IIdentidadPort`, `IAlmacenObjetosPort`, `IClock`.
- **AR-13 · LLM Claude Sonnet 4.6 en AWS Bedrock (ADR-0007):** cliente `AnthropicBedrockMantleClient`; id de modelo `anthropic.claude-sonnet-4-6`; región requerida; caché de prompt manual. El LLM **solo redacta**, nunca decide.
- **AR-14 · Fidelidad prosa↔verdicto:** el ensamblado fuerza que la prosa cite literalmente `banda_confianza`, `tipo_lista` y `accion_sugerida`; si contradice u omite, se rechaza y cae a plantilla (`redaccion: plantilla`). Probado como seam.
- **AR-15 · Degradación:** caída de Bedrock → plantilla; caída de OpenSanctions (SPOF) → dictamen con tipos de lista locales + marca "sanciones pendientes"; backoff exponencial + circuit breaker.
- **AR-16 · Detección de staleness de listas:** cada refresco registra `ultima_actualizacion_exitosa` y dispara alerta si una lista no se refresca dentro de su ventana; el dictamen marca advertencia si corre contra lista stale.

**API, jobs y comunicación**
- **AR-17 · Contrato REST `/v1/` versionado:** changelog + deprecación; sync (single-name) + batch async; el esquema del dictamen se congela antes de exponer el canal API.
- **AR-18 · Esquema de dictamen (ADR-0004):** campos mínimos `nombre, rfc, fecha_nacimiento, tipo_persona, coincidencia, lista_fuente, tipo_lista, programa, score, banda_confianza, fundamento, nivel_riesgo, accion_sugerida, redaccion, fecha_corte, version_lista, version_politica_corte`; auditoría agrega `revisado_por, revisado_en, ajuste_manual, justificacion`. `accion_sugerida` y `nivel_riesgo` se derivan de la matriz `(banda_confianza × tipo_lista)`. Separa resolución de identidad de consecuencia AML (PEP nunca escala a rechazo automático).
- **AR-19 · Errores:** ProblemDetails (RFC 7807) con catálogo de `type` versionado en `/v1/`.
- **AR-20 · Webhooks:** patrón **Outbox** sobre el event store; payload firmado HMAC; entrega at-least-once + idempotencia por id de evento; alerta de lag del Outbox.
- **AR-21 · Jobs Hangfire:** (1) refresco SAT + staleness; (2) rescreening recurrente que dispara alerta, arranca el reloj 24h y emite webhook; (3) outbox relay. Reloj de aviso vía **`IClock` inyectable** (UTC + zona MX explícita; tests de DST; monitoreo dedicado de la cadena, R4).
- **AR-22 · API keys:** hasheadas en reposo, mostradas una vez, nunca logueadas, con scope/rotación/metering por key; rate limiter nativo de ASP.NET Core.

**Infraestructura y despliegue**
- **AR-23 · Hosting right-sized:** arranque "barato dentro de AWS" (EC2/Lightsail + contenedores, RDS mínimo o Postgres on-box) **o** variante lean VPS compartido (Coolify/Dokploy) — elegida por el propietario. Frontera no-negociable en ambos: **expediente WORM y backups en almacenamiento gestionado con Object Lock (idealmente `mx-central-1`), nunca en disco de VPS**; backups DB off-box (RTO≤8h/RPO≤24h).
- **AR-24 · OpenSanctions API-first:** arrancar por API `/match` (€0.10/llamada); self-host diferido (US-56) tras el puerto. **AS-1** (licencia Reseller/OEM) bloquea la *comercialización* del white-label, no el build.
- **AR-25 · CI/CD:** GitHub Actions (build + test + imagen Linux) al inicializar el repo.
- **AR-26 · Observabilidad:** logs estructurados con `traceId` + `sujeto_obligado_id` (nunca PII/LPB/secretos) + costo por corrida primero; OpenTelemetry diferido.

**Testing (convención del PRD, prior art greenfield)**
- **AR-27 · Seams de prueba:** (1) contrato HTTP `/v1/` (seam principal, integración con puertos fakeados, Postgres en contenedor); (2) núcleo determinístico (unitarios de tabla: bandas, matriz, homonimia, EBR); (3) puertos fakeados; (4) orquestación LLM + fidelidad; (5) persistencia/rastro 10 años inmutable; (6) jobs Hangfire + clock inyectable.

### UX Design Requirements

> Requisitos accionables extraídos de DESIGN.md (identidad visual Vigía) y EXPERIENCE.md (comportamiento). Marcados como **UX-DR-N**. Cada uno es lo bastante específico para generar una historia con criterios de aceptación verificables.

**Sistema de diseño y marca (DESIGN.md)**
- **UX-DR-1 · Sistema de tokens de 3 temas:** implementar tokens de color/tipografía/espaciado/radios/elevación con **Toga** como tema canónico y **Notario** / **Salvaguarda** como variantes seleccionables (mismos nombres de token). Nunca mezclar tokens de dos temas en una vista.
- **UX-DR-2 · Semántica de banda de confianza (token central, no decorativo):** color-codificar `banda-real` (rojo) / `banda-homónimo` (azul) / `banda-verificar` (ámbar) / `banda-descartado` (verde) / `pep` (morado); cada token con tres roles separados (hue vivo, fondo `-bg`, texto `-text`); **Bloqueo-UIF** es el único badge invertido (fondo `banda-real` sólido + texto blanco).
- **UX-DR-3 · Tipografía editorial-legal:** Spectral (serif, display/titulares/prosa de dictamen), Hanken Grotesk (sans, UI/cuerpo), IBM Plex Mono (todo dato: scores, RFC, folios, KPIs, precios, reloj 24h, versiones de lista).
- **UX-DR-4 · Cifras tabulares (regla dura, CLS):** `font-variant-numeric: tabular-nums` obligatorio en todo lo `mono`; el reloj que tickea cada segundo no puede saltar de ancho ni provocar micro-CLS.
- **UX-DR-5 · Componentes Vigía sobre shadcn:** Button (primario/secundario/ghost-hero/**aviso** full-width "Generar aviso XML (SPPLD)"), Badge de banda, Tarjeta de dictamen, KPI card, Tabla de dictámenes (grid + score con color por umbral), Reloj de aviso 24h (dot pulsante + contador con color de urgencia <6h/<12h), Sidebar (`hero` oscuro), Dropzone + stepper de 3 pasos, Inputs.

**Comportamiento y arquitectura de información (EXPERIENCE.md)**
- **UX-DR-6 · Dos caras conectadas:** landing pública (venta, scroll de una página: hero/lead magnets/homónimo/cómo funciona/cobertura/precios/trust+disclaimers/footer) y panel privado (sidebar fijo + main con scroll).
- **UX-DR-7 · Multi-cuenta y selector de sujeto obligado:** el panel opera siempre en el contexto de un sujeto obligado activo; el despacho cambia de contexto con un selector + vista "Todos" (resumen consolidado). Cambiar contexto recarga las vistas. Aislamiento por sujeto obligado respetado en cada vista.
- **UX-DR-8 · Surfaces del panel capa 1:** Resumen (dashboard con KPIs, reloj 24h en vivo, hits por banda, corridas recientes, cobertura), Subir padrón (dropzone → mapeo de columnas + validación → procesamiento → resumen), Dictámenes (tabla filtrable por banda), Detalle de dictamen (veredicto, fundamento+cotejos, prosa LLM/plantilla, expediente timeline, reloj+aviso si aplica).
- **UX-DR-9 · Surface capa 2:** Configuración de rescreening (cadencia y alcance; piso legal mínimo visible, ADR-0005).
- **UX-DR-10 · Surfaces de suite:** BC/partes, EBR, Programa PLD, Validación de identidad; **módulos bloqueados (Monitoreo AS-4 / Avisos AS-2)** aparecen en la nav con estado visible "Próximamente — pendiente de configuración legal", no ocultos.
- **UX-DR-11 · Verificador en vivo (lead magnet landing):** input + chips de ejemplo → animación de cotejo (~1.25 s) → tarjeta de resultado con banda, sin registro.
- **UX-DR-12 · Dropzone + stepper de carga:** 3 pasos (Cargar/Mapear/Resultado); acepta .xlsx/.xls/.csv, máx 10,000 sujetos/corrida (NFR-2); mapeo de columnas editable; reporta filas inválidas/duplicadas antes de procesar; botón "Usar archivo de ejemplo".
- **UX-DR-13 · Estados (State Patterns):** Skeleton de carga, padrón vacío, corrida procesando (spinner + log mono), sin hits, `requiere-verificación` (ámbar + qué dato pedir + capturar→re-resolver), LLM caído (plantilla + "Re-redactar"), lista stale, módulo bloqueado, reloj <6h, sin permiso, offline (Toast). Nunca pantalla en blanco.
- **UX-DR-14 · Primitivas de interacción:** clic-para-actuar en filas; el reloj cuenta solo (lo dispara el motor, no el usuario); confirmación solo en lo irreversible (aprobar dictamen, generar aviso); prohibido scroll infinito (paginación) y color como único portador de significado.
- **UX-DR-15 · Voz y tono (microcopy):** español de México, registro técnico-legal; "Requiere verificación — falta RFC o fecha" (nunca "indeterminado"); "PEP — dispara EDD, no rechazo"; disclaimer "Herramienta de apoyo, no garantía. El aviso lo presenta el sujeto obligado" siempre presente; cifras de multa siempre con fuente (UMA 2026 · DOF).

**Accesibilidad (regla dura)**
- **UX-DR-16 · WCAG 2.2 AA** en toda la web responsive.
- **UX-DR-17 · Gate de contraste:** las 15 combinaciones banda×tema (`banda-*-text` sobre `banda-*-bg`) verificadas ≥4.5:1 antes de release; igual `on-hero` sobre `hero` e `ink`/`ink-soft` sobre superficies. `muted` NO es color de texto de cuerpo/dato.
- **UX-DR-18 · Foco visible (delta obligatorio):** anillo de foco visible en TODO interactivo, adaptativo a la superficie (`accent` en claro, claro `on-hero`/#fff sobre sidebar oscuro), ≥3:1 (WCAG 2.4.11). Corrige `outline:none` del prototipo.
- **UX-DR-19 · Banda y score nunca solo por color:** badge con etiqueta textual + punto (daltonismo-seguro); score con valor numérico + etiqueta de riesgo (Alto/Medio/Bajo).
- **UX-DR-20 · Reloj 24h y `aria-live`:** contador que cambia cada segundo es `aria-hidden`; región `aria-live="polite"` separada anuncia solo cruces de umbral (<12h, <6h).
- **UX-DR-21 · Movimiento (WCAG 2.3.3):** dot pulsante, scanline y spinner respetan `prefers-reduced-motion: reduce`; ninguna información depende del movimiento.
- **UX-DR-22 · Teclado y objetivos táctiles:** orden de Tab = orden de lectura; `Esc` cierra modal/popover; filas/chips operables por teclado; objetivos táctiles ≥24px (idealmente 44px) en móvil; `lang="es"`.

**Rendimiento (piso obligatorio)**
- **UX-DR-23 · Virtualizar tablas/listas >50 filas:** la tabla de dictámenes (cientos de hits de una corrida de 10,000) se virtualiza + paginación; aplica también a tarjetas en móvil.
- **UX-DR-24 · Reservar espacio para contenido asíncrono (CLS < 0.1):** skeletons con forma/dimensiones reales; tarjeta de dictamen del hero, KPIs y verificador reservan su caja; el dashboard que se repuebla solo no debe saltar.
- **UX-DR-25 · Estrategia de fuentes y code-splitting:** `font-display: swap` en las 3 familias; `preload` solo de críticas above-the-fold; subset `latin`; latencia de interacción <100ms; code-splitting por ruta (landing vs panel).

**Responsive (piso obligatorio)**
- **UX-DR-26 · Web responsive:** breakpoints `≥lg` (sidebar fijo 248px + tablas completas), `md` (sidebar → iconos, dashboard apila), `<md`/`sm` (sidebar → Sheet/drawer, **tabla de dictámenes → tarjetas** con etiqueta visible por campo, detalle a pantalla completa, reloj 24h como banner superior). Trabajo central en móvil: revisar dictámenes y atender el reloj de aviso.

### FR Coverage Map

> Mapeo de cada user story (US-N) a su épica. Garantiza que ningún FR se pierda. (NFR/AR/UX-DR se atienden transversalmente dentro de las épicas indicadas en cada `**FRs covered**`.)

| US | Épica | Descripción breve |
|---|---|---|
| US-1 | Épica 1 | Subir padrón Excel/CSV |
| US-2 | Épica 1 | Mínimo de nombres, aprovechar RFC/fecha |
| US-3 | Épica 1 + 2 | Mapeo/validación de columnas (motor + UI dropzone) |
| US-4 | Épica 1 + 2 | Reporte de filas inválidas/duplicadas |
| US-5 | Épica 1 | Single-name síncrono (onboarding tiempo real) |
| US-6 | Épica 1 | Re-subir padrón / agregar clientes |
| US-7 | Épica 1 | Dictamen con banda de confianza |
| US-8 | Épica 1 | Lista y programa de coincidencia |
| US-9 | Épica 1 | Score y nivel de riesgo |
| US-10 | Épica 1 | Acción AML sugerida |
| US-11 | Épica 1 | Fundamento de la banda (qué dato pedir) |
| US-12 | Épica 1 + 2 | Revisar/aprobar/ajustar dictamen |
| US-13 | Épica 1 | Ajuste manual auditable (evento nuevo) |
| US-14 | Épica 1 | Prosa LLM en español LFPIORPI + fidelidad |
| US-15 | Épica 1 + 2 | Export expediente PDF/Excel |
| US-16 | Épica 1 | Evidencia completa (qué/cuándo/versión) |
| US-17 | Épica 1 + 2 | Descargar expediente histórico |
| US-18 | Épica 1 | Conservación 10 años |
| US-19 | Épica 1 | Expediente inmutable |
| US-20 | Épica 3 | Rescreening automático al actualizar listas |
| US-21 | Épica 3 | Alerta de coincidencia real sobreviniente |
| US-22 | Épica 3 | Marcar reloj de aviso 24h |
| US-23 | Épica 3 | Configurar frecuencia/alcance (piso legal) |
| US-24 | Épica 3 | Historial de rescreenings |
| US-25 | Épica 9 **[BLOQUEADO AS-2]** | Autogenerar XML del aviso SPPLD |
| US-26 | Épica 9 **[BLOQUEADO AS-2]** | Validar XML contra esquema SAT |
| US-27 | Épica 9 **[BLOQUEADO AS-2]** | Distinguir aviso reportable vs 24h |
| US-28 | Épica 9 **[BLOQUEADO AS-2]** | Registrar fecha/acuse de presentación |
| US-29 | Épica 9 **[BLOQUEADO AS-2]** | Acumular operaciones 6 meses |
| US-30 | Épica 5 | Registrar/conservar BC declarado |
| US-31 | Épica 5 | Cruzar BC contra mismas listas |
| US-32 | Épica 5 | Conservar evidencia del BC |
| US-33 | Épica 5 | Sociedad sin actividad vulnerable registra BC |
| US-34 | Épica 6 | Matriz EBR por nivel de riesgo |
| US-35 | Épica 6 | EBR de 5 dimensiones oficiales |
| US-36 | Épica 6 | Exportar EBR documentada |
| US-37 | Épica 4 | Despacho administra varios sujetos obligados |
| US-38 | Épica 4 | Separar padrones/expedientes por cliente |
| US-39 | Épica 4 | Revender dictamen con marca propia |
| US-40 | **Fase 2 — diferido** | Screening desde agente Capafy (Out of Scope fase 1) |
| US-41 | Épica 4 | Resumen consolidado de cartera |
| US-42 | Épica 4 | Endpoint síncrono single-name |
| US-43 | Épica 4 | Endpoint batch asíncrono (job id) |
| US-44 | Épica 4 | Resultado batch por webhook/poll |
| US-45 | Épica 4 | Dictamen LFPIORPI en una llamada |
| US-46 | Épica 4 | API keys self-serve |
| US-47 | Épica 4 | Rate limits y errores documentados |
| US-48 | Épica 4 | Esquema versionado /v1/ + changelog |
| US-49 | Épica 4 | Webhooks de rescreening |
| US-50 | Épica 4 | Incrustar UI white-label |
| US-51 | Épica 1 + 4 | Tres caras → misma Core API |
| US-52 | Épica 4 | Metering por Cuenta |
| US-53 | Épica 1 + 3 | Hospedar (E1) y refrescar (E3) listas SAT 69/69-B |
| US-54 | Épica 1 | Cliente sube su copia de la LPB |
| US-55 | Épica 1 | Una sola integración OpenSanctions |
| US-56 | Épica 4 | Alternar OpenSanctions API ↔ self-hosted |
| US-57 | Épica 1 + 4 | Credenciales desde bóveda |
| US-58 | Épica 4 | Observabilidad de costo por corrida |
| US-59 | Épica 3 | Expediente muestra listas/versiones usadas |
| US-60 | Épica 3 | Cadena de decisiones (auto + manual) |
| US-61 | Épica 3 | Confirmar rescreening continuo y avisos |
| US-62 | Épica 8 **[BLOQUEADO AS-4]** | Cargar operaciones ligadas a cliente |
| US-63 | Épica 8 **[BLOQUEADO AS-4]** | Acumular operaciones en ventanas 6m |
| US-64 | Épica 8 **[BLOQUEADO AS-4]** | Cruce de umbral → reloj + XML |
| US-65 | Épica 8 **[BLOQUEADO AS-4]** | Marcar intento de operación (aviso 24h) |
| US-66 | Épica 8 **[BLOQUEADO AS-4]** | Marcar operaciones fuera de perfil |
| US-67 | Épica 5 | Declarar partes relacionadas |
| US-68 | Épica 5 | Screenear cada parte relacionada |
| US-69 | Épica 5 | Ligar evidencia de partes al expediente |
| US-70 | Épica 7 | Plantilla de manual de cumplimiento |
| US-71 | Épica 7 | Tracking de capacitación PLD |
| US-72 | Épica 7 | Segregación de funciones por rol (RBAC) |
| US-73 | Épica 7 | Registro de auditoría interna |
| US-74 | Épica 7 | Validar RFC/CURP/69-B (nivel barato) |
| US-75 | Épica 7 **[AS-3]** | Identidad INE/biometría/RENAPO (proveedor a definir) |
| US-76 | Épica 7 | Adjuntar validación de identidad al expediente |

**Cobertura NFR:** NFR-1 (Épica 1) · NFR-2 (Épica 4) · NFR-3 (Épica 3) · NFR-4 (Épica 1) · NFR-5 (Épica 4) · NFR-6 (Épica 4).

## Epic List

> **Alcance de detalle (directiva PRD §Secuenciación; confirmado por Jorge 2026-06-24):** solo las **Épicas 1–4** (capas 1–3: núcleo, portal, vigencia, canales) se cortan a detalle con historias en el Paso 3. Las **Épicas 5–7** (módulos de suite no bloqueados) se crean a nivel de épica y se detallarán en una pasada posterior. Las **Épicas 8–9** (monitoreo AS-4, avisos XML AS-2) se crean **bloqueadas, sin historias detalladas**, hasta confirmar el dato externo con el especialista PLD.

### Épica 1: Motor de dictamen verificable — "Sube padrón → dictamen + expediente" (capa 1)
La cuña del producto y el Nivel 1: un sujeto obligado (o un integrador) entrega un padrón y, por cada sujeto evaluado, el motor devuelve un **dictamen auditable** (banda de confianza, tipo de lista, score, riesgo, acción AML, fundamento) más un **expediente inmutable** conservable 10 años, exportable a PDF/Excel. Incluye la inicialización de la solución (Clean Architecture + Postgres/Marten + portal), el núcleo determinístico de homonimia (matriz `banda × tipo_lista`), la integración de fuentes de listas tras puerto (OpenSanctions/SAT 69-B/LPB del cliente), la redacción LLM con fidelidad prosa↔verdicto y degradación a plantilla, el contrato público `/v1/` (síncrono + batch) y el aislamiento por sujeto obligado. Verificable de extremo a extremo vía el contrato HTTP `/v1/` (seam principal de prueba). **Es el entregable cuya disposición a pagar se valida antes de apilar módulos.**
**FRs covered:** US-1, US-2, US-3, US-4, US-5, US-6, US-7, US-8, US-9, US-10, US-11, US-12, US-13, US-14, US-15, US-16, US-17, US-18, US-19, US-53 (hospedaje inicial), US-54, US-55, US-51 (parcial), US-57 (parcial) · NFR-1, NFR-4 · AR-1..6, AR-8, AR-12..18, AR-23, AR-27.

### Épica 2: Portal Vigía capa 1 — el sujeto obligado opera el motor desde la web
La cara visible del Nivel 1: la landing pública (venta + verificador en vivo + calculadora de multa) y el panel privado capa-1 (dashboard con KPIs y reloj 24h en vivo, subir padrón con dropzone+stepper+mapeo, tabla de dictámenes filtrable por banda, detalle del dictamen con fundamento/prosa/expediente, export). Implementa la **capa de marca Vigía** (sistema de tokens de 3 temas, semántica de banda, tipografía editorial-legal) sobre shadcn/ui, y los pisos duros de **accesibilidad WCAG 2.2 AA**, **rendimiento** (virtualización, CLS, fuentes) y **responsive**. Cascarón delgado sobre `/v1/` (Épica 1). Realiza el Flow 1 (Don Refugio sube su padrón).
**FRs covered:** realización en UI de US-1..19 (especialmente US-3, US-4, US-12, US-15, US-17) · auth de portal (Identity) · UX-DR-1..26.

### Épica 3: Vigencia y continuidad — rescreening, alertas y reloj de aviso 24h (capa 2)
El padrón se mantiene vivo: refresco automático de las listas SAT 69-B con detección de staleness, rescreening recurrente (Hangfire + `IClock`), alertas cuando un cliente ya cargado cae en una lista, y el **reloj de aviso de 24 horas** que arranca desde el evento origen (sin generar aún el XML — eso es Épica 9). Configuración de cadencia/alcance con piso legal visible (ADR-0005), historial de rescreenings, y las vistas que el auditor necesita para confirmar monitoreo continuo. Realiza el Flow 3 (reloj de aviso). Construye sobre Épicas 1–2.
**FRs covered:** US-20, US-21, US-22, US-23, US-24, US-53 (refresco recurrente + staleness), US-59, US-60, US-61 · NFR-3 · AR-16, AR-21.

### Épica 4: Canal API / white-label y gateway multi-cuenta (capa 3)
El motor se entrega por su canal comercial: gateway con **auth por API keys self-serve** (hasheadas, mostradas una vez), rate limits, catálogo de errores ProblemDetails, contrato `/v1/` versionado con changelog/deprecación, **webhooks vía Outbox firmados HMAC**, y el alternar OpenSanctions API↔self-hosted tras puerto. Habilita al **despacho multi-cliente** (selector de sujeto obligado, resumen consolidado, separación estricta de padrones/expedientes) y al **integrador/ERP** (síncrono + batch async en una llamada, embed white-label). Incluye **metering por Cuenta** y observabilidad de costo por corrida. Construye sobre Épicas 1–3.
**FRs covered:** US-37, US-38, US-39, US-41, US-42, US-43, US-44, US-45, US-46, US-47, US-48, US-49, US-50, US-51 (cierre), US-52, US-56, US-57 (cierre), US-58 · NFR-2, NFR-5, NFR-6 · AR-7, AR-9, AR-11, AR-19, AR-20, AR-22, AR-24, AR-25, AR-26.

### Épica 5: Beneficiario Controlador y partes relacionadas (capa 4 — suite)
Registrar y conservar al BC declarado y a las partes relacionadas (socios, avales, beneficiarios, propietarios reales) de clientes persona moral, screenearlos contra los mismos tipos de lista que al cliente, y ligar su evidencia y cadena de control al expediente (ADR-0006, sin resolución recursiva). Construye sobre Épica 1.
**FRs covered:** US-30, US-31, US-32, US-33, US-67, US-68, US-69.

### Épica 6: EBR — matriz de riesgo de 5 dimensiones (capa 4 — suite)
Clasificación EBR por las cinco dimensiones oficiales (clientes/usuarios, geografía, productos/servicios, canales de distribución, perfil transaccional), consumiendo el nivel de riesgo del dictamen, documentada y exportable para el programa PLD y la auditoría (Art. 18 Fr. VII). Construye sobre Épica 1.
**FRs covered:** US-34, US-35, US-36.

### Épica 7: Programa PLD y validación de identidad (capa 4 — suite)
Herramientas de programa: plantilla de manual de cumplimiento rellenada con la configuración del sujeto obligado, tracking de capacitación, **segregación de funciones por rol** (screener/aprobador/auditor) y registro de auditoría interna; más **validación de identidad** nivel barato (RFC/CURP/69-B) en fase 1 y nivel reforzado (INE/biometría/RENAPO) por integración tras puerto, adjuntando el resultado al expediente. Construye sobre Épicas 1 y 4 (roles).
**FRs covered:** US-70, US-71, US-72, US-73, US-74, US-75 **[reforzada bloqueada por AS-3]**, US-76.

### Épica 8: Monitoreo transaccional (capa 4 — suite) — 🔒 BLOQUEADA (AS-4)
Ingesta de operaciones ligadas a un cliente, acumulación en ventanas móviles de 6 meses, umbrales por actividad vulnerable que disparan el reloj de aviso, marcado de intento de operación y de operaciones fuera de perfil por reglas (ADR-0008, sin IA en fase 1). **Arquitectónicamente soportada, pero sin historias detalladas:** la tabla de umbrales reportables (reforma 2026) no está confirmada (**AS-4**). No se inventan umbrales. Se detalla cuando el especialista PLD provea la tabla versionada.
**FRs covered:** US-62, US-63, US-64, US-65, US-66.

### Épica 9: Avisos XML SPPLD (capa 4 — suite) — 🔒 BLOQUEADA (AS-2)
Autogeneración del XML del aviso (operación reportable y aviso 24h) a partir del hit/operación, validación contra el esquema oficial del SAT, distinción de tipos de aviso, y registro de fecha/acuse de presentación. **Arquitectónicamente soportada, pero sin historias detalladas:** la existencia/estabilidad del XSD público del SAT no está confirmada (**AS-2**, ADR-0010). No se infiere el formato del esquema. Se detalla cuando se confirme el XSD oficial con el especialista/portal SAT.
**FRs covered:** US-25, US-26, US-27, US-28, US-29.

---

## Epic 1: Motor de dictamen verificable — "Sube padrón → dictamen + expediente"

La cuña del producto (Nivel 1). El sujeto obligado o el integrador entrega un padrón y el motor devuelve, por cada sujeto evaluado, un dictamen auditable más un expediente inmutable conservable 10 años, verificable de extremo a extremo vía el contrato HTTP `/v1/`. Las historias se ordenan para que cada una se complete apoyándose solo en las anteriores.

### Story 1.1: Inicializar la solución y el esqueleto del proyecto

As a operador del producto (Jorge),
I want inicializar la solución backend (Clean Architecture + PostgreSQL + Marten) y el portal frontend (Vite + React + TS) con CI desde el primer commit,
So that el equipo construya cada feature sobre una base con capas, puertos y pruebas de fábrica, sin reinventar convenciones.

**Acceptance Criteria:**

**Given** un repositorio greenfield sin código
**When** se genera el backend con la plantilla Clean Architecture (`dotnet new ca-sln --client-framework None --database PostgreSQL`)
**Then** la solución compila en Windows y en Linux con .NET 10 LTS
**And** MediatR queda sustituido por el source-generator `Mediator` (MIT) o handlers propios, sin dependencia de licencia comercial
**And** Marten 8.x queda montado en `Infrastructure/Persistence` como event store sobre Postgres, y EF Core queda para CRUD/catálogos

**Given** la solución inicializada
**When** se levanta el entorno de desarrollo con `docker-compose up`
**Then** arrancan Postgres y la app en contenedores Linux, y la API responde un health check en `/health`

**Given** el portal frontend
**When** se inicializa Vite + React-TS + Tailwind v4 + shadcn/ui + TanStack Query + React Router
**Then** el portal compila y sirve una página base, desacoplado del backend

**Given** el repositorio con backend y portal
**When** se hace push a la rama
**Then** GitHub Actions ejecuta build + `dotnet test` + build de imagen Linux + build del portal, y el pipeline pasa en verde
**And** el `README.md` incluye el mapping glosario↔código (autoridad `CONTEXT.md`)

### Story 1.2: Contexto de Cuenta y sujeto obligado con aislamiento de datos

As a operador del producto,
I want que toda entidad de dominio viva bajo un sujeto obligado y que cada consulta se filtre por el `sujeto_obligado_id` del principal autenticado (RLS + filtro de repositorio),
So that una Cuenta-despacho con varios sujetos obligados nunca pueda leer datos cruzados, garantizando el aislamiento que la ley y ADR-0003 exigen.

**Acceptance Criteria:**

**Given** el modelo de dominio inicial
**When** se definen los agregados `Cuenta` y `SujetoObligado`
**Then** existe un contexto de tenencia que expone el `sujeto_obligado_id` del principal autenticado mediante un seam inyectable (fakeable en pruebas), nunca tomado del input del request

**Given** una consulta a cualquier repositorio
**When** se ejecuta dentro del contexto de un sujeto obligado
**Then** Row-Level Security de Postgres y un filtro obligatorio en el repositorio restringen los resultados a ese `sujeto_obligado_id`
**And** el término `tenant` no aparece en código ni en esquema (proscrito, CONTEXT.md)

**Given** dos sujetos obligados (A y B) dentro de una misma Cuenta-despacho
**When** un test adversarial intenta leer datos de B operando en el contexto de A (IDOR sweep)
**Then** ningún dato de B es accesible por ningún camino, y la prueba de fuga cruzada pasa

### Story 1.3: Ingesta de padrón con mapeo y validación de columnas

As a sujeto obligado,
I want subir mi padrón en Excel/CSV con un mapeo de columnas y una validación que marque filas inválidas o duplicadas antes de procesar,
So that pueda corregir un archivo mal formado y cruzar mi padrón sin capturar nombre por nombre ni procesar basura.

**Acceptance Criteria:**

**Given** un archivo `.xlsx`/`.xls`/`.csv`
**When** se sube al motor
**Then** se detecta el encabezado y se propone un mapeo de columnas a los campos del `SujetoEvaluado` (nombre, RFC, fecha de nacimiento, tipo de persona)

**Given** un archivo con columnas no autodetectadas
**When** falta mapear un campo obligatorio (al menos `nombre`)
**Then** el motor reporta que se requiere asignar la columna a mano y **no permite procesar** hasta resolverlo

**Given** un padrón con filas sin nombre o duplicadas (misma clave RFC, o nombre+fecha cuando no hay RFC)
**When** se valida antes de procesar
**Then** se reportan al usuario las filas inválidas y duplicadas con su número de fila, para corregir y re-subir

**Given** un padrón vacío o de un solo registro
**When** se procesa
**Then** el resultado es válido (posiblemente vacío) y nunca produce un error 500

### Story 1.4: Padrón mínimo de nombres que aprovecha RFC y fecha de nacimiento

As a sujeto obligado,
I want que el sistema acepte un mínimo de solo nombres pero aproveche el RFC y la fecha de nacimiento cuando los tengo,
So that pueda empezar con datos pobres y reducir falsos positivos a medida que enriquezco el padrón.

**Acceptance Criteria:**

**Given** un padrón que solo trae nombres
**When** se ingesta
**Then** cada `SujetoEvaluado` se acepta como válido para screening, sin exigir RFC ni fecha

**Given** un padrón que trae RFC y/o fecha de nacimiento
**When** se ingesta
**Then** el RFC se valida en su formato y se modela como value object (`Rfc`), y la fecha se normaliza, quedando disponibles para endurecer el cruce posterior

**Given** un RFC con formato inválido
**When** se ingesta la fila
**Then** se conserva el nombre para screening y se marca el RFC como no utilizable para resolución, sin descartar la fila

### Story 1.5: Puerto de OpenSanctions con sello de versión de dataset

As a operador del producto,
I want una única integración con OpenSanctions detrás de un puerto (`IOpenSanctionsPort`) que selle la versión del dataset devuelta en cada respuesta,
So that el motor tenga un solo medidor y punto de control de costo, sea fakeable en pruebas, y cada expediente sea reproducible aun cuando el scoring de OpenSanctions cambie.

**Acceptance Criteria:**

**Given** un sujeto evaluado con nombre (+ RFC + fecha cuando existan)
**When** el motor consulta el match
**Then** la llamada cruza el puerto `IOpenSanctionsPort` (adaptador API `/match` en `Infrastructure`), enviando nombre + RFC + fecha de nacimiento

**Given** una respuesta de OpenSanctions
**When** se recibe el resultado de match
**Then** se captura el score, las listas/programas de coincidencia y la **versión del dataset** que devuelve la respuesta, para sellarla luego en el expediente (mitiga R2)

**Given** OpenSanctions no disponible o con error
**When** se agotan los reintentos (backoff exponencial + circuit breaker)
**Then** el motor degrada (no falla la corrida) y marca el resultado como "sanciones pendientes" para ese sujeto evaluado

**Given** un entorno de pruebas
**When** se ejecutan tests del núcleo o de contrato
**Then** el puerto se reemplaza por un fake sin red real ni costo

### Story 1.6: Cruce contra fuentes locales — SAT 69/69-B hospedado y LPB del cliente

As a sujeto obligado,
I want que el motor cruce mi padrón también contra las listas SAT 69/69-B (que el operador hospeda) y la LPB de la UIF (que yo subo),
So that el dictamen identifique coincidencias EFOS-EDOS y de bloqueados-UIF, sin que se hospede centralmente una lista confidencial.

**Acceptance Criteria:**

**Given** la fuente SAT 69/69-B
**When** el operador la carga al motor tras el puerto `ISatListaPort`
**Then** queda disponible para cruzar el padrón, con su versión registrada para el sellado del expediente

**Given** la LPB de la UIF (confidencial)
**When** el sujeto obligado sube su propia copia
**Then** la LPB se cifra y se asocia a su `sujeto_obligado_id`, nunca se hospeda centralmente ni se persiste más de lo necesario

**Given** un sujeto evaluado que coincide en una de estas listas
**When** se arma el resultado
**Then** se determina el `tipo_lista` (`efos-edos` o `bloqueados-uif`) y el `programa` correspondiente

**Given** la LPB de un sujeto obligado A
**When** se procesa el padrón del sujeto obligado B
**Then** la LPB de A nunca se usa ni es accesible para B (aislamiento)

### Story 1.7: Núcleo determinístico — banda de confianza y matriz (banda × tipo_lista)

As a sujeto obligado,
I want que reglas determinísticas resuelvan la banda de confianza de cada sujeto evaluado y deriven su acción AML y nivel de riesgo de la matriz `(banda × tipo_lista)`,
So that no trate a un homónimo como criminal ni dé por limpio lo que falta verificar, y la decisión legalmente sensible nunca dependa del LLM.

**Acceptance Criteria:**

**Given** un nombre que coincide y RFC **y** fecha de nacimiento iguales a la lista
**When** se resuelve la banda
**Then** la banda es `resuelto-real`

**Given** un nombre que coincide pero RFC/fecha **difieren**
**When** se resuelve la banda
**Then** la banda es `resuelto-homonimo`

**Given** un nombre con score alto pero **faltan** RFC/fecha para resolver
**When** se resuelve la banda
**Then** la banda es `requiere-verificacion` y el campo `fundamento` indica exactamente qué dato pedir (RFC y/o fecha de nacimiento)

**Given** un score bajo sin coincidencia relevante
**When** se resuelve la banda
**Then** la banda es `descartado-por-nombre`

**Given** una banda resuelta y un `tipo_lista`
**When** se derivan `accion_sugerida` y `nivel_riesgo`
**Then** se obtienen de la matriz `(banda × tipo_lista)`, separando resolución de identidad de consecuencia AML: un `pep` **nunca** produce `accion_sugerida = rechazar` (dispara EDD), y `efos-edos` se trata como factor de riesgo, no como "no operar"

**Given** la conversión del score continuo de OpenSanctions a bandas
**When** se aplica el corte
**Then** se usa la política de corte versionada del operador (ADR-0001), cuya versión queda disponible para sellar en el expediente
**And** el término `indeterminado` no aparece en ningún resultado (proscrito, CONTEXT.md)

### Story 1.8: Redacción del dictamen con LLM y fidelidad prosa↔verdicto

As a sujeto obligado,
I want que el dictamen se redacte en español claro encuadrado en la LFPIORPI, citando literalmente la banda, el tipo de lista y la acción del verdicto, y que caiga a una plantilla si el LLM contradice o no está disponible,
So that pueda entregarlo a mi auditor sin reescribirlo y el expediente nunca contenga una contradicción interna ni dependa de la disponibilidad del LLM.

**Acceptance Criteria:**

**Given** un verdicto determinístico completo
**When** la capa de orquestación invoca el LLM (Claude Sonnet 4.6 en AWS Bedrock) tras el puerto `ILlmRedactorPort`
**Then** la prosa redactada cita literalmente la `banda_confianza`, el `tipo_lista` y la `accion_sugerida` del verdicto, y el dictamen se marca `redaccion: LLM`

**Given** una prosa del LLM que contradice u omite la banda/tipo_lista/acción del verdicto
**When** se ensambla el dictamen
**Then** la prosa se rechaza y el dictamen cae a la plantilla de respaldo con los campos determinísticos completos, marcado `redaccion: plantilla`

**Given** Bedrock no disponible
**When** se solicita la redacción
**Then** el dictamen se emite igual con plantilla y todos los campos determinísticos, sin degradar la disponibilidad del screening

**Given** un sujeto evaluado `descartado-por-nombre`
**When** se arma su dictamen
**Then** se usa plantilla sin invocar el LLM (control de costo)

### Story 1.9: Contrato `/v1/` del dictamen — screening síncrono de un nombre

As a sujeto obligado,
I want verificar a un solo cliente en tiempo real durante el onboarding mediante un endpoint síncrono que devuelve el dictamen completo,
So that pueda decidir si doy de alta a un cliente antes de cerrar la operación.

**Acceptance Criteria:**

**Given** una solicitud de screening de un solo nombre a `/v1/` (síncrona)
**When** el motor la procesa (match + resolución de banda + redacción/plantilla)
**Then** responde con el esquema de dictamen completo: `nombre, rfc, fecha_nacimiento, tipo_persona, coincidencia, lista_fuente, tipo_lista, programa, score, banda_confianza, fundamento, nivel_riesgo, accion_sugerida, redaccion, fecha_corte, version_lista, version_politica_corte`

**Given** una solicitud síncrona
**When** se mide la latencia extremo a extremo
**Then** el p95 es < 3 s (NFR-1), incluyendo `/match` y redacción o plantilla

**Given** una solicitud mal formada o un límite excedido
**When** falla la validación
**Then** se responde con ProblemDetails (RFC 7807) con `type` del catálogo versionado, sin exponer stack traces

**Given** los valores de enum en la respuesta JSON
**When** se serializa el dictamen
**Then** se emiten como tokens canónicos del glosario en kebab (`resuelto-real`, `bloqueados-uif`, etc.), fechas en ISO 8601 UTC y montos `decimal`

### Story 1.10: Screening de padrón completo y re-procesamiento incremental

As a sujeto obligado,
I want procesar mi padrón completo y, al re-subirlo o agregar clientes, que se rescree solo lo que cambió,
So that obtenga el dictamen de toda mi cartera y no reprocese ni pague de más por lo que no cambió.

**Acceptance Criteria:**

**Given** un padrón validado de hasta 10,000 sujetos evaluados
**When** se procesa
**Then** el motor produce un dictamen por cada sujeto evaluado con el mismo esquema que el endpoint síncrono

**Given** un padrón ya procesado
**When** el sujeto obligado re-sube una versión actualizada o agrega clientes nuevos
**Then** el motor identifica qué sujetos evaluados son nuevos o cambiaron y rescrea solo esos, conservando los dictámenes vigentes de los no modificados

**Given** un padrón que excede el tamaño máximo permitido por corrida
**When** se intenta procesar
**Then** se rechaza con ProblemDetails (`413`/`422`) y no se ejecuta una corrida a medias

### Story 1.11: Expediente inmutable event-sourced con sello de versiones

As a auditor / sujeto obligado,
I want que cada dictamen quede registrado como evento inmutable con la versión de cada lista, de la política de corte y del matcher usados,
So that el expediente sea prueba completa e inalterable ante el Art. 12 Bis durante 10 años.

**Acceptance Criteria:**

**Given** un dictamen emitido
**When** se persiste
**Then** se registra el evento `DictamenEmitido` en el event store (Marten), y una proyección arma el `Expediente` recuperable

**Given** un expediente generado
**When** se consulta
**Then** contiene qué se revisó, cuándo, contra qué listas y **qué versión** de cada lista, de la política de corte y del matcher de OpenSanctions, qué coincidió y qué se decidió

**Given** un evento ya registrado
**When** cualquier operación intenta modificarlo
**Then** no es posible mutar un evento pasado (append-only); los eventos de dominio van en español, en pasado, como `record` inmutables

**Given** una corrida histórica de hace meses
**When** se solicita su expediente
**Then** se recupera íntegro y reproducible

### Story 1.12: Ajuste manual del dictamen registrado como evento auditable

As a sujeto obligado,
I want revisar y aprobar o ajustar manualmente un dictamen, quedando registrado quién, cuándo y por qué,
So that pueda ejercer mi criterio cuando los datos son ambiguos sin que se pueda alegar que alteré la evidencia.

**Acceptance Criteria:**

**Given** un dictamen emitido
**When** el sujeto obligado lo aprueba o ajusta manualmente
**Then** el ajuste se registra como un **evento nuevo** (`AjusteManualRegistrado`) con `revisado_por`, `revisado_en`, `ajuste_manual` y `justificacion`, **sin sobrescribir** el dictamen original

**Given** un dictamen con ajustes manuales
**When** se consulta su expediente
**Then** se ve la cadena completa de decisiones: la automática y cada ajuste con su justificación

**Given** una acción irreversible (aprobar dictamen)
**When** el usuario la inicia
**Then** se pide confirmación antes de registrarla

### Story 1.13: Almacén WORM de expedientes y export PDF/Excel

As a sujeto obligado,
I want exportar el expediente en PDF y Excel y que su versión binaria se conserve en almacenamiento inmutable verificado,
So that pueda entregarlo en una auditoría y demostrar que no se pudo alterar, cumpliendo la conservación de 10 años.

**Acceptance Criteria:**

**Given** un expediente generado
**When** el sujeto obligado lo exporta
**Then** obtiene un PDF y un Excel con el dictamen, el fundamento, los cotejos y las versiones de listas/política

**Given** el almacén de objetos tras el puerto `IAlmacenObjetosPort`
**When** se persiste un binario de expediente
**Then** se guarda con cifrado en reposo y con S3 Object Lock en modo compliance (retención 10 años), accesible solo por su sujeto obligado

**Given** la configuración de inmutabilidad WORM
**When** corre la suite de pruebas
**Then** un test verifica el Object Lock set-at-creation **antes** de permitir persistir cualquier expediente real (mitiga R3)

**Given** una falla del almacén de objetos al guardar el binario
**When** el evento de dictamen ya se registró en el event store
**Then** el dictamen persiste en la base aunque el binario falle (desacoplados), y el binario se reintenta sin perder el rastro

---

## Epic 2: Portal Vigía capa 1 — el sujeto obligado opera el motor desde la web

La cara visible del Nivel 1: landing pública (venta) y panel privado capa-1, como cascarón delgado sobre el contrato `/v1/` de la Épica 1. Implementa la capa de marca Vigía (tokens de 3 temas, semántica de banda, tipografía editorial-legal) sobre shadcn/ui y realiza el Flow 1 (Don Refugio sube su padrón). Las historias se ordenan de la base de diseño hacia las superficies.

### Story 2.1: Sistema de tokens y temas Vigía con foco y contraste accesibles

As a usuario del portal,
I want una capa de marca con tokens de color/tipografía/espaciado y la semántica de banda de confianza, con foco visible y contraste verificado,
So that el color comunique el dictamen de un vistazo y la interfaz sea accesible y consistente en los tres temas.

**Acceptance Criteria:**

**Given** la base shadcn/ui + Tailwind v4
**When** se implementa la capa de marca Vigía
**Then** existen tokens nombrados para los tres temas (Toga canónico, Notario, Salvaguarda) con los mismos nombres, y ninguna vista mezcla tokens de dos temas (UX-DR-1)

**Given** la semántica de banda de confianza
**When** se definen sus tokens
**Then** cada banda tiene tres roles separados —hue vivo (`banda-*`), fondo (`banda-*-bg`) y texto (`banda-*-text`)—, con Bloqueo-UIF como único badge invertido, y nunca se usa el hue vivo como color de texto (UX-DR-2)

**Given** la tipografía
**When** se cargan las familias
**Then** Spectral (display/prosa), Hanken Grotesk (UI/cuerpo) e IBM Plex Mono (todo dato) cumplen sus roles, y todo dato `mono` usa `font-variant-numeric: tabular-nums` (UX-DR-3, UX-DR-4)

**Given** las 15 combinaciones banda×tema (`banda-*-text` sobre `banda-*-bg`)
**When** se verifican antes de release
**Then** todas alcanzan ≥4.5:1, igual que `on-hero` sobre `hero` e `ink`/`ink-soft` sobre superficies; `muted` no se usa como texto de cuerpo/dato (UX-DR-17)

**Given** cualquier elemento interactivo
**When** recibe foco por teclado
**Then** muestra un anillo de foco visible adaptativo a la superficie (`accent` en claro, claro `on-hero`/#fff sobre el sidebar oscuro), ≥3:1 (UX-DR-18); no queda `outline:none` sin reemplazo

### Story 2.2: Landing pública con urgencia factual y disclaimers

As a sujeto obligado prospecto,
I want una landing que explique el producto, la urgencia de la reforma 2026 y la cobertura de listas, con cifras de multa con fuente y disclaimers claros,
So that entienda el valor y el alcance sin que se me prometa inmunidad ante multas.

**Acceptance Criteria:**

**Given** la landing pública (scroll de una página)
**When** se carga
**Then** presenta las secciones Nav, Hero (urgencia con badge reforma + multa), Cómo funciona (4 pasos), Cobertura de listas, Precios (Esencial/Negocio/Despacho), Trust + disclaimers y Footer (UX-DR-6)

**Given** cualquier cifra de multa
**When** se muestra
**Then** va acompañada de su fuente (UMA 2026 · DOF), nunca como cifra alarmista sin fuente (UX-DR-15)

**Given** las secciones de la landing
**When** se lee el microcopy
**Then** el disclaimer "Herramienta de apoyo, no garantía. El aviso lo presenta el sujeto obligado" está presente y no se promete cumplimiento garantizado (UX-DR-15)

### Story 2.3: Verificador en vivo de un nombre (lead magnet, sin registro)

As a visitante de la landing,
I want verificar un solo nombre en vivo, sin registrarme, y ver su dictamen con banda,
So that compruebe el valor del producto antes de crear una cuenta.

**Acceptance Criteria:**

**Given** el verificador en la landing
**When** escribo un nombre o elijo un chip de ejemplo
**Then** se ejecuta una animación de cotejo (~1.25 s) y se muestra una tarjeta de resultado con la banda de confianza, consumiendo el endpoint síncrono `/v1/` (UX-DR-11)

**Given** el resultado del verificador
**When** se presenta la banda
**Then** lleva etiqueta textual + punto (nunca solo color) y el score con su valor numérico + etiqueta de riesgo (UX-DR-19)

**Given** `prefers-reduced-motion: reduce`
**When** corre la animación de cotejo
**Then** el scanline pasa a un indicador no animado, sin perder información (UX-DR-21)

### Story 2.4: Autenticación del portal y selector de sujeto obligado activo

As a sujeto obligado o despacho,
I want entrar al panel con mi cuenta y operar en el contexto de un sujeto obligado activo (o cambiarlo si soy despacho),
So that trabaje sobre los datos correctos sin mezclar información entre clientes.

**Acceptance Criteria:**

**Given** un usuario con credenciales
**When** inicia sesión (ASP.NET Core Identity)
**Then** accede al panel y su sesión queda asociada a su Cuenta

**Given** un despacho con varios sujetos obligados
**When** abre el selector del sidebar
**Then** ve la lista de sus sujetos obligados y una opción "Todos"; al cambiar de contexto se recargan las vistas del panel (UX-DR-7)

**Given** un sujeto obligado directo (único)
**When** entra al panel
**Then** se muestra su único contexto sin selector activo

**Given** el contexto de un sujeto obligado activo
**When** se renderiza cualquier vista
**Then** nunca expone datos de otro sujeto obligado de la Cuenta (UX-DR-7, consistente con aislamiento de Épica 1)

### Story 2.5: Shell del panel — sidebar, responsive y módulos bloqueados visibles

As a usuario del panel,
I want una navegación con sidebar que se adapte a móvil y muestre los módulos próximos como "pendientes de configuración legal",
So that navegue cómodamente en cualquier dispositivo y entienda qué funciones llegarán después.

**Acceptance Criteria:**

**Given** el panel en escritorio (`≥lg`)
**When** se renderiza
**Then** muestra el sidebar fijo de 248px (fondo `hero`) con el item activo resaltado y el main con scroll propio (UX-DR-8)

**Given** breakpoints menores
**When** la pantalla baja a `md`
**Then** el sidebar colapsa a iconos; en `<md`/`sm` se vuelve un `Sheet` (drawer) desde una top bar (UX-DR-26)

**Given** los módulos bloqueados (Monitoreo AS-4 / Avisos AS-2)
**When** aparecen en la navegación
**Then** se muestran con estado visible "Próximamente — pendiente de configuración legal", no ocultos (UX-DR-10)

**Given** objetivos táctiles en móvil
**When** se interactúa con filas, chips o botones del drawer
**Then** miden ≥24px (idealmente 44px) y son operables por teclado (UX-DR-22)

### Story 2.6: Subir padrón con dropzone, stepper, mapeo y validación

As a sujeto obligado,
I want subir mi padrón arrastrando el archivo, mapear columnas y ver las filas inválidas/duplicadas antes de procesar,
So that corrija mi Excel y obtenga dictámenes limpios (Flow 1).

**Acceptance Criteria:**

**Given** la superficie Subir padrón
**When** se abre
**Then** muestra un dropzone (`2px dashed`) con un stepper de 3 pasos (Cargar/Mapear/Resultado) y un botón "Usar archivo de ejemplo" (UX-DR-12)

**Given** un archivo `.xlsx`/`.xls`/`.csv` de hasta 10,000 sujetos
**When** se carga
**Then** se ofrece el mapeo de columnas editable y se reportan filas inválidas/duplicadas antes de procesar (UX-DR-12, realiza US-3/US-4 en UI)

**Given** una corrida en proceso
**When** el motor coteja
**Then** se muestra un spinner + log mono de listas cotejadas y, al terminar, un resumen (reales/homónimos/a verificar/limpios) (UX-DR-13)

**Given** un archivo con columnas irreconocibles
**When** no se autodetecta el mapeo
**Then** se pide asignar "Nombre" y "RFC" a mano y no se permite procesar hasta resolverlo (Flow 1, caso de falla)

### Story 2.7: Dashboard de resumen con KPIs y estado del padrón

As a sujeto obligado,
I want un dashboard que resuma mi padrón (KPIs, hits por banda, corridas recientes y reloj de aviso),
So that entienda de un vistazo qué es urgente al entrar.

**Acceptance Criteria:**

**Given** un sujeto obligado con padrón procesado
**When** abre el dashboard
**Then** ve KPIs en mono (padrón/hits/reales/avisos), hits por banda, corridas recientes y la cobertura de listas (UX-DR-8)

**Given** un sujeto obligado sin padrón
**When** abre el dashboard
**Then** ve "Aún no has subido tu padrón." con CTA "Subir padrón", nunca una pantalla en blanco (UX-DR-13)

**Given** contenido que carga de forma asíncrona
**When** se renderiza el dashboard
**Then** los KPIs y tarjetas usan Skeleton con la forma/dimensiones reales y reservan su caja, evitando saltos de layout (CLS < 0.1) al repoblarse (UX-DR-24)

### Story 2.8: Tabla de dictámenes filtrable, virtualizada y responsive

As a sujeto obligado,
I want una tabla de dictámenes filtrable por banda, que rinda con cientos de hits y se adapte a móvil,
So that priorice qué casos revisar primero.

**Acceptance Criteria:**

**Given** los dictámenes de una corrida
**When** se listan
**Then** se muestran en una tabla con score y banda visibles sin abrir, y clic en cualquier parte de la fila abre el Detalle (UX-DR-5, UX-DR-14)

**Given** los chips de filtro (Todas/Reales/Homónimos/A verificar/Descartados con conteo)
**When** selecciono uno
**Then** la tabla filtra por esa banda y el chip activo se resalta con `accent`

**Given** una corrida con cientos de hits
**When** se renderiza la lista
**Then** la tabla se virtualiza (solo filas visibles) y pagina (sin scroll infinito) (UX-DR-23, UX-DR-14)

**Given** la vista en móvil (`<md`)
**When** la tabla colapsa a tarjetas
**Then** cada campo conserva su etiqueta visible (Sujeto/Tipo de lista/Score/Banda/Riesgo) (UX-DR-26)

**Given** la banda y el score
**When** se muestran
**Then** la banda lleva etiqueta + punto y el score su valor + etiqueta de riesgo (Alto/Medio/Bajo), nunca solo color (UX-DR-19)

### Story 2.9: Detalle del dictamen con fundamento, prosa y expediente

As a sujeto obligado,
I want abrir el detalle de un dictamen y ver el veredicto, su fundamento y cotejos, la prosa, y el expediente, pudiendo capturar un dato faltante o ajustar manualmente,
So that confíe en el dictamen, lo defienda ante el auditor y ejerza mi criterio.

**Acceptance Criteria:**

**Given** un dictamen
**When** abro su detalle
**Then** muestra el veredicto (banda + tipo de lista + acción), el fundamento con los cotejos (qué dato coincidió/faltó), la prosa con su badge (`LLM` o `plantilla`) y el expediente como timeline (realiza US-10/US-11/US-14/US-16 en UI)

**Given** un dictamen `requiere-verificación`
**When** se presenta
**Then** se muestra en ámbar indicando **qué dato pedir** ("Falta RFC y fecha de nacimiento para resolver") con la acción de capturar el dato y re-resolver (UX-DR-13)

**Given** el LLM caído
**When** se muestra el detalle
**Then** el dictamen aparece completo con prosa de plantilla y badge `redacción: plantilla`, con opción "Re-redactar" cuando el servicio vuelva (UX-DR-13)

**Given** una acción irreversible (aprobar dictamen)
**When** la inicio
**Then** se pide confirmación; filtrar o navegar no piden confirmación (UX-DR-14, realiza US-12)

### Story 2.10: Exportar y descargar el expediente histórico

As a sujeto obligado,
I want exportar el expediente en PDF/Excel y descargar el de cualquier corrida pasada,
So that conserve la evidencia y responda un requerimiento del SAT/UIF años después.

**Acceptance Criteria:**

**Given** un dictamen o expediente abierto
**When** elijo exportar
**Then** descargo el expediente en PDF y Excel desde la UI (realiza US-15)

**Given** corridas históricas
**When** consulto el histórico
**Then** puedo localizar y descargar el expediente de cualquier corrida pasada (realiza US-17)

### Story 2.11: Gate de accesibilidad y rendimiento del portal (release floor)

As a operador del producto,
I want que el portal cumpla un piso verificado de accesibilidad WCAG 2.2 AA y de rendimiento antes de cada release,
So that la herramienta sea usable por todos y no introduzca jitter ni regresiones en el elemento de mayor consecuencia legal (el reloj).

**Acceptance Criteria:**

**Given** toda la web responsive
**When** se audita antes de release
**Then** cumple WCAG 2.2 AA: orden de Tab = orden de lectura, `Esc` cierra modal/popover, `lang="es"`, y filas/chips operables por teclado (UX-DR-16, UX-DR-22)

**Given** el reloj de aviso 24h en pantalla
**When** el contador cambia cada segundo
**Then** el tick es `aria-hidden` y una región `aria-live="polite"` separada anuncia solo cruces de umbral (<12h, <6h) (UX-DR-20)

**Given** elementos con movimiento (dot pulsante, scanline, spinner)
**When** el usuario activó `prefers-reduced-motion: reduce`
**Then** pasan a estados estáticos de alto contraste, sin que ninguna información dependa del movimiento (UX-DR-21)

**Given** las tres familias tipográficas
**When** se cargan
**Then** usan `font-display: swap`, `preload` solo de las críticas above-the-fold y subset `latin`; landing y panel usan code-splitting por ruta; la latencia de interacción al tap es <100 ms (UX-DR-25)

---

## Epic 3: Vigencia y continuidad — rescreening, alertas y reloj de aviso 24h

El padrón se mantiene vivo: refresco de listas con detección de staleness, rescreening recurrente determinista, alertas de coincidencia sobreviniente y el reloj de aviso de 24 horas que arranca desde el evento origen (sin generar aún el XML — eso es la Épica 9). Realiza el Flow 3. Construye sobre Épicas 1–2.

### Story 3.1: Refresco automático de listas SAT 69-B con detección de staleness

As a operador del producto,
I want que un job refresque automáticamente las listas SAT 69/69-B y alerte si una lista no se refresca dentro de su ventana esperada,
So that el cruce siempre use la versión vigente y nunca se selle "versión vigente" sobre una lista vieja en silencio.

**Acceptance Criteria:**

**Given** un job de Hangfire de refresco de la fuente SAT
**When** se ejecuta con éxito
**Then** actualiza la lista tras el puerto `ISatListaPort` y registra su `ultima_actualizacion_exitosa` con la nueva versión

**Given** una lista que no se refresca dentro de su ventana esperada
**When** el job falla o no corre
**Then** se dispara una alerta de staleness, y un dictamen que corra contra esa lista puede marcar una advertencia (no bloquea)

**Given** el cambio silencioso de URL/columnas/cadencia del portal SAT
**When** el job no puede parsear la fuente
**Then** falla de forma observable (no en silencio) y mantiene la última versión válida marcada como stale

### Story 3.2: Rescreening recurrente del padrón (determinista vía clock inyectable)

As a sujeto obligado,
I want que mi padrón se rescree automáticamente cuando las listas se actualizan,
So that detecte a un cliente que cae en una lista después de darlo de alta, sin gestionarlo yo.

**Acceptance Criteria:**

**Given** una abstracción de tiempo `IClock` inyectable (UTC en almacenamiento, zona MX explícita)
**When** los jobs y el reloj la usan
**Then** nunca se usa `DateTime.Now` directo y los tiempos son deterministas en pruebas (incluye pruebas de DST)

**Given** un job de Hangfire de rescreening recurrente
**When** se ejecuta según la cadencia configurada
**Then** rescrea el padrón contra las versiones vigentes de las listas y registra el evento `RescreeningEjecutado`

**Given** una indisponibilidad del LLM o de OpenSanctions durante el rescreening
**When** el job corre
**Then** el rescreening determinístico se completa con degradación (plantilla / "sanciones pendientes"), sin tumbar la disponibilidad del screening (NFR-3)

### Story 3.3: Alerta de coincidencia real sobreviniente

As a sujeto obligado,
I want recibir una alerta cuando un cliente existente se vuelve coincidencia real tras un rescreening,
So that actúe a tiempo.

**Acceptance Criteria:**

**Given** un rescreening que detecta que un sujeto evaluado ya cargado pasa a `resuelto-real`
**When** se registra el cambio
**Then** se emite el evento `CoincidenciaDetectada` y se genera una alerta visible en el dashboard del sujeto obligado

**Given** una coincidencia real sobreviniente
**When** se presenta en el dashboard
**Then** sube al tope de pendientes y queda lista para la emisión de webhook (entrega vía Outbox, productizada en la Épica 4)

### Story 3.4: Reloj de aviso de 24 horas que cuenta desde el evento origen

As a sujeto obligado,
I want que el reloj del aviso de 24 horas arranque al detectarse una coincidencia o un intento de operación reportable, contando desde el evento y no desde que yo lo veo,
So that no pierda el plazo legal del aviso.

**Acceptance Criteria:**

**Given** un evento `CoincidenciaDetectada`, `IntentoDeOperacion` o `UmbralAcumulado`
**When** ocurre
**Then** el `RelojDeAviso` arranca sellando `evento_origen` y `marca_de_tiempo_origen`, y cuenta desde el evento (no desde la lectura), de forma determinista vía `IClock`

**Given** un reloj de aviso activo
**When** el sujeto obligado abre el dashboard o el detalle
**Then** ve el contador con color de urgencia (<6h rojo, <12h ámbar) y, en el detalle, la acción "Generar aviso XML (SPPLD)" (que en esta fase muestra estado; la generación del XML es la Épica 9, bloqueada AS-2)

**Given** un reloj cuyo plazo de 24h se agota antes de presentar
**When** vence
**Then** se marca "Vencido" y se registra como tal en el expediente (no se oculta el incumplimiento)

**Given** toda la cadena Hangfire + clock + detección
**When** corre en pruebas
**Then** un monitoreo dedicado de la cadena del reloj verifica el disparo y conteo deterministas (mitiga R4)

### Story 3.5: Configuración de cadencia y alcance del rescreening con piso legal

As a sujeto obligado,
I want configurar la frecuencia y el alcance del rescreening (todo el padrón o solo activos) respetando un piso legal mínimo,
So that lo ajuste a mi operación sin quedar por debajo de lo que la ley exige.

**Acceptance Criteria:**

**Given** la configuración de rescreening
**When** el sujeto obligado fija cadencia y alcance
**Then** se aplica al job recurrente, y el alcance puede ser todo el padrón o solo los clientes activos

**Given** el piso legal mínimo de cadencia (ADR-0005)
**When** el usuario intenta configurar una cadencia por debajo del piso
**Then** la UI muestra el piso mínimo y no permite quedar por debajo de él

### Story 3.6: Historial de rescreenings

As a sujeto obligado,
I want un historial de los rescreenings ejecutados,
So that demuestre monitoreo continuo en una auditoría.

**Acceptance Criteria:**

**Given** los eventos `RescreeningEjecutado` registrados
**When** consulto el historial
**Then** veo cada rescreening con su fecha, alcance y las versiones de listas usadas

**Given** un requerimiento de auditoría
**When** se exporta o consulta el historial
**Then** evidencia la continuidad del monitoreo en el periodo solicitado

### Story 3.7: Vistas de auditor — continuidad de monitoreo y cadena de decisiones

As a auditor del Art. 12 Bis,
I want ver qué listas y versiones se usaron y cuándo, la cadena de decisiones (automática + ajustes manuales) y que el rescreening fue continuo y los avisos se dispararon,
So that verifique que el screening fue real, oportuno y bien fundamentado.

**Acceptance Criteria:**

**Given** un expediente
**When** el auditor lo consulta
**Then** muestra qué listas y versiones se usaron y cuándo (realiza US-59, sobre los datos sellados en la Épica 1)

**Given** un dictamen con su historia
**When** el auditor revisa la cadena de decisiones
**Then** ve la decisión automática y cada ajuste manual con su justificación (realiza US-60)

**Given** el historial de rescreenings y los relojes de aviso
**When** el auditor los revisa
**Then** confirma que el rescreening fue continuo y que los relojes de aviso 24h se dispararon en los eventos correspondientes (realiza US-61)

---

## Epic 4: Canal API / white-label y gateway multi-cuenta

El motor se entrega por su canal comercial: gateway con auth por API keys self-serve, rate limits, contrato versionado, webhooks confiables y aislamiento estricto, habilitando al integrador/ERP y al despacho multi-cliente, con metering y observabilidad de costo. Todos los canales consumen el mismo `/v1/` de la Épica 1 (US-51). Construye sobre Épicas 1–3.

### Story 4.1: API keys self-serve, rate limits y catálogo de errores

As a integrador,
I want generar y administrar mis API keys yo mismo, con rate limits claros y errores documentados,
So that arranque mi integración sin esperar provisión manual y la construya con confianza.

**Acceptance Criteria:**

**Given** una Cuenta
**When** el integrador genera una API key
**Then** la key se muestra una sola vez, se guarda hasheada en reposo, nunca se loguea, y tiene scope, rotación y metering por key (AR-22)

**Given** una API key
**When** se hacen llamadas a `/v1/`
**Then** se aplican rate limits con el limitador nativo de ASP.NET Core, y al excederlos se responde con ProblemDetails (RFC 7807) del catálogo versionado (AR-19, AR-22)

**Given** cualquier error de la API
**When** se devuelve al cliente
**Then** usa ProblemDetails con `type`, `title`, `status`, `detail`, `traceId` del catálogo versionado en `/v1/`, sin exponer stack traces (AR-19)

### Story 4.2: Endpoint síncrono de screening de un nombre para integradores

As a integrador,
I want llamar a un endpoint síncrono de screening de un solo nombre y recibir un dictamen LFPIORPI terminado en una sola llamada,
So that resuelva el onboarding en tiempo real desde el ERP sin integrar 5 fuentes ni construir el dictamen yo.

**Acceptance Criteria:**

**Given** una API key válida
**When** el integrador llama al endpoint síncrono de un nombre en `/v1/`
**Then** recibe el dictamen completo (69-B/LPB/RFC + resolución de homonimia) con el mismo esquema que usa el portal (US-42, US-45, US-51)

**Given** la misma Core API
**When** el portal (Épica 2) y la API/white-label invocan el screening
**Then** ambos consumen el mismo contrato `/v1/`, sin reescribir la lógica de decisión por canal (US-51)

**Given** una solicitud síncrona vía API
**When** se mide la latencia
**Then** cumple NFR-1 (p95 < 3 s) extremo a extremo

### Story 4.3: Webhooks de entrega confiable firmados HMAC (Outbox)

As a integrador,
I want suscribirme a webhooks y recibir notificaciones firmadas de forma confiable,
So that me entere cuando un cliente ya cargado cae en una lista, sin perder eventos.

**Acceptance Criteria:**

**Given** un evento entregable (p. ej. rescreening con `CoincidenciaDetectada`)
**When** se notifica al integrador
**Then** la entrega usa el patrón Outbox sobre el event store, con payload firmado HMAC y reintentos idempotentes (at-least-once + idempotencia por id de evento) (US-49, AR-20)

**Given** un retraso en la entrega del Outbox
**When** el lag supera el umbral
**Then** se dispara una alerta de lag del Outbox (AR-20)

**Given** un webhook entregado
**When** el receptor verifica la firma
**Then** puede validar el HMAC con su secreto compartido

### Story 4.4: Screening batch asíncrono con job id y resultado por webhook o poll

As a integrador,
I want enviar un padrón completo a un endpoint batch asíncrono y recibir un job id, y obtener el resultado por webhook o poll,
So that procese volúmenes sin bloquear mi integración.

**Acceptance Criteria:**

**Given** un padrón enviado al endpoint batch async de `/v1/`
**When** se acepta
**Then** se devuelve un `job id` y el procesamiento corre como job de Hangfire (US-43)

**Given** un job batch en curso o terminado
**When** el integrador consulta por poll o espera el webhook
**Then** obtiene el estado y, al terminar, los dictámenes con el mismo esquema (US-44, vía Outbox de la historia 4.3)

**Given** un padrón de referencia de 10,000 sujetos
**When** se procesa en batch
**Then** termina en < 30 min (NFR-2)

**Given** un padrón que excede el tamaño máximo por corrida
**When** se envía
**Then** se rechaza con ProblemDetails (`413`/`422`), con el límite duro documentado (NFR-2)

### Story 4.5: Contrato `/v1/` versionado con changelog y política de deprecación

As a integrador,
I want un esquema de dictamen versionado bajo `/v1/` con changelog y política de deprecación,
So that mi integración no se rompa cuando el contrato evolucione.

**Acceptance Criteria:**

**Given** el contrato público
**When** se publica
**Then** vive bajo `/v1/`, con documentación OpenAPI (nativo .NET 10) y un changelog del esquema del dictamen (US-48, AR-17)

**Given** un cambio futuro del contrato
**When** se introduce
**Then** sigue una política de deprecación documentada, y el esquema del dictamen se considera congelado dentro de `/v1/`

### Story 4.6: Despacho multi-cliente con separación estricta y resumen consolidado

As a despacho PLD,
I want administrar el screening de varios sujetos obligados desde una sola cuenta, con sus padrones y expedientes separados y un resumen consolidado,
So that lleve su PLD a escala y priorice mi trabajo del día sin mezclar información.

**Acceptance Criteria:**

**Given** una Cuenta-despacho con varios sujetos obligados
**When** el despacho opera
**Then** administra el screening de todos desde una sola cuenta (US-37), con padrones y expedientes separados por cliente final (US-38, sobre el aislamiento de la Épica 1)

**Given** la vista "Todos"
**When** el despacho la abre
**Then** ve un resumen consolidado de hits y pendientes de toda la cartera (US-41)

**Given** dos sujetos obligados de la misma Cuenta-despacho
**When** se accede por cualquier canal (portal o API)
**Then** ninguno puede leer el padrón/expediente/LPB del otro (verificado por el IDOR sweep de la Épica 1) (AR-7)

### Story 4.7: White-label — reventa y UI incrustable del dictamen

As a despacho PLD / integrador white-label,
I want revender el dictamen con mi propia marca e incrustar la UI del dictamen bajo mi marca,
So that lo ofrezca como mi servicio o dentro de mi producto sin construir pantallas.

**Acceptance Criteria:**

**Given** un despacho que revende
**When** entrega el dictamen a su cliente final
**Then** puede presentarlo con su propia marca (US-39)

**Given** un integrador white-label
**When** incrusta la UI del dictamen
**Then** la muestra bajo su marca dentro de su producto, consumiendo el contrato `/v1/` (US-50)

**Nota:** la reventa de datos de OpenSanctions a clientes finales requiere licencia Reseller/OEM (**AS-1**) — bloquea la *comercialización* del white-label, no el build de esta historia.

### Story 4.8: Metering por Cuenta y observabilidad de costo por corrida

As a operador del producto,
I want medir el consumo por Cuenta y ver el costo por corrida desglosado (OpenSanctions vs. IA),
So that cobre por uso, recupere el costo de OpenSanctions y vigile el margen en tiempo real.

**Acceptance Criteria:**

**Given** las llamadas de screening por Cuenta
**When** se procesan
**Then** se registra el metering por Cuenta para cobro por uso (US-52)

**Given** una corrida
**When** termina
**Then** se registra su costo desglosado (OpenSanctions vs. IA en Bedrock) por Cuenta, observable para vigilar SM-4 (US-58, NFR-6)

**Given** los logs estructurados
**When** se emiten
**Then** llevan `traceId` y `sujeto_obligado_id` en scope y nunca incluyen PII sensible, LPB ni secretos (AR-26)

### Story 4.9: Alternar OpenSanctions entre API y self-hosted tras el puerto

As a operador del producto,
I want alternar entre OpenSanctions por API y self-hosted sin tocar el núcleo,
So that optimice el costo según el volumen.

**Acceptance Criteria:**

**Given** el puerto `IOpenSanctionsPort` (Épica 1)
**When** se cambia de adaptador API (`/match`, €0.10/consulta) a self-hosted (`yente`)
**Then** el cambio se hace por configuración sin modificar el núcleo determinístico (US-56, AR-24)

**Given** cualquiera de los dos modos
**When** se emite un dictamen
**Then** el expediente sella la versión de dataset correspondiente (consistente con el sello de la Épica 1)

### Story 4.10: Seguridad de canal — secretos en bóveda, cifrado y aislamiento verificable

As a operador del producto,
I want que todas las credenciales vivan en una bóveda y los datos estén cifrados y aislados por sujeto obligado de forma verificable,
So that ningún canal exponga secretos y el aislamiento de datos resista due-diligence (NFR-5).

**Acceptance Criteria:**

**Given** las credenciales (Bedrock, Core API, OpenSanctions, claves por sujeto obligado)
**When** se inyectan en los canales
**Then** provienen de AWS Secrets Manager + IAM role, con cero credenciales en contenedor o logs (US-57, AR-9)

**Given** los datos en reposo y en tránsito
**When** se almacenan o transmiten
**Then** están cifrados (RDS + S3 SSE; LPB cifrada por sujeto obligado, no persistida más de lo necesario) (NFR-5, AR-9)

**Given** el aislamiento por sujeto obligado
**When** se ejecuta el test adversarial de fuga cruzada (IDOR) sobre todos los canales
**Then** queda verificable que ningún canal expone datos de otro sujeto obligado (NFR-5, AR-7)
