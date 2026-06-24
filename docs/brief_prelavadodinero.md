# Handoff — Skill de Screening PLD (AML / Sanciones / PEP) para México

> Documento de traspaso. Consolida lo definido para construir una skill de Capafy + web app que verifica padrones de clientes contra listas de sanciones, PEP y listas mexicanas, y entrega un dictamen PLD auditable. Listo para alimentar a BMAD o retomar en una nueva sesión.
> Estado: definición de negocio cerrada. **Panorama competitivo actualizado (jun-2026, ver Sección 10).** **Arquitectura revisada: 3 canales de primera clase —Capafy, portal web propio y API/white-label— y el cerebro del dictamen reubicado dentro del motor (ver Secciones 4 y 5).** Pendiente: confirmaciones con OpenSanctions y arranque de build.

---

## 1. Resumen ejecutivo

Producto: motor de **screening PLD/FT** (prevención de lavado de dinero / financiamiento al terrorismo) para **sujetos obligados** mexicanos bajo la LFPIORPI. El cliente sube su padrón (clientes, proveedores, donantes); el sistema lo cruza contra listas vivas de sanciones, PEP y listas mexicanas, y devuelve un **dictamen auditable**: quién es coincidencia real vs. homónimo, contra qué lista, score, nivel de riesgo y acción AML sugerida, más el expediente que aguanta una auditoría.

Se distribuye en **un solo motor con tres canales de primera clase**: skill en Capafy (despachos/devs), portal web propio con pantallas y formularios (sujeto obligado directo), y API / white-label para integradores y ERPs (Contpaqi, Aspel, SAP B1). El motor es el centro; los tres canales solo lo invocan y presentan su resultado. El valor real no es "buscar en una lista" — es **el dictamen que separa la coincidencia real del homónimo** y la **evidencia auditable** que la ley exige guardar 10 años.

**Realidad de mercado (jun-2026):** el screening de listas mexicanas (OFAC + ONU + PEP + SAT 69-B) **ya no es una capacidad novedosa** — hay al menos 6 jugadores mexicanos haciéndolo, varios con clientes grandes (ver Sección 10). Esto no mata la idea, pero obliga a competir con una **cuña afilada** —distribución en Capafy + nicho de sujetos obligados chicos + dictamen LLM + precio— y **no** con un "me-too" de suite completa. El screening en sí no se vende como diferenciador; el canal, el foco y el dictamen sí.

---

## 2. Problema y oportunidad (LFPIORPI)

- La **Ley Antilavado (LFPIORPI)** obliga a 17 actividades vulnerables (Art. 17): joyerías, inmobiliarias, desarrollo inmobiliario, notarios, vehículos, casas de empeño, activos virtuales, obras de arte, donativos, servicios profesionales, agentes aduanales, etc.
- Obligaciones: identificar clientes, **cruzar contra listas**, presentar avisos al SAT, **conservar documentación 10 años**, enfoque basado en riesgos y **mecanismos automatizados**.
- Multas por incumplir: de 200 a 65,000 UMAs ≈ **$23,462 a $7,625,150 MXN (2026)** (UMA 2026 = $117.31 diarios). Ese es el "por qué pagan": evitar una multa millonaria, no ahorrar.
- **Reforma marzo 2026 (DOF 27/03/2026) — viento a favor:**
  - Capítulo PEPs (Arts. 45 Bis–Quinquies): **screening de PEP ahora obligatorio** para actividades vulnerables (antes solo sector financiero).
  - Auditoría obligatoria vía **dictamen interno o externo** (Art. 12 Bis).
  - **Aviso de 24 horas** por intento de operación, aunque no se concrete (Art. 7 Bis).
  - Acumulación de operaciones en periodos de 6 meses (Art. 7).
  - **Beneficiario Controlador (BC)** se consolida como eje central: toda sociedad mercantil —aun sin actividad vulnerable— debe identificar a su BC, conservar evidencia y atender requerimientos de SAT/UIF. (Ver brecha de feature en Sección 10.)
  - Términos exactos pendientes de reglas de carácter general, pero la obligación ya es exigible.
- El dolor de cabeza operativo #1 del sector es la **homonimia** (falsos positivos). Resolverlo bien = el diferenciador de producto.

---

## 3. Modelo de negocio (Capafy + API propia)

