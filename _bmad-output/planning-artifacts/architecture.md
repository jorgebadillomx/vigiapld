---
stepsCompleted: [1, 2, 3, 4, 5, 6, 7, 8]
lastStep: 8
status: 'complete'
completedAt: '2026-06-24'
inputDocuments:
  - docs/prd_screening_pld.md
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
  - docs/brief_prelavadodinero.md
  - docs/plan_distribucion.md
  - docs/.decision-log.md
workflowType: 'architecture'
project_name: 'PreLavadoDinero'
user_name: 'Jorge'
date: '2026-06-24'
ownerConstraints:
  - 'Base de datos: PostgreSQL'
  - 'Almacenamiento de archivos: almacén de objetos seguro y de bajo costo tras un puerto (cifrado en reposo, acceso por sujeto obligado, inmutabilidad/retención 10 años)'
  - 'Multiplataforma: compila y corre en Windows y Linux; producción Linux; stack .NET/ASP.NET Core; sin dependencias solo-Windows; contenedores Linux'
---

# Architecture Decision Document — Motor de Screening PLD/FT (PreLavadoDinero)

_This document builds collaboratively through step-by-step discovery. Sections are appended as we work through each architectural decision together._

## Project Context Analysis

### Requirements Overview

**Functional Requirements:**
76 User Stories (PRD) agrupadas por actor → mapean a componentes arquitectónicos:
- Ingesta de padrón (Excel/CSV: mapeo de columnas, validación, dedup) — US-1..6
- Núcleo de dictamen y homonimia (banda de confianza, matriz banda×tipo_lista) — US-7..14
- Evidencia/conservación 10 años (expediente inmutable, export PDF/Excel) — US-15..19
- Rescreening, alertas y reloj de aviso 24h — US-20..24
- Avisos XML SPPLD [bloqueado AS-2] — US-25..29
- Beneficiario Controlador + partes relacionadas (declarado, no recursivo) — US-30..33, 67..69
- EBR 5 dimensiones — US-34..36
- Despacho multi-cliente / Integrador-ERP / Operador / Auditor (canales y gobierno) — US-37..61
- Monitoreo transaccional [bloqueado AS-4] — US-62..66
- Programa PLD y validación de identidad (por integración) — US-70..76

**Non-Functional Requirements:**
- NFR-1 Latencia síncrona p95 < 3 s · NFR-2 Batch (10k en <30 min, tamaño máx. acotado) ·
  NFR-3 Disponibilidad 99.5% con degradación LLM→plantilla · NFR-4 Conservación/inmutabilidad
  10 años (RPO≤24h, RTO≤8h) · NFR-5 Seguridad (bóveda, cifrado, aislamiento por sujeto obligado) ·
  NFR-6 Observabilidad de costo por corrida (OpenSanctions vs IA).

**Scale & Complexity:**
- Primary domain: backend/API (engine-first) + portal web React
- Complexity level: alta (regulatorio + multi-tenant + event-sourced + ~10 módulos)
- Estimated architectural components: ~12 (gateway, core determinístico, orquestación LLM,
  ingesta, persistencia event-sourced, proyecciones, jobs Hangfire, módulos suite,
  almacén de objetos, portal, contrato API, observabilidad/metering)

### Technical Constraints & Dependencies

- Stack: .NET / ASP.NET Core + EF Core; **PostgreSQL**; Hangfire; React/Vite/shadcn (portal).
- **Multiplataforma Windows+Linux, producción Linux**; contenedores Linux; sin APIs solo-Windows.
- **Almacén de objetos** seguro/barato tras puerto (cifrado, acceso por sujeto obligado, retención 10a).
- Dependencias externas tras puerto: OpenSanctions (/match, API↔self-hosted), Claude Sonnet 4.6 en
  AWS Bedrock (ADR-0007), fuente SAT 69-B (hospedada+refrescada), proveedor de identidad (a definir),
  esquema XML SAT (a confirmar, ADR-0010). LPB la sube el cliente, nunca se hospeda.
- Modos de invocación: single-name síncrono + batch asíncrono; mismo esquema de dictamen.

### Cross-Cutting Concerns Identified

- Aislamiento multi-tenant por **Cuenta** (metering) y **sujeto obligado** (datos/LPB) — ADR-0003.
- Auditabilidad/inmutabilidad event-sourced + sellado de versiones (listas, política de corte) — ADR-0002/0001.
- Abstracción de tiempo (clock inyectable) para el reloj de aviso 24h y rescreening deterministas.
- Puertos para toda dependencia externa (I/O, costo, red, fakeo en pruebas).
- Versionado del contrato público /v1/ (changelog, deprecación).
- Seguridad/secretos (bóveda), cifrado en reposo/tránsito.
- Protección de datos personales (LFPDPPP/ARCO) vs. retención 10 años.
- Detección de staleness de listas; observabilidad de costo por tenant/corrida.

### Architectural Risks & Tensions (pre-stack)

_(Derivado de elicitación avanzada: Assumption Audit, Pre-mortem, Cascading Failure, ADR personas, Security Audit.)_

**Tensiones a resolver en las decisiones de stack:**
- **Event store vs EF Core.** El rastro es event-sourced (ADR-0002); EF Core es ORM, no event
  store. Decidir: tabla de eventos + proyecciones a mano vs. Marten (event store sobre Postgres,
  una sola dependencia). Recomendación de la elicitación: híbrido — Marten para eventos, EF Core
  para CRUD/catálogos/metering.
