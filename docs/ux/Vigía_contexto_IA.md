# Vigía — Contexto para IA

Documento de apoyo para que un modelo entienda el diseño **Vigía** y pueda trabajar sobre él sin volver a inferirlo. Es un *prototipo de diseño en HTML*, no el producto productivo (ese motor real es ASP.NET Core + PostgreSQL; aquí solo está la cara visual e interactiva).

---

## 1. Qué es

**Vigía** es un sitio de **Prevención de Lavado de Dinero (PLD/FT)** para **sujetos obligados** mexicanos bajo la **LFPIORPI** (joyerías, notarías, inmobiliarias, casas de empeño, etc.). Cruza el padrón del cliente contra listas de sanciones, PEP y listas mexicanas y emite un **dictamen auditable** que separa la **coincidencia real** del **homónimo**.

> "Pre lavado de dinero" = **Prevención** de Lavado de Dinero (cumplimiento), NO el delito. Es lo contrario al lavado: detecta a sancionados/PEP para cumplir la ley.

El diseño tiene **dos caras conectadas por un flujo**:
- **Pública (venta):** landing orientada a la urgencia (multa + reforma 2026), con lead magnets para captar.
- **Privada (panel):** administrador donde el sujeto obligado revisa su historial, sube padrón, lee dictámenes y atiende alertas (reloj de aviso 24 h).

Comprador objetivo de la landing: **sujeto obligado directo** (no el despacho).

---

## 2. Marca

- **Nombre:** Vigía (el que vigila desde lo alto → vigilancia + protección).
- **Logotipo:** cuadro redondeado con la inicial **"V"** en serif sobre el color de acento.
- **Badge:** `PLD/FT` (acrónimo correcto de la ley; NO "DLP").
- **Tono de copy:** despacho legal pero moderno y nada acartonado; confiable; la parte privada prioriza ser **clara y usable**.

---

## 3. Archivo y arquitectura

| Archivo | Qué es |
|---|---|
| `Vigía.dc.html` | **Fuente editable**. Design Component (DC): un único componente que pinta en streaming. |
| `Vigía.html` | Export **autónomo** (todo inlineado, funciona sin conexión). Generado; **no editar a mano** — recompilar desde el `.dc.html`. |
| `support.js` | Runtime de los DC (no tocar). |

**Estructura del DC** (`.dc.html`):
- `<helmet>` — fuentes (Google Fonts), tokens de tema en `<style>` (variables CSS), keyframes, resets. **Único lugar con CSS no inline.**
- Template (markup entre `<x-dc>`) — **todo con estilos inline**; usa `var(--token)`. Control de flujo con `<sc-if>` / `<sc-for>`. Holes `{{ ruta.punteada }}`.
- Clase de lógica `class Component extends DCLogic` — `state`, `renderVals()` devuelve los valores que consume el template. Datos demo y handlers viven aquí.

Reglas al editar: estilos **inline** (no clases nuevas), nada de `<script>` fuera de `<helmet>`, no expresiones dentro de `{{ }}` (se calculan en `renderVals`). Editar con herramientas de DC, no reescribir entero.

---

## 4. Sistema visual

**Tipografía**
- Display/titulares: **Spectral** (serif) — da gravitas legal.
- UI/cuerpo: **Hanken Grotesk** (sans, legible).
- Datos/RFC/scores/versiones: **IBM Plex Mono**.

**Temas** (definidos como bloques `[data-theme="…"]` de variables CSS en `<helmet>`):
- `toga` — azul institucional sobre claro frío (por defecto).
- `notario` — papel cálido + oxblood (`#9c3a2a`).
- `salvaguarda` — verde confianza (`#0b6b43`) + dorado.

**Tokens por tema** (mismos nombres en los 3): `--bg --surface --surface-2 --ink --ink-soft --muted --line --accent --accent-ink --accent-soft --hero --on-hero --on-hero-soft --gold`.

**Semántica de banda de confianza / listas** (color-codificada y consistente):
- `--b-real` rojo — **resuelto · real** (coincidencia confirmada).
- `--b-hom` azul — **resuelto · homónimo** (mismo nombre, otra persona).
- `--b-ver` ámbar — **requiere verificación** (falta dato).
- `--b-desc` verde — **descartado por nombre** (limpio).
- `--pep` morado — tipo de lista PEP.

---

## 5. Estructura de pantallas