- La **skill corre en Capafy** (cerrada, en su nube) y/o el cliente la usa por **web app propia**.
- La skill conecta a una **Core API operada y pagada por Jorge**, que a su vez consume APIs de terceros (OpenSanctions, DeepSeek). Jorge absorbe el costo de esas APIs y **monetiza por acceso/suscripción**.
- Capafy: comisión **20%** + Platform Sandbox Fee (solo en modo suscripción) + cuota única de certificación US$0.99. Modos: suscripción / renta por hora / descarga.
  - **Evitar el modo descarga**: expone prompts, lógica y código fuente al usuario.
- En la web, Jorge cobra directo (sin 20% ni Sandbox Fee) → mucho mejor margen por cliente.

---

## 4. Canales — un motor, tres caras de primera clase

El **motor (Core API) es el centro**; los tres canales son cascarones delgados que lo invocan y presentan su resultado. No son tres productos.

| Canal | Público objetivo | Notas |
|---|---|---|
| **Skill de Capafy** | Despachos/consultores PLD, contadores, devs que ya viven en agentes | Revenden el screening a SUS clientes. Corre cerrada en la nube de Capafy. La skill solo **presenta** el dictamen que ya armó la API (no lo reescribe). **Canal sin competencia directa hoy (ver Sección 10).** |
| **Portal web propio** | El sujeto obligado directo (joyería, notaría, inmobiliaria) | GUI con pantallas y formularios que **simula lo que hace Capafy**: sube Excel → dashboard de hits → revisión/aprobación del dictamen → export PDF. Cobro directo (sin 20% ni Sandbox Fee). |
| **API / white-label** | Integradores, ERPs y software contable (Contpaqi, Aspel, SAP B1) y despachos con sistema propio | El motor expuesto. Dos sabores: **API pura (headless, JSON)** y **white-label (UI embebible bajo su marca)**. **El canal de mayor apalancamiento** (un ERP = muchos clientes finales) y el que **justifica la licencia plana de OpenSanctions** (ver Sección 9). Requiere contrato de API versionado, webhooks, jobs async y self-serve de API keys. |

*A futuro:* **PWA** (instalable + push para alertas de rescreening) sobre el mismo motor; bajo ROI para B2B documental, por eso no es de las tres primeras.

---

## 5. Arquitectura técnica

```
CANALES      Skill (Capafy)  ·  Portal web (forms)  ·  API / white-label (ERPs)
                    \                  |                       /
                     v                 v                      v
GATEWAY        Auth + metering · API keys · webhooks   (multi-tenant, cobro por uso)
                                       v
MOTOR        CORE API (ASP.NET Core)  ──▶  SQL Server (rastro 10 años)
             ├─ capa determinística: matching · 69-B/LPB · score · reglas RFC/fecha · PDF
             └─ capa de orquestación: DeepSeek → REDACTA EL DICTAMEN   ◄ (antes en Capafy)
                  /            |              |            \
                 v             v              v             v
DEPENDENCIAS  OpenSanctions  SAT 69/69-B   LPB (la sube     DeepSeek
              (Jorge paga)   (hospedadas)  el cliente)      (Flash + Pro)
              + Hangfire: refresco listas SAT + rescreening recurrente → webhooks/alertas
```

**Decisiones de arquitectura (cerradas / revisadas):**
1. **Los tres canales llaman a la Core API de Jorge, NO a OpenSanctions directo.** El valor (capa México, dictamen, rastro, rescreening) vive en el motor; una sola integración con OpenSanctions = una sola licencia, un solo medidor, un solo punto de control de costos.
2. **[REVISADA por la arquitectura de 3 canales] El cerebro del dictamen (DeepSeek + reglas) vive DENTRO del motor**, en una capa de orquestación sobre el núcleo determinístico. Los tres canales solo **muestran** el dictamen que la API ya armó. Antes vivía solo en Capafy; se reubicó porque el portal web y la API/white-label **no tienen agente** que lo redacte, y duplicar esa lógica por canal se desincronizaría. El núcleo de datos (matching, listas, score, reglas, rastro) sigue siendo determinístico; el LLM se aísla en su capa. El "doble IA" ya no aplica: en Capafy la skill no reescribe el dictamen (solo lo presenta), y en web/API no hay agente, así que el LLM del motor es el único.
3. **La LPB es confidencial → el cliente sube su propia copia**; Jorge nunca la hospeda. Las listas SAT 69/69-B sí son públicas y Jorge las hospeda/refresca.
4. **El dictamen decide lo legalmente sensible con reglas** (¿coincide RFC + fecha de nacimiento? → real; ¿no? → homónimo) y el LLM solo lo redacta. Más barato, consistente y auditable.