- **Despliegue.** Decidir explícitamente **monolito modular** (un despliegue, fronteras por puerto)
  vs. servicios. Recomendación: monolito modular; puertos como costuras de extracción futura.
- **Degradación ante caída de OpenSanctions (SPOF).** A diferencia del LLM (cae a plantilla),
  OpenSanctions es SPOF del screening de sanciones/PEP. Decidir degradación parcial (emitir con
  tipos de lista locales + marca "sanciones pendientes") y/o el modo self-hosted como mitigación.

**Riesgos arquitectónicos priorizados (de pre-mortem + cascading):**
- R1 [alto] Replay de proyecciones impagable a escala → snapshots + versión de esquema de
  proyección desde el día 1.
- R2 [alto] Cambio silencioso del scoring de OpenSanctions rompe reproducibilidad → sellar la
  **versión del matcher**, no solo la de la lista, en cada expediente.
- R3 [alto] Inmutabilidad legal depende de object-lock/WORM → verificar por test, no por confianza;
  el evento de dictamen persiste en DB aunque el binario del expediente falle (desacoplar).
- R4 [alto] Reloj de aviso 24h: cadena Hangfire + clock + detección → clock inyectable en UTC con
  zona MX explícita, tests de DST, monitoreo dedicado de la cadena.
- R5 [medio] Jobs Hangfire atorados → listas stale silenciosas → alerta de jobs + staleness (ya en PRD).
- R6 [medio] Margen invisible → observabilidad de costo por corrida como NFR de primer nivel.

**Concerns de seguridad elevados:**
- Aislamiento por sujeto obligado a nivel de query (RLS Postgres o filtro obligatorio en repositorio),
  con test adversarial de fuga cruzada dentro de una misma Cuenta-despacho.
- Cifrado de secretos en bóveda y de la LPB confidencial por sujeto obligado; LPB no persistida
  más allá de lo necesario.
- Protección de datos (LFPDPPP/ARCO) vs. retención 10 años como concern de primer nivel (base de
  licitud documentada; rectificación como evento nuevo).

## Starter Template Evaluation

### Primary Technology Domain

API/backend (.NET 10 LTS, engine-first) + portal web React desacoplado. Dos starters, uno por capa.
Versiones verificadas en web (jun-2026): .NET 10 es el LTS vigente (soporte hasta nov-2028); .NET 9/8
expiran nov-2026.

### Starter Options Considered (matriz comparativa de backend)

| Criterio (peso) | Jason Taylor CA | Ardalis CA | Custom (minimal-API + vertical slice) |
|---|---|---|---|
| Fit con event-sourcing/Marten (×5) | 3 | 3.5 | 5 |
| Sin licencias comerciales (×4) | 2 (MediatR) | 3 | 5 (FastEndpoints + Mediator MIT) |
| Ligereza para operador solo (×3) | 3 | 4 | 4 |
| PostgreSQL nativo (×3) | 5 (`--database PostgreSQL`) | 3 (SQLite default) | 5 |
| .NET 10 / multiplataforma (×2) | 5 | 5 | 5 |
| Capas/puertos/testing de fábrica (×4) | 5 | 4 | 3 |
| Velocidad de arranque (×2) | 5 | 4 | 2 |
| **Total ponderado / 115** | **87 (76%)** | **84.5 (73%)** | **98 (85%)** |

El custom puntúa más alto por fit de event-sourcing y cero licencias, pero a costa de velocidad y de
construir las convenciones a mano. El desempate real son **MediatR** (hoy licencia comercial, gratis
bajo umbral de ingresos) y el **event store**. Se resuelve conservando la plantilla con baterías y
neutralizando esos dos puntos, no eligiendo un extremo.

**Frontend:** Vite + React-TS standalone (coincide con el stack fijado en el PRD) vs. cliente React
empaquetado de la plantilla CA → se elige **standalone** para controlar shadcn/Tailwind v4/TanStack
Query/React Router y consumir el contrato `/v1/` como cascarón delgado.

### Selected Starter

**Backend — Jason Taylor Clean Architecture, con dos ajustes desde el día 1:**
```bash
dotnet new install Clean.Architecture.Solution.Template
dotnet new ca-sln --client-framework None --database PostgreSQL --output src/CoreApi
```
- **Ajuste 1 — sin MediatR:** sustituir por el **`Mediator` source-generator (Martin Othamar, MIT)**
  o handlers propios → elimina el riesgo de licencia comercial.
- **Ajuste 2 — event store:** montar **Marten** (OSS sobre Postgres) en Infrastructure para el rastro
  event-sourced (ADR-0002); EF Core queda para CRUD/catálogos/metering. Una sola dependencia de datos.
- *Alternativa ligera:* **Ardalis CA** con los mismos dos ajustes, si se prefiere arrancar más delgado.

**Frontend — Vite React-TS + Tailwind v4 + shadcn (desacoplado):**
```bash
npm create vite@latest portal -- --template react-ts
cd portal && npm install
npm install tailwindcss @tailwindcss/vite
npx shadcn@latest init
# luego: TanStack Query, React Router
```

**Rationale:** Clean Architecture aporta las capas y costuras (puertos) que el diseño determinístico-
tras-puerto y los tests de contrato del PRD ya asumen; PostgreSQL nativo cumple la restricción del
propietario; .NET 10 LTS + contenedores Linux cumplen multiplataforma/producción Linux. Los dos
ajustes cierran el fit de event-sourcing y la exposición a licencia comercial. El frontend desacoplado
respeta el stack exacto del PRD y el principio "canal = cascarón delgado sobre el contrato".