**Pública (landing), en orden:**
1. Nav (logo, secciones, "Entrar al panel").
2. Hero — badge de urgencia + titular de multa + CTA + stats (320+ fuentes / 24 h / 10 años) + tarjeta de dictamen de muestra.
3. **Lead magnets:** verificación de 1 nombre en vivo (escribe/elige ejemplo → animación de cotejo → dictamen con banda) + calculadora de multa por giro (UMA 2026).
4. El homónimo (diferenciador: mismo nombre, RFC+fecha distinta → dos dictámenes).
5. Cómo funciona (4 pasos).
6. Cobertura de listas (OFAC, ONU, UE, UK, PEP, SAT 69-B, LPB UIF, matching difuso).
7. Precios (Esencial / Negocio / Despacho, por volumen).
8. Trust + disclaimers.
9. Footer.

**Privada (panel), sidebar fijo + main con scroll interno:**
1. **Resumen (dashboard):** KPIs (padrón 248, hits, coincidencias reales, avisos 24 h), **reloj de aviso 24 h con cuenta regresiva en vivo**, hits por banda, corridas recientes, cobertura de listas (versiones).
2. **Subir padrón:** dropzone → mapeo de columnas + validaciones (filas inválidas, duplicados) → procesamiento → resumen.
3. **Dictámenes:** tabla filtrable por banda (Todas/Reales/Homónimos/A verificar/Descartados).
4. **Detalle de dictamen:** veredicto (lista/programa, score, riesgo, acción AML), fundamento + cotejos (nombre/RFC/fecha), prosa redactada (etiqueta LLM/plantilla), expediente (timeline) y, si aplica, reloj de aviso + "Generar aviso XML".

Flujo: la landing entra al panel ("Entrar al panel"); el demo de verificación puede abrir el dictamen completo en el panel; el panel vuelve al sitio público.

---

## 6. Modelo de datos demo

Cada **sujeto evaluado** (en `this.padron`) tiene:
`id, nombre, tipo` (`fisica`|`moral`), `rfc, fnac` (fecha nac.), `coincide`, `listaFuente`, `programa`, `tipoLista` (`sanciones`|`bloqueados-UIF`|`PEP`|`EFOS-EDOS`|`limpio`), `score` (0–1), `banda` (`real`|`homonimo`|`verificar`|`descartado`), `riesgo` (`alto`|`medio`|`bajo`), `accion`, `redaccion` (`LLM`|`plantilla`), `aviso` (id de reloj 24 h o null), `avisoEvento`, `fundamento`, `prosa`.

**Regla central de la banda** (determinística): nombre coincide + RFC **y** fecha iguales → `real`; nombre coincide pero RFC/fecha difieren → `homonimo`; score alto pero faltan RFC/fecha → `verificar`; score bajo → `descartado`. **PEP nunca implica rechazo** (dispara EDD). El LLM solo redacta la prosa; no decide.

---

## 7. Tweaks (props del DC)

Declarados en `data-props`; se leen con `this.props.X` (fallback dentro de `renderVals`). Reconfiguran la *sensación*, no píxeles:
- **`paleta`** — `Toga` | `Notario` | `Salvaguarda` → recolorea toda la app (cambia `data-theme`).
- **`vibe`** — `Editorial` | `Tecnológico` → cambia la fuente de titulares (Spectral serif ↔ Hanken sans) vía `data-vibe`.
- **`tono`** — `Agresivo` | `Equilibrado` | `Sobrio` → reescribe badge, titular y CTA del hero a otro registro.

Hay además un **switcher de tema flotante** (atajo en vivo) que respeta lo elegido en Tweaks.

---

## 8. Glosario (LFPIORPI)

- **Sujeto obligado** — negocio obligado por la ley; sube su padrón y es responsable legal.
- **Padrón** — conjunto de sujetos evaluados (clientes, proveedores, donantes).
- **Dictamen** — veredicto auditable por sujeto evaluado.
- **Banda de confianza** — qué tan firme es el dictamen (4 estados, §6).
- **Reloj de aviso 24 h** — cuenta regresiva legal que arranca con una coincidencia detectada o un intento de operación (Art. 7 Bis); cuenta desde el evento, no desde la lectura.
- **Expediente** — historia auditable del sujeto, se conserva 10 años (Art. 12 Bis).

---

## 9. Disclaimers (mantener)

Vigía es **herramienta de apoyo, no garantía**. No reemplaza la auditoría del Art. 12 Bis ni presenta avisos en el SPPLD del SAT en nombre del cliente. La presentación del aviso y la conservación del expediente son responsabilidad del sujeto obligado. Cifras de multa con base en la UMA 2026 (DOF) y la LFPIORPI.