**Implicaciones del canal API / white-label (Sección 4):** el esquema del dictamen pasa a ser un **contrato público versionado** (`/v1/`, changelog, política de deprecación) → hay que clavarlo desde el inicio (Sección 12, paso 3). La API debe soportar **single-name síncrono** (onboarding en tiempo real) **y batch async** (padrón → job Hangfire → webhook/poll), más **API keys self-serve, rate limits y webhooks** de rescreening en el gateway.

**Stack (de Jorge, probado):**
- Backend/motor: **ASP.NET Core + EF Core**.
- Datos y rastro: **SQL Server** (opción PostgreSQL/Docker).
- Jobs: **Hangfire** → rescreening recurrente (el "reloj" del aviso 24h) + refresco de listas SAT.
- Frontend web: **Vite + React + TypeScript + shadcn/ui + Tailwind v4 + TanStack Query + React Router**.
- Multi-tenant + metering en el gateway (clave para cobrar y recuperar costo de OpenSanctions).

---

## 6. Stack de verificación (fuentes y APIs) — CONFIRMADO

**Resumen:** OpenSanctions (global + PEP, vía API o self-hosted) + SAT 69-B (público, lo hospeda Jorge) + LPB del cliente (confidencial, la sube él) + DeepSeek para redactar el dictamen. Esa es la verificación completa.

| Fuente | Qué cubre | Acceso | Quién la hospeda |
|---|---|---|---|
| **OpenSanctions** | 320+ fuentes: OFAC, ONU, UE, UK + **PEPs**, matching difuso | API `api.opensanctions.org` endpoint `/match` (€0.10/consulta) o self-hosted (licencia plana) | Jorge (motor) |
| **SAT 69 / 69-B (EFOS/EDOS)** | Factureras / contribuyentes incumplidos (MX) | **Pública**, datos abiertos, Excel/CSV, ~trimestral | Jorge (la descarga y refresca con Hangfire) |
| **UIF Lista de Personas Bloqueadas (LPB)** | Bloqueados por la UIF (MX) | **Confidencial**, portal SITI/SPPLD, solo sujetos obligados con credenciales | El **cliente** sube su copia |
| **UIF Lista de PEP (MX)** | PEPs mexicanos | Medio electrónico **pendiente** (reforma 2026) | Mientras tanto, cubierto vía OpenSanctions |
| **DeepSeek** | Redacción del dictamen | API (`api.deepseek.com`) | Jorge (key inyectada en bóveda de Capafy) |

**Fuentes oficiales primarias que OpenSanctions ya agrega** (gratis; por si se quiere jalar directo o cross-checkear):
- **OFAC (EE.UU.):** Sanctions List Service (SDN + Consolidated, con API) → `ofac.treasury.gov/sanctions-list-service`
- **ONU:** Consolidated List → `main.un.org/securitycouncil` (`scsanctions.un.org`)
- **UE:** EU Consolidated Financial Sanctions List → `data.europa.eu` / servicio FSF
- **Reino Unido:** **UK Sanctions List** en `gov.uk` (la lista consolidada de OFSI cerró el 28-ene-2026)

**Llamada típica:** enviar a `/match` el **nombre + RFC + fecha de nacimiento** (los dos últimos matan falsos positivos) y leer el score por coincidencia.

**Modelos DeepSeek:** V4 **Flash** (orquestador, barato) + V4 **Pro** (dictamen, razonamiento).

---

## 7. Flujo de producto (quién paga, input, output)

- **Quién paga:** el sujeto obligado (joyería, inmobiliaria, notaría, etc.) **o** un despacho/contador que lleva el PLD de muchos.
- **Input (lo que dan):** su **padrón en Excel/CSV** — clientes, proveedores, donantes. Mínimo nombres; idealmente **+ RFC + fecha de nacimiento**. Para uso continuo, agregan clientes nuevos o re-suben el padrón y el sistema rescrea solo.
- **Output (lo que reciben):**
  1. **Dictamen**: por nombre — coincidencia real vs. homónimo, lista (OFAC/ONU/SAT/PEP), score, nivel de riesgo, acción sugerida.
  2. **Archivo de evidencia auditable** (PDF/Excel): qué se revisó, cuándo, contra qué listas y versión, qué coincidió, qué se decidió. El expediente de 10 años.
  3. **Alertas**: cuando el rescreening detecta que un cliente existente cae en una lista (dispara el aviso 24h).