**Architectural Decisions Provided by Starter:** layering DDD (Domain/Application/Infrastructure/Web),
pipeline de validación, inyección de dependencias, proyectos de prueba unitarios/integración, EF Core
(CRUD), PostgreSQL. (El event store con Marten y los puertos a dependencias externas se detallan en el
paso de decisiones de arquitectura.)

**Note:** La inicialización con estos comandos (backend + frontend, con los dos ajustes) debe ser la
primera historia de implementación.

## Core Architectural Decisions

### Decision Priority Analysis

**Critical (bloquean implementación):** event store (Marten) + EF Core; aislamiento multi-tenant
(RLS + filtro de repositorio); hosting AWS (S3 Object Lock para WORM, RDS Postgres, Secrets Manager);
reemplazo de MediatR (Mediator MIT); puertos a dependencias externas.
**Important (moldean arquitectura):** RBAC con segregación de funciones; Outbox para webhooks/avisos;
observabilidad + costo por corrida; ProblemDetails; rate limiting.
**Deferred (post-fase-1 o bloqueadas):** detección de inusualidad con IA (fase 2); módulos de
monitoreo (AS-4) y avisos XML (AS-2) bloqueados por datos externos; self-hosting de OpenSanctions
(jugada de escala); Fargate, async daemon de Marten y OpenTelemetry completo (cuando el volumen lo pida).

### Data Architecture

- **PostgreSQL en AWS RDS** (Postgres 12+; Marten requiere 12+).
- **Event store: Marten 8.x** (OSS, sobre Postgres) para el rastro append-only (ADR-0002); proyecciones
  **inline al inicio**, async daemon + snapshots cuando el volumen lo demande (mitiga R1). EF Core para
  CRUD/catálogos/metering.
- **Aislamiento multi-tenant:** Row-Level Security de Postgres **+** filtro obligatorio en repositorio
  (defensa en profundidad) por sujeto obligado; test adversarial de fuga cruzada (IDOR sweep).
- **Sello de reproducibilidad:** cada expediente sella versión de lista, de política de corte y
  **del matcher de OpenSanctions** (mitiga R2).
- Migraciones: EF Core migrations (CRUD) + gestión de esquema de Marten.

### Authentication & Security

- **Portal:** ASP.NET Core Identity (self-hosted, gratis).
- **API/integradores:** API keys self-serve (hasheadas en reposo, mostradas una vez, nunca logueadas,
  con scope/rotación/metering por key) + rate limiter nativo de ASP.NET Core; webhooks firmados (HMAC).
- **AuthZ:** RBAC con segregación screener / aprobador / auditor (US-72).
- **Secretos:** AWS Secrets Manager + IAM role (Bedrock, OpenSanctions, Core API, claves por tenant);
  cero credenciales en contenedor/logs.
- **Cifrado:** en reposo (RDS + S3 SSE) y en tránsito; LPB confidencial cifrada por sujeto obligado,
  no persistida más de lo necesario.
- **WORM/inmutabilidad:** S3 Object Lock (modo compliance, retención 10a) para expedientes; verificado
  por test set-at-creation (mitiga R3).
- **Aislamiento por principal:** toda query filtrada por `sujeto_obligado_id` del **principal
  autenticado, nunca del input del cliente**; RLS forzado por middleware de conexión.
- **Protección de datos (LFPDPPP/ARCO):** base de licitud PLD documentada; rectificación como evento
  nuevo; supresión diferida al vencer el plazo de 10 años.

### API & Communication Patterns

- **REST `/v1/`** versionado (changelog + deprecación); sync (single-name) + batch async (Hangfire
  job → webhook/poll).
- **Errores:** ProblemDetails (RFC 7807) + catálogo de errores versionado.
- **Webhooks:** patrón **Outbox** sobre el event store (entrega confiable de rescreening/avisos) +
  alerta de lag del Outbox.
- **Docs:** OpenAPI nativo .NET 10.
- **Puertos** para OpenSanctions, Bedrock, fuente SAT, identidad, almacén de objetos (I/O, costo,
  fakeo en pruebas). Degradación: caída de Bedrock → plantilla; caída de OpenSanctions → dictamen con
  tipos de lista locales + marca "sanciones pendientes".

### Frontend Architecture

- Vite + React + TS + shadcn/ui + Tailwind v4; **TanStack Query** (estado servidor) + estado local
  mínimo; React Router; cascarón delgado que consume `/v1/`.

### Infrastructure & Deployment

- **AWS, huella "barata" al inicio (escala cuando el volumen lo pida):** cómputo en **una EC2/Lightsail
  + contenedores** (subir a ECS Fargate al escalar) · **RDS PostgreSQL mínimo** (o Postgres on-box al
  inicio) · **S3 Object Lock** + **Secrets Manager** desde el día 1 (lo legalmente crítico es gestionado)
  · región con Bedrock + residencia de datos deseada.
- **Contenedores Linux** como artefacto; multiplataforma Win+Linux en desarrollo.
- **CI/CD:** GitHub Actions (build + test + imagen) — al inicializar el repo.
- **Observabilidad:** **logs estructurados + costo por corrida primero** (OpenSanctions vs IA, NFR-6);
  OpenTelemetry cuando la topología lo justifique; alerta de staleness de listas; monitoreo dedicado de
  la cadena del reloj de aviso 24h (R4).