Ciclo: **suben padrón → dictamen + evidencia → guardan 10 años y presentan avisos al SAT cuando aplica → en la auditoría 12 Bis, la evidencia es la prueba.**

---

## 8. Cumplimiento legal

- **Almacenamiento:** obligatorio **10 años** — pero es obligación **del sujeto obligado**, no de Jorge. El producto lo genera/guarda automáticamente y lo deja descargable. Jorge es la herramienta, no el responsable legal del expediente.
- **Certificación — separar tres cosas:**
  - El **dictamen de screening** NO lo certifica ninguna autoridad; es evidencia interna.
  - El **"aviso"** sí se presenta al SAT (portal SPPLD `sppld.sat.gob.mx`), por operación reportable o coincidencia en listas (24h).
  - El **dictamen de auditoría (Art. 12 Bis)** es una **auditoría periódica del programa PLD** (interna o externa) que el SAT puede requerir. El producto **no la reemplaza, pero produce la evidencia que el auditor necesita** → punto de venta.
- Las actividades vulnerables **no** requieren "oficial de cumplimiento certificado" (esa figura con examen es exclusiva del sector financiero).
- *Nota: confirmar detalles finos con un especialista PLD; varias reglas de la reforma 2026 siguen publicándose.*

---

## 9. Economía unitaria y costos reales

**Precios verificados (jun-2026):**
- DeepSeek V4 **Flash**: $0.14 input / $0.28 output por 1M tokens.
- DeepSeek V4 **Pro**: ~$1.74 / $3.48 por 1M (post-promo). **Verificar** — la promo de lanzamiento lo dejaba ~$0.44/$0.87.
- OpenSanctions API: **€0.10 por consulta** (un lote de N nombres = N consultas).
- Capafy: 20% comisión + Sandbox Fee (suscripción) + US$0.99 certificación única.
- *Ancla de mercado global:* ComplyAdvantage parte de ~US$99.99/mes por 1,000 entidades monitoreadas. Útil como referencia de techo de precio por volumen.

**Por corrida de 500 nombres:**
- DeepSeek (orquestador + dictamen): **~$0.05** (centavos — despreciable, el "doble IA" es ruido).
- OpenSanctions: **~$55** (99.9% del costo variable). **El costo real es OpenSanctions, no la IA.**

**Costos fijos:** infra ~$30/mes (VPS chico + Postgres en el mismo VPS + dominio + monitoreo). Mantenimiento = tiempo de Jorge.

**Licencia OpenSanctions — estructura (NO es por cliente ni por consulta, es plana):**
- La licencia plana cubre **clientes ilimitados**; el precio depende del **caso de uso** (reseller) y el **volumen**, no de un conteo de clientes.
- El API por consulta (€0.10) es lo más barato **debajo de ~30,000 consultas/mes** (≈ **60 clientes** de 500 nombres); arriba conviene el plan plano self-hosted.
- Ancla del self-hosted **interno** ≈ **~€3,000/mes**; el **reseller es más caro** (cotización).
- **Se puede arrancar con el API por consulta para uso reseller** → **sin costo fijo grande el día 1**. La licencia plana es jugada de escala (~60–100 clientes). **El canal API / white-label (Sección 4) es justo lo que dispara esa escala:** un solo integrador/ERP con muchos clientes finales puede empujarte arriba del umbral de ~60 clientes y voltear el margen hacia la licencia plana mucho más rápido que vendiendo uno por uno.

**Ejemplo (a $99/mes, 10 clientes, SaaS):** margen Web ~$44/cliente, utilidad ~$260/mes; margen Capafy ~$22/cliente, utilidad ~$38/mes. Break-even ~4 (web) a ~8 (Capafy) clientes.

**Regla de oro:** precio **por volumen** (o tiers por tamaño de padrón). Una suscripción plana barata se va a **números rojos** contra el €0.10/nombre de OpenSanctions.

> Modelo editable: `modelo_costos_pld.xlsx` (ya generado; cambiar celdas azules).

---

## 10. Panorama competitivo (actualizado jun-2026)

> **Lectura honesta:** el screening de listas mexicanas (OFAC + ONU + PEP + SAT 69-B) **no es una capacidad novedosa**. Ya hay al menos 6 jugadores mexicanos activos que lo hacen, varios con clientes grandes y SEO agresivo sobre "mejor software PLD". No mata la idea, pero obliga a competir con una cuña afilada (distribución + nicho + dictamen + precio), **no** con un "me-too" de suite completa. Tu foso no es el screening en sí.

### Tier A — SaaS PLD mexicano (competencia directa, mismo comprador)

| Producto | Posicionamiento | Qué tan cerca está de ti |
|---|---|---|
| **ArmorAML** (Spot IT Solutions, +20 años) | Gama alta. Bancos, fintechs, aseguradoras, SOFOMes + actividades vulnerables. ISO 27001, API <1 s, batch masivo. Clientes: Zurich Santander, Stori, RappiCard, Quálitas, Clip, Nacional Monte de Piedad, El Palacio de Hierro. | **El más parecido en función:** ya cruza OFAC + ONU + PEP + SAT EFOS con algoritmos fonéticos. Apunta arriba (alto volumen / CNBV); deja descubierto al sujeto obligado chico. |
| **KYC Systems** | Mexico-native, **enfoque láser en actividades vulnerables** (tu nicho exacto). Verticalizado por giro (vehículos, activos virtuales, juegos/sorteos, traslado de valores). Venta por WhatsApp, SMB-friendly. | **El más cercano a tu nicho.** Multi-lista completa (PEP, UIF bloqueados, OFAC, ONU, 69-B, GAFI) + avisos XML SPPLD/SITI + EBR + beneficiario controlador. |
| **Regcheq** | Regional LATAM (MX/Chile/Brasil/Perú), +500 clientes, suite generalista. Fundada 2021. | Marca fuerte, full-suite. Compite por el mismo sujeto obligado. |
| **Artu** (artu.ai) | IA-nativa, modular (pagas por módulo), implementación ~4 semanas. **Máquina de contenido/SEO** (controla/inclina el sitio `pld.mx` y satura "software PLD"). PyME/vulnerable. Clientes LATAM: Nubank, Jeeves. | Mensajería fuerte de PEP + 69-B. Tu competidor más "moderno" y ruidoso en marketing. *Ojo: trata los rankings de pld.mx como marketing de Artu, no como fuente neutral.* |
| **ALDDA, PreveNet** | Generalistas "clásicos" — escalón "sal de Excel" / expedientes básicos. | Gama baja, poco técnica. |
| **AppsPLD** | Económico, específico. | Gama baja. |
| **Truora** | Más verificación de identidad / onboarding (biometría, KYC-lite). | Adyacente, no PLD completo. |

**Segmentación que usa el mercado (útil para posicionarte):** alto volumen / CNBV estricto → ArmorAML; actividad vulnerable que quiere automatizar avisos al SAT de forma sencilla → Artu; solo salir de Excel / expedientes básicos → generalista clásico (Regcheq, ALDDA, PreveNet).

### Tier B — Suites AML enterprise globales (para bancos grandes; NO es tu comprador)
**NICE Actimize, SAS AML, Oracle FCCM.** Implementación 8–12+ meses, presupuestos >US$500k. Irrelevantes para el sujeto obligado chico, pero marcan el techo del mercado.

### Tier C — Datos/screening de sanciones global (la capa que todos envuelven; aquí vive OpenSanctions)
- **Premium** (grado banco, caro, por cotización): **LSEG World-Check, Dow Jones, LexisNexis, Moody's/RDC**. World-Check es explícitamente fuerte en cobertura LatAm + PEP.
- **API-first más baratos:** **ComplyAdvantage** (entrada ~US$99.99/1000 entidades), **Sanctions.io, NameScan, Sanction Scanner, SanctScan, ZIGRAM**.
- **Open data:** **OpenSanctions** (tu fuente) — transparente, self-hosteable, barata.
- **Clave:** todos cubren OFAC/ONU/UE/UK + PEP + matching difuso + score + rastro + rescreening + alertas. **Ninguno está localizado a México** (sin 69-B, LPB UIF, RFC, avisos SPPLD, dictamen en español, encuadre LFPIORPI). Esta capa es **commodity/upstream**, no compite con el producto mexicano terminado. **Tu foso no es el dato de sanciones — es la localización + el dictamen + la distribución.** (Corolario: un competidor podría envolver cualquiera de estas APIs igual que tú envuelves OpenSanctions; por eso el dato no defiende nada.)