### Decision Impact Analysis

**Implementation Sequence:** (1) init solución CA + Postgres/Marten + puertos → (2) núcleo
determinístico + contrato `/v1/` + expediente → (3) AWS/S3/Secrets + identidad/auth → (4) jobs +
rescreening + reloj aviso → (5) canales → (6) módulos de suite.

**Cross-Component Dependencies:** Marten habilita Outbox y proyecciones (expediente/constancia); el
clock inyectable (UTC + zona MX) habilita reloj de aviso y rescreening deterministas; los puertos
habilitan el fakeo en todos los seams de prueba; el aislamiento por sujeto obligado condiciona cada
repositorio.

### Right-sizing y endurecimiento (refinación de elicitación)

_(De Occam's Razor + Failure Mode + Red Team/Blue Team + Cross-Functional War Room.)_

**Arranque "barato dentro de AWS":** EC2/Lightsail + contenedores (no Fargate al inicio); RDS mínimo o
Postgres on-box; S3 Object Lock + Secrets Manager desde el día 1; proyecciones Marten inline primero;
logs estructurados + metering de costo antes que OpenTelemetry. Lo legalmente crítico (Marten event
store, RLS, Outbox, WORM) se mantiene; lo operacionalmente caro se difiere hasta que el volumen lo pida.
Justificación de costo: full-AWS con Fargate+NAT ≈ $150–300+/mes baseline vs. EC2 chica + RDS mínimo
≈ $40–70/mes, relevante contra el ARPU ~$70 USD/mes y margen 55–60%.

**Endurecimiento de seguridad:** filtrado por `sujeto_obligado_id` del principal autenticado (IDOR
sweep en pruebas); API keys hasheadas/mostradas-una-vez; webhooks HMAC; LPB cifrada por sujeto obligado;
S3 Object Lock verificado por test; alerta de lag del Outbox + monitoreo del reloj 24h (R4).

### Variante de despliegue: lean / VPS compartido (opción de costo)

_(Decisión del propietario, jun-2026: existe un VPS compartido entre varios proyectos, así que el costo
de cómputo se amortiza. Esta variante **coexiste** con el escenario "todo-AWS"; no lo reemplaza como
referencia de cumplimiento. La frontera no-negociable es la misma en ambos: el expediente WORM y los
backups viven en almacenamiento gestionado con Object Lock, nunca en disco de VPS.)_

**Tres escenarios de hosting, con su trade-off:**

| Escenario | Fijo USD/mes (arranque) | Pros | Contras |
|---|---|---|---|
| **A · Todo-AWS bien hecho** | ~$43–60 | máxima narrativa de cumplimiento (ISO/auditoría); ops gestionadas (RDS, backups) | más caro; lock-in AWS para cómputo/DB |
| **B · VPS dedicado a este proyecto** | ~$22–28 | barato; control | ops tuyas (parches, backups, uptime); ahorro chico vs A no compensa el riesgo |
| **C · VPS compartido entre proyectos** ✅ | **~$5–10** (marginal) | cómputo amortizado entre proyectos → costo marginal casi nulo; mantiene la frontera WORM en AWS | aislamiento del workload regulado en caja compartida; residencia (Hetzner EE.UU./EU, no MX) |

**Escenario C — cómo se compone (elegido por el propietario):**
- **Cómputo + Postgres/Marten:** contenedores de PreLavadoDinero en el VPS compartido, gestionados con un
  PaaS self-hosted (Coolify/Dokploy: deploy git-push, TLS Let's Encrypt, variables cifradas). **DB Postgres
  con credenciales aisladas** de los demás proyectos (no compartir instancia ni usuario).
- **Expediente WORM 10 años → S3 Object Lock (modo compliance), idealmente en `mx-central-1`** para tener
  la evidencia legal en territorio MX. **Innegociable; no va en disco de VPS** (R3/NFR-4 — es el producto).
- **Backups DB → off-box** (pgBackRest → S3/B2); un VPS es punto único de falla (RTO≤8h/RPO≤24h, NFR-4).
- **Bedrock → API externa** (IAM desde el VPS); **OpenSanctions → API `/match`** (ver decisión abajo).
- **Secretos:** Coolify cifrado o Infisical self-host; nunca en repo/logs (igual que escenario A).

**Cuidados del escenario C (producto regulado en caja compartida):**
1. **Aislamiento.** Maneja LPB confidencial: aísla DB y secretos; no correr experimentos en la misma caja
   que procesa datos financieros confidenciales. Opción intermedia si incomoda: VPS dedicado a lo regulado
   + VPS compartido para lo demás.
2. **Residencia (LFPDPPP).** El cómputo en EE.UU./EU con datos cifrados es defendible (Bedrock tampoco está
   en MX, ver corrección ADR-0007); pero el **expediente inmutable (S3) sí en `mx-central-1`**.

**Stack lean resultante:**
> VPS compartido (Hetzner + Coolify) para app + Postgres · **S3 Object Lock en `mx-central-1`** para
> expedientes · Bedrock por uso · **OpenSanctions por API `/match` (€0.10/call)** hasta que el volumen
> justifique self-host (US-56) · backups off-box a S3/B2.

### Decisión: OpenSanctions API-first (diferir self-host)

- **Arranque por la API hospedada (`/match`, €0.10/llamada exitosa, descuento >20k/mes).** Da **toda la
  funcionalidad de matching** (mismo motor `yente`, mismos datasets sanciones/PEP). Cero adelanto, cero
  compromiso — coherente con la incertidumbre de volumen ("no sabemos si habrá muchos clientes").
- **Self-host (`yente` + licencia bulk anual) se difiere** hasta que `€0.10 × volumen > licencia anual`.
  El puerto `IOpenSanctionsPort` (US-56) permite el cambio sin tocar el núcleo.
- **R2 en modo API:** como la API no permite pinear la versión del matcher, **sellar en cada expediente la
  versión de dataset que la API devuelve** en la respuesta (mitigación equivalente al pineo de self-host).
- **LFPDPPP:** en modo API el nombre+RFC+fecha del sujeto evaluado viaja a OpenSanctions (EU); base de
  licitud AML documentada. Self-host lo elimina si la residencia se vuelve requisito.
- **Bloqueador comercial AS-1 (no de build):** el producto **distribuye datos de OpenSanctions a sus
  clientes** → requiere **licencia Reseller/OEM** (la más cara), y aplica **aunque se use la API**.
  Confirmar términos y costo con ventas de OpenSanctions **antes de comercializar el canal white-label**.

## Implementation Patterns & Consistency Rules

### Pattern Categories Defined

~6 áreas de conflicto entre agentes resueltas: naming, estructura, formato API, eventos, errores, logging.

### Naming Patterns

**Lenguaje ubicuo (autoridad: CONTEXT.md):** los nombres de dominio van en **español** según el glosario
— `SujetoObligado`, `Padron`, `SujetoEvaluado`, `Dictamen`, `BandaConfianza`, `TipoLista`,
`PoliticaDeCorte`, `Expediente`, `ConstanciaDeCorrida`, `RelojDeAviso`, `Operacion`, `ClasificacionEBR`.
Términos proscritos del glosario (`indeterminado`, `tenant`) **no aparecen en código**. La plomería
(genéricos, infra) va en inglés.

**Database (PostgreSQL):** `snake_case` para tablas y columnas; tablas en plural (`sujetos_obligados`,
`dictamenes`); FK `<entidad>_id` (`sujeto_obligado_id`); índices `ix_<tabla>_<columnas>`. EF Core
configurado con convención `snake_case`. **Marten gestiona sus propias tablas de eventos** (no tocar).

**API REST:** rutas `/v1/<recurso-plural-kebab>` (`/v1/sujetos-evaluados`); parámetros `{id}`;
JSON en **camelCase**; **valores de enum = tokens canónicos del glosario en kebab**
(`resuelto-real`, `resuelto-homonimo`, `descartado-por-nombre`, `requiere-verificacion`;
`sanciones` / `bloqueados-uif` / `pep` / `efos-edos`); fechas **ISO 8601 UTC**; montos `decimal`.

**C#:** `PascalCase` (tipos, métodos, propiedades), `camelCase` (locales/parámetros), interfaces con
prefijo `I`; **puertos** nombrados `I<Dependencia>Port` (`IOpenSanctionsPort`, `ILlmRedactorPort`,
`IClock`, `IAlmacenObjetosPort`); métodos asíncronos con sufijo `Async`; un tipo por archivo.

**React/TS:** componentes `PascalCase` (archivo `PascalCase.tsx`); hooks `useX`; funciones/vars
`camelCase`; tipos del contrato generados/derivados del OpenAPI `/v1/`.

### Structure Patterns

- **Backend (Clean Architecture):** `Domain` (entidades, eventos, reglas — sin dependencias) ·
  `Application` (casos de uso, puertos como interfaces, handlers) · `Infrastructure` (Marten, EF Core,
  adaptadores de puertos, AWS) · `Web` (endpoints `/v1/`, auth, rate limit). El **núcleo determinístico
  vive en Domain/Application**; toda I/O externa detrás de un puerto en Application, implementado en
  Infrastructure.
- **Tests:** proyectos separados `*.UnitTests` (núcleo, sin I/O), `*.IntegrationTests` (contrato HTTP
  `/v1/` con puertos fakeados, DB real en contenedor). Convención del PRD: contrato HTTP como seam
  principal; tabla de casos para el núcleo.
- **Frontend:** organización **por feature** (`features/screening`, `features/expediente`); `components/ui`
  para shadcn; `lib/api` cliente del contrato; `hooks`.

### Format Patterns

- **Respuestas API:** recurso directo (sin wrapper `{data}`); colecciones con paginación
  `{ items, total, page, pageSize }`.
- **Errores:** **ProblemDetails (RFC 7807)** — `type`, `title`, `status`, `detail`, `traceId`; catálogo
  de `type` versionado en `/v1/`.
- **JSON:** camelCase; booleanos `true/false`; nulos omitidos donde aplique; enums como string-token del
  glosario (no enteros).

### Communication Patterns (eventos)

- **Eventos de dominio (Marten):** nombres en **pasado, español, según glosario** —
  `CoincidenciaDetectada`, `IntentoDeOperacion`, `UmbralAcumulado`, `DictamenEmitido`,
  `AjusteManualRegistrado`, `RescreeningEjecutado`. Payloads como `record` inmutables.
- **Versionado de eventos:** upcasting de Marten; nunca mutar un evento publicado.
- **Webhooks:** entrega vía **Outbox** sobre el event store; payload firmado HMAC; reintentos idempotentes.

### Process Patterns

- **Errores:** excepciones de dominio → ProblemDetails en el borde; nunca exponer stack traces;
  el dictamen siempre se emite (caída de LLM → `redaccion: plantilla`).
- **Tiempo:** **`IClock` inyectable**, UTC en almacenamiento, zona MX explícita para el reloj 24h;
  jamás `DateTime.Now` directo.
- **Logging:** estructurado, con `traceId` y `sujeto_obligado_id` en scope; **nunca** PII sensible,
  LPB ni secretos en logs.
- **Multi-tenant:** `sujeto_obligado_id` se toma del principal autenticado (nunca del input);
  cada repositorio filtra por él (+ RLS).

### Pattern Examples (✅ / ❌)

```csharp
// ✅ Evento de dominio: español, pasado, record inmutable, enums del glosario
public sealed record DictamenEmitido(Guid SujetoEvaluadoId, BandaConfianza Banda, TipoLista Tipo);
// ❌ inglés + término proscrito + mutable
public class ScreeningResult { public string Decision = "indeterminado"; }

// ✅ Puerto para dependencia externa
public interface IOpenSanctionsPort { Task<MatchResultado> MatchAsync(ConsultaMatch q, CancellationToken ct); }
// ❌ HttpClient a OpenSanctions inyectado directo en un handler (I/O sin puerto, no fakeable)

// ✅ Tiempo y tenant
var ahora = _clock.UtcNow;                                        // IClock inyectable
q.Where(x => x.SujetoObligadoId == _principal.SujetoObligadoId);  // del principal autenticado
// ❌ DateTime.Now;  y  sujetoObligadoId tomado del body del request
```

```jsonc
// ✅ enum como token del glosario, fecha ISO UTC
{ "bandaConfianza": "requiere-verificacion", "tipoLista": "pep", "fechaCorte": "2026-06-24T06:00:00Z" }
// ❌ { "decision": 3 }  o  { "decision": "indeterminado" }
```

```tsx
// ✅ DictamenCard.tsx + hook con TanStack Query
export function DictamenCard() { const { data } = useDictamen(id); }
// ❌ dictamen_card.tsx con fetch() crudo dentro del componente
```

### Edge & Boundary Conventions

- Paginación `{ items, total, page, pageSize }`; `pageSize` default 50, máx 200.
- Padrón vacío / single-name → resultado válido (posiblemente vacío), **nunca 500**.
- Webhooks **at-least-once** + idempotencia por id de evento; payload HMAC.
- Reintentos de puertos (OpenSanctions/Bedrock/S3): backoff exponencial + circuit breaker; al agotar →
  **degradar** (plantilla / "sanciones pendientes"), no fallar la corrida.
- Límite de tamaño de padrón (NFR-2) → `422`/`413` ProblemDetails, no corrida a medias.
- RFC/fecha faltantes → `requiere-verificacion` (no error).
- Score en frontera de banda → política de corte versionada, inclusividad documentada.
- Montos `decimal`, nunca `float`; moneda explícita (MXN/UMA).

### Enforcement Guidelines

**Todos los agentes DEBEN:** usar los términos del glosario `CONTEXT.md`; poner I/O externa tras puerto;
filtrar por `sujeto_obligado_id` del principal; usar `IClock`; emitir enums como tokens canónicos;
no introducir `indeterminado`/`tenant`. **Verificación:** analizadores/`.editorconfig` + revisión de
contrato `/v1/` + test de fuga cruzada (IDOR) + test de fidelidad prosa↔verdicto. **Convención de idioma:**
dominio en español (lenguaje ubicuo), plomería en inglés, mapping glosario↔código en el README.

## Project Structure & Boundaries

### Complete Project Directory Structure

```
PreLavadoDinero/                          # repo raíz (greenfield: git init pendiente)
├── README.md                          # incl. mapping glosario↔código (CONTEXT.md autoridad)
├── PreLavadoDinero.sln
├── Directory.Build.props              # versión .NET 10, analizadores, nullable
├── .editorconfig                      # convenciones (PascalCase/camelCase, etc.)
├── .gitignore
├── docker-compose.yml                 # dev: Postgres + app en contenedores Linux
├── .github/workflows/ci.yml           # build + test + imagen
├── docs/                              # PRD, CONTEXT.md, adr/ (ya existen)
├── deploy/
│   ├── Dockerfile                     # imagen Linux del CoreApi
│   └── aws/                           # IaC: EC2/Lightsail, RDS, S3 Object Lock, Secrets Manager
├── src/
│   ├── CoreApi/
│   │   ├── Domain/                    # entidades, eventos, reglas — SIN dependencias externas
│   │   │   ├── SujetosObligados/
│   │   │   ├── Padrones/              # SujetoEvaluado, TipoPersona
│   │   │   ├── Dictamenes/            # BandaConfianza, TipoLista, PoliticaDeCorte, matriz (banda×tipo)
│   │   │   ├── Expedientes/           # eventos: DictamenEmitido, AjusteManualRegistrado, RescreeningEjecutado
│   │   │   ├── Avisos/                # RelojDeAviso, eventos CoincidenciaDetectada/IntentoDeOperacion/UmbralAcumulado
│   │   │   ├── BeneficiarioControlador/  # + PartesRelacionadas
│   │   │   ├── Ebr/                   # ClasificacionEBR (5 dimensiones)
│   │   │   ├── Operaciones/           # monitoreo: ventanas 6 meses, umbrales
│   │   │   └── Common/                # ValueObjects (Rfc, Curp, Money), IClock (interfaz)
│   │   ├── Application/               # casos de uso + PUERTOS (interfaces) + handlers
│   │   │   ├── Screening/             # ingesta padrón, match, armado de dictamen
│   │   │   ├── Rescreening/
│   │   │   ├── Avisos/                # (bloqueado AS-2 para XML; reloj sí)
│   │   │   ├── Bc/  Ebr/  Monitoreo/  # (Monitoreo bloqueado AS-4 para umbrales)
│   │   │   ├── Identidad/  ProgramaPld/
│   │   │   └── Common/Ports/          # IOpenSanctionsPort, ILlmRedactorPort, ISatListaPort,
│   │   │                              #   IIdentidadPort, IAlmacenObjetosPort, IClock
│   │   ├── Infrastructure/            # adaptadores de puertos + datos + AWS
│   │   │   ├── Persistence/           # Marten (event store + proyecciones), EF Core DbContext, migrations, RLS
│   │   │   ├── OpenSanctions/  Bedrock/  Sat69B/  Identidad/   # adaptadores de puertos
│   │   │   ├── AlmacenObjetos/        # S3 (Object Lock)
│   │   │   ├── Jobs/                  # Hangfire (refresco SAT, rescreening, reloj, outbox relay)
│   │   │   └── Seguridad/             # Secrets Manager, cifrado, contexto RLS
│   │   └── Web/                       # cascarón HTTP
│   │       ├── Endpoints/v1/          # contrato público /v1/ (sync + batch async)
│   │       ├── Auth/                  # Identity (portal) + API keys (integradores) + rate limit
│   │       ├── Webhooks/              # firma HMAC, entrega desde Outbox
│   │       └── Program.cs             # composición, OpenAPI, ProblemDetails
│   └── Portal/                        # Vite + React + TS (cascarón delgado sobre /v1/)
│       ├── src/
│       │   ├── features/              # screening, expediente, rescreening, bc, ebr, avisos, configuracion
│       │   ├── components/ui/         # shadcn/ui
│       │   ├── lib/api/               # cliente del contrato /v1/ (tipos derivados de OpenAPI)
│       │   ├── hooks/                 # useDictamen, etc. (TanStack Query)
│       │   └── main.tsx
│       ├── package.json  vite.config.ts  tailwind.config  tsconfig.json
└── tests/
    ├── CoreApi.UnitTests/             # núcleo determinístico (tabla de casos: bandas, matriz, homonimia)
    ├── CoreApi.IntegrationTests/      # contrato HTTP /v1/, puertos fakeados, Postgres en contenedor; IDOR sweep
    └── Portal.Tests/                  # componentes (*.test.tsx)
```

### Architectural Boundaries

- **API:** `Web/Endpoints/v1/` es la única frontera pública; los tres canales (portal, API/white-label,
  Capafy fase 2) la consumen. Nada fuera de `Web` conoce HTTP.
- **Componentes:** `Domain` no depende de nadie; `Application` depende solo de `Domain` y define puertos;
  `Infrastructure` implementa puertos; `Web` compone. Dependencias apuntan hacia adentro (regla CA).
- **Servicios/externos:** toda I/O (OpenSanctions, Bedrock, SAT, identidad, S3) cruza un puerto en
  `Application/Common/Ports` con adaptador en `Infrastructure`. Fakeables en test.
- **Datos:** Marten posee el rastro event-sourced (no se escribe SQL crudo a sus tablas); EF Core posee
  CRUD/catálogos/metering; RLS + filtro por `sujeto_obligado_id` en todo repositorio.

### Requirements to Structure Mapping

| Módulo PRD (US) | Vive en |
|---|---|
| Ingesta padrón (US-1..6) | `Application/Screening` + `Domain/Padrones` |
| Dictamen + homonimia (US-7..14) | `Domain/Dictamenes` + `Application/Screening` |
| Evidencia 10 años (US-15..19) | `Domain/Expedientes` + `Infrastructure/Persistence` + `AlmacenObjetos` |
| Rescreening + reloj 24h (US-20..24) | `Application/Rescreening` + `Domain/Avisos` + `Infrastructure/Jobs` |
| Avisos XML (US-25..29) [AS-2] | `Application/Avisos` (XML bloqueado hasta ADR-0010) |
| BC + partes relacionadas (US-30..33,67..69) | `Domain/BeneficiarioControlador` |
| EBR (US-34..36) | `Domain/Ebr` |
| Canales/gobierno (US-37..61) | `Web/Endpoints/v1`, `Web/Auth`, `Portal/features` |
| Monitoreo (US-62..66) [AS-4] | `Domain/Operaciones` + `Application/Monitoreo` (umbrales bloqueados) |
| Programa PLD + identidad (US-70..76) | `Application/ProgramaPld`, `Application/Identidad` |

### Integration Points & Data Flow

- **Flujo de datos:** Web `/v1/` → handler en Application → núcleo determinístico (Domain) decide banda;
  puerto OpenSanctions para match; puerto LLM redacta (o plantilla); evento `DictamenEmitido` a Marten;
  proyección `Expediente`; binario PDF a S3; Outbox → webhook.
- **Externos:** OpenSanctions, Bedrock, SAT 69-B, identidad (proveedor a definir), S3, Secrets Manager.
- **Interno:** eventos de dominio vía Marten; jobs Hangfire para rescreening/refresco/reloj/outbox.

### Development & Deployment

- **Dev:** `docker-compose up` (Postgres + app); portal `npm run dev`; multiplataforma Win+Linux.
- **Build:** GitHub Actions → `dotnet test` + build imagen Linux + build portal.
- **Deploy:** imagen Linux a EC2/Lightsail; RDS Postgres; S3 Object Lock; Secrets Manager (huella barata,
  escala a Fargate después).

## Architecture Validation Results

### Coherence Validation ✅

- **Compatibilidad:** .NET 10 + EF Core + Marten 8.x (Postgres 12+) + Hangfire + AWS son compatibles;
  versiones verificadas. Mediator (MIT) reemplaza MediatR sin conflicto. Sin decisiones contradictorias.
- **Patrones↔decisiones:** snake_case/Postgres, eventos español/Marten, puertos/Clean Architecture,
  ProblemDetails/REST `/v1/` — todos alineados.
- **Estructura↔arquitectura:** el árbol Domain/Application/Infrastructure/Web realiza la regla de
  dependencias hacia adentro y aloja cada puerto/adaptador donde corresponde.

### Requirements Coverage Validation ✅

- **Cobertura FR:** las 10 categorías de US mapean a directorios concretos (tabla de mapeo).
- **NFR:** NFR-1 latencia, NFR-2 batch (Hangfire+límite), NFR-3 disponibilidad (degradación LLM→plantilla),
  NFR-4 retención/inmutabilidad (Marten+S3 Object Lock), NFR-5 seguridad (RLS+Secrets+cifrado), NFR-6 costo
  (metering) — todos con hogar arquitectónico.
- **Módulos diferidos por dato externo (no son gaps):** monitoreo (AS-4) y avisos XML (AS-2) están
  **arquitectónicamente soportados**; solo esperan el dato (umbrales/XSD) — diseño los acomoda en capa 4.

### Implementation Readiness Validation ✅

- **Decisiones:** completas y versionadas. **Patrones:** con ejemplos copiables ✅/❌ y convenciones de
  borde. **Estructura:** árbol completo, fronteras y puntos de integración definidos.

### Gap Analysis Results

- **Críticos (bloquean build del núcleo):** ninguno.
- **Importantes:** proveedor de identidad reforzada sin definir (AS-3, puerto listo); re-derivación de
  economía unitaria (AS-6, negocio no arquitectura); profundizar el modelo de datos Marten/EF por agregado
  en la primera épica.
- **Nice-to-have:** diagramas C4/secuencia por módulo; IaC detallada de AWS.

### Architecture Completeness Checklist

**Requirements Analysis**

- [x] Project context thoroughly analyzed
- [x] Scale and complexity assessed
- [x] Technical constraints identified
- [x] Cross-cutting concerns mapped

**Architectural Decisions**

- [x] Critical decisions documented with versions
- [x] Technology stack fully specified
- [x] Integration patterns defined
- [x] Performance considerations addressed

**Implementation Patterns**

- [x] Naming conventions established
- [x] Structure patterns defined
- [x] Communication patterns specified
- [x] Process patterns documented

**Project Structure**

- [x] Complete directory structure defined
- [x] Component boundaries established
- [x] Integration points mapped
- [x] Requirements to structure mapping complete

### Architecture Readiness Assessment

**Overall Status:** READY FOR IMPLEMENTATION (alcance: capas 1–3 del núcleo y canales; los módulos de
monitoreo y avisos XML quedan soportados pero diferidos hasta cerrar AS-4/AS-2 — por diseño, no por gap).
**Confidence Level:** high.
**Key Strengths:** núcleo determinístico aislado tras puertos; rastro event-sourced auditable; aislamiento
multi-tenant defendido en dos capas; degradación elegante (LLM→plantilla); huella AWS right-sized.
**Areas for Future Enhancement:** async daemon + snapshots de Marten al escalar; Fargate; OTel; diagramas
C4; cerrar AS-1..8 con especialista PLD/ventas.

### Implementation Handoff

**AI Agent Guidelines:** seguir las decisiones y patrones al pie; términos del glosario CONTEXT.md; I/O
tras puerto; filtrar por sujeto obligado; `IClock`; no `indeterminado`/`tenant`.
**First Implementation Priority:** inicializar la solución (Clean Architecture + Postgres/Marten + portal
Vite) — los comandos del paso de starter, con los dos ajustes (Mediator MIT, Marten).

### Stress-test addendum (Thesis Defense + Self-Consistency + Hindsight)

**Veredicto confirmado:** READY (núcleo) se sostiene; arquitectura auto-consistente, sin incoherencias.
Aclaración: AS-1 es términos comerciales de reventa, **no** acceso técnico — no bloquea construir/correr.

**No-negociables del día 1 (de hindsight):**

1. Sellar la **versión del matcher** de OpenSanctions en cada expediente desde el primer dictamen (R2).
2. **S3 Object Lock en modo compliance** verificado por test antes de persistir cualquier expediente real (R3).
3. Confirmar por escrito que el uso inicial de OpenSanctions cabe en los términos actuales (de-risk AS-1).

**Recordatorio de secuencia:** entregar y validar disposición a pagar de la **capa 1** (sube Excel →
dictamen + expediente) antes de construir monitoreo/avisos — eco de la tensión C1/C2 del PRD.