### Tier D — Verificadores RFC/69-B puntuales (commodity, NO son PLD)
Apify `leongael/verificador-rfc-mexico` (RFC + 69/69-B, ~US$0.21–0.30 / 1,000), `parseforge/mexico-rfc-scraper`, etc. Caso de uso **fiscal/CFDI** (¿es válido el RFC?, ¿es EFOS para no perder deducibilidad?), no antilavado. Baratos, dormidos, sin sanciones/PEP/matching por nombre/dictamen. **Confirman que la capa 69-B es regalada** → no la cobres como diferenciador.

### Canal de distribución (Capafy) — tu cuña real
En el marketplace de Capafy hoy solo existe **una skill fiscal MX** (ISR/IVA de anfitrión de Airbnb); **no hay ninguna skill de PLD/screening**. El nicho está abierto. Arquitectura confirmada: la skill corre **cerrada en la nube de Capafy**, el creador cobra por corrida/suscripción y los prompts/código quedan protegidos. **La distribución agente-nativa a despachos/devs es lo que ningún incumbente PLD-MX tiene** — es tu diferenciador más limpio.

### Diferenciador que sobrevive al análisis
1. **Distribución Capafy / agente-nativa** a despachos PLD, contadores y devs que revenden a sus clientes. Nadie en PLD-MX juega ahí.
2. **Cola larga de sujetos obligados chicos** a quienes ArmorAML/Regcheq les queda grande o caro: foco en "subir Excel → dictamen + evidencia", no en suite completa.
3. **Dictamen redactado por LLM + resolución de homonimia** como UX (RFC + fecha nac. para matar falsos positivos).
4. **Precio por volumen** y simplicidad de adopción.
5. **API mexicana "lista para LFPIORPI":** frente a las APIs globales (ComplyAdvantage, OpenSanctions, Sanctions.io) que solo dan el match crudo, tu canal API/white-label entrega un **dictamen LFPIORPI terminado + 69-B/LPB/RFC** en una sola llamada. Pitch a un ERP/contador mexicano: "no integres 5 fuentes ni construyas el dictamen — llama a una API mexicana".
> **Lo que NO es diferenciador:** la capacidad de screening en sí (los incumbentes ya cruzan OFAC/ONU/PEP/69-B). No lo vendas como si fuera nuevo.

### Brechas de feature vs. incumbentes (table-stakes que el brief subestimaba)
- **Beneficiario Controlador (BC):** la reforma 2026 lo vuelve central y obliga a **toda sociedad mercantil** (aun sin actividad vulnerable) a identificarlo → **amplía tu mercado direccionable**. KYC Systems y ArmorAML ya lo hacen. Considéralo.
- **Generación de avisos XML para el SPPLD:** KYC Systems, Artu y ArmorAML los **autogeneran**. El brief trata el aviso como tarea del cliente; **autogenerar el XML** es un value-add esperado y lo que separa un producto de un simple "buscador de listas".
- **EBR / matriz de riesgo** documentada (Art. 18 Fr. VII): los incumbentes la traen de serie.
> **Decisión de alcance pendiente:** ¿te quedas como **"screening + dictamen" enfocado** (cuña delgada, fácil de vender en Capafy) o creces hacia **suite completa** (compites de frente con KYC Systems/Regcheq)? **Recomendación:** arranca delgado y diferenciado por canal; BC y avisos XML como fase 2.

---

## 11. Decisiones abiertas / riesgos a confirmar

1. **[CRÍTICO] OpenSanctions — licencia reseller vs. API.** Confirmar con ventas si el modelo (revender dictámenes a muchos clientes finales vía Capafy) se permite bajo los **términos del API hospedado** o exige la licencia OEM/reseller. La evidencia dice que el API sirve para casos externos, pero pedirlo por escrito. **Es el mayor swing del ROI.**
2. **Precio vigente de DeepSeek V4 Pro** (¿promo permanente ~$0.44/$0.87 o steady-state ~$1.74/$3.48?).
3. **Monto exacto del Platform Sandbox Fee** de Capafy (está en sus docs).
4. **Responsabilidad legal / disclaimers:** posicionar como **herramienta de apoyo, no garantía**; la responsabilidad del aviso es del sujeto obligado.
5. **Competencia (ver Sección 10):** el campo está **más poblado** de lo que asumíamos. ArmorAML y KYC Systems ya hacen el screening multi-lista (OFAC/ONU/PEP/69-B) que planeábamos, con clientes reales. **El screening dejó de ser diferenciador**; el diferenciador real es distribución Capafy + nicho chico + dictamen LLM + precio. Riesgo concreto: entrar como late-comer "me-too" a una categoría saturada de SEO. Mitigación: cuña delgada por canal, no suite.
6. **Pricing por volumen** desde el día 1 (no suscripción plana barata).
7. **Alcance de producto** (decisión de Sección 10): screening+dictamen enfocado vs. suite. Define esto antes de BMAD, porque cambia el PRD.

---

## 12. Siguientes pasos sugeridos (BMAD / build)

1. **Cerrar la decisión de alcance** (Sección 10 / riesgo 7): cuña delgada (screening + dictamen) vs. suite. Recomendado: delgada en fase 1.
2. Meter este documento a **BMAD** como project brief y generar PRD + arquitectura + historias.
3. Definir el **esquema de salida del dictamen** (columnas: nombre, RFC, fecha nac., coincidencia S/N, lista/fuente, programa, score, decisión real/homónimo, fundamento, riesgo, acción, fecha de corte, versión de lista). **Este esquema es ahora un contrato público de API (Secciones 4 y 5): versiónalo (`/v1/`) y clávalo antes de exponer la API.**
4. Construir el **motor (Core API)**: ingesta de padrón → `/match` de OpenSanctions → cruce con SAT 69-B (hospedada) + LPB (subida por cliente) → score + reglas RFC/fecha en el núcleo determinístico → **capa de orquestación: DeepSeek redacta el dictamen** → persistencia del rastro en SQL Server.
5. Job de **Hangfire**: refresco de listas SAT + rescreening recurrente con alertas (webhooks).
6. **Portal web propio** (stack React) → subir Excel, dashboard de hits, revisión/aprobación del dictamen, export PDF, configuración de rescreening.
7. Empaquetar la **skill de Capafy** (SKILL.md + scripts) que llama a la Core API y solo presenta el dictamen; key de DeepSeek y de la Core API en la bóveda de Capafy.
8. **Canal API / white-label:** publicar la API con docs, **API keys self-serve, rate limits, webhooks** de rescreening, modos **síncrono + batch async**, y (fase 2) un **widget/embed white-label**. Primer objetivo de venta: un ERP/despacho con cartera (apalanca la licencia plana, Sección 9).
9. Llamada a **ventas de OpenSanctions** para cerrar la pregunta de licencia (riesgo 1 de la Sección 11).
10. *(Fase 2)* Beneficiario Controlador + autogeneración de avisos XML SPPLD (brechas de la Sección 10).

---

## Anexo — URLs de referencia

**Fuentes de datos y APIs**
- OpenSanctions: `opensanctions.org/api/` · `api.opensanctions.org` · licencias `opensanctions.org/licensing/`
- OFAC SLS: `ofac.treasury.gov/sanctions-list-service`
- ONU Consolidated List: `main.un.org/securitycouncil` · `scsanctions.un.org`
- UE (FSF / datos): `data.europa.eu`
- UK Sanctions List: `gov.uk` (UK Sanctions List)
- SAT 69-B (datos abiertos): `omawww.sat.gob.mx/cifras_sat/Paginas/datos/vinculo.html?page=ListCompleta69B.html`
- SPPLD (avisos): `sppld.sat.gob.mx`
- DeepSeek API: `api.deepseek.com`
- Capafy: `capafy.ai` · publishers `capafy.ai/earn` · repo `github.com/Capafy/Capafy-skills`

**Competencia (Sección 10)**
- ArmorAML: `armor-aml.com`
- KYC Systems: `kyc-systems.com`
- Regcheq: `regcheq.com.mx`
- Artu: `artu.ai` (y su propiedad de contenido `pld.mx`)
- Comparador (sesgo a ArmorAML): `softwarepld.com.mx`
- Verificador RFC commodity (Apify): `apify.com/leongael/verificador-rfc-mexico`
- Datos globales: `complyadvantage.com` · `lseg.com` (World-Check) · `sanctions.io` · `namescan.io`

**Referencia legal**
- Guía LFPIORPI: `lfpiorpi.com`
- Beneficiario Controlador / reforma: `contpaqi.com` (tendencias fiscales)
