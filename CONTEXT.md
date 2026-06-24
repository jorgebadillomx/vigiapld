# PreLavadoDinero — Motor de Screening PLD/FT (LFPIORPI México)

Motor único que cruza el padrón de un sujeto obligado contra listas de sanciones, PEP y listas mexicanas, y emite un dictamen auditable que separa la coincidencia real del homónimo.

## Language

**Cuenta**:
La frontera de autenticación, facturación y metering. Un despacho es una cuenta con muchos sujetos obligados; un sujeto obligado directo es una cuenta con uno solo. Ver ADR-0003.
_Avoid_: Tenant, cliente, cliente final.

**Sujeto obligado**:
La persona o entidad obligada por la LFPIORPI (joyería, notaría, inmobiliaria, etc.) que sube su padrón y es responsable legal de sus expedientes y de su LPB. Es la frontera de aislamiento de datos. Ver ADR-0003.
_Avoid_: Cliente, cliente final, tenant.

**Padrón**:
El conjunto de sujetos evaluados de un sujeto obligado (clientes, proveedores, donantes). Pertenece y se aísla por sujeto obligado.
_Avoid_: Base de clientes, lista, cartera.

**Sujeto evaluado**:
Un renglón del padrón al que se le emite un dictamen: una persona física o moral. Tiene un atributo `tipo de persona` (física | moral) que selecciona la regla de decisión aplicable. Un Beneficiario Controlador es un sujeto evaluado ligado a un sujeto evaluado de tipo moral.
_Avoid_: Nombre (es solo un atributo del sujeto evaluado, no la entidad), registro, fila.

**Tipo de persona**:
Atributo del sujeto evaluado: `física` o `moral`. Derivable del RFC (13 posiciones = física, 12 = moral). Determina qué regla de homonimia aplica.

**Dictamen**:
El veredicto auditable que el motor emite por cada sujeto evaluado: contra qué lista coincidió, con qué score, su banda de confianza, nivel de riesgo y acción AML sugerida, más el fundamento. El LLM solo lo redacta; las reglas determinísticas lo deciden. Siempre se emite con sus campos determinísticos completos; si el LLM (Claude Sonnet en AWS Bedrock, ver ADR-0007) no responde, la prosa usa una plantilla de respaldo y el dictamen se marca `redacción: plantilla` (vs `redacción: LLM`), re-redactable después sin tocar la decisión.
_Avoid_: Resultado, reporte, veredicto.

**Beneficiario Controlador**:
La persona física que finalmente controla a un sujeto evaluado de tipo moral. El cliente lo declara; el producto lo screenea contra todos los tipos de lista y conserva su cadena de control como evidencia. No se resuelve la cadena de forma recursiva. Ver ADR-0006.
_Avoid_: BC (usar el término completo en docs), dueño, accionista.

**Corrida de screening**:
El evento que dispara la emisión de dictámenes sobre un padrón completo o sobre un solo sujeto evaluado. Puede ser inicial (carga del padrón) o de rescreening (recurrente). No es el contenedor de la evidencia — solo el disparador. Ver ADR-0002.
_Avoid_: Batch, lote, proceso, ejecución.

**Expediente**:
Proyección de lectura del rastro: la historia completa de un sujeto evaluado en el tiempo (dictamen inicial + rescreenings + ajustes manuales + avisos). Es lo que el auditor del Art. 12 Bis consume. Se conserva 10 años. Ver ADR-0002.
_Avoid_: Archivo, registro, file.

**Constancia de corrida**:
Proyección de lectura del rastro: el snapshot de qué se hizo en una corrida (fecha, listas y versiones usadas, sujetos evaluados). Evidencia de monitoreo continuo, complementaria al expediente. Ver ADR-0002.

**Cadencia de rescreening**:
La frecuencia con que se re-consulta cada tipo de lista. Los cruces locales (69-B, LPB) corren en cada refresco; la re-consulta a OpenSanctions (sanciones + PEP) corre por cadencia configurable por tier, con un piso legal mínimo igual para todos los tiers. Ver ADR-0005.

**Reloj de aviso**:
La cuenta regresiva de 24h que el sistema arranca y vigila cuando ocurre un evento que obliga a avisar. Sella su `evento origen` y `marca de tiempo origen`; cuenta desde el evento, no desde que el usuario lee la alerta. El producto alerta y cuenta; presentar el aviso es responsabilidad del sujeto obligado.
_Avoid_: Timer, cuenta regresiva, plazo.

**Coincidencia detectada**:
Evento que genera el propio motor (en una corrida o rescreening) cuando un sujeto evaluado pasa a ser coincidencia real contra una lista. Es uno de los dos disparadores del reloj de aviso.

**Intento de operación**:
Evento que **siempre entra por el canal** (el ERP vía API o el usuario en el portal) marcando que un cliente intentó una operación reportable, aunque no se concrete. El motor nunca lo infiere solo. Es el segundo disparador del reloj de aviso (Art. 7 Bis, reforma 2026).

**Aviso**:
El reporte que se presenta al SAT en el portal SPPLD. El producto autogenera y valida su XML y registra el acuse de presentación, pero la presentación la hace el sujeto obligado. Distinto del dictamen (evidencia interna) y de la auditoría del Art. 12 Bis.
_Avoid_: Reporte, notificación, alerta (la alerta es la señal interna; el aviso es el documento legal al SAT).

**Clasificación EBR**:
La calificación de riesgo de un sujeto evaluado para el enfoque basado en riesgos del sujeto obligado (Art. 18 Fr. VII). Se calcula sobre **cinco dimensiones** canónicas: clientes/usuarios · geografía · productos/servicios · canales de distribución · perfil transaccional. **Consume** el nivel de riesgo del dictamen como insumo (dimensión de listas) y el perfil transaccional del módulo de operaciones. No reemplaza al dictamen: un sujeto puede salir `descartado-por-nombre` y aun así quedar riesgo alto en EBR. Es por sujeto evaluado, documentada y exportable.
_Avoid_: Matriz de riesgo (úsese para el artefacto exportable), scoring.

**Operación**:
Una transacción que el sujeto obligado reporta, ligada a un sujeto evaluado. Es un evento del log append-only del rastro; sobre las operaciones corren las reglas de umbral y acumulación en ventanas de 6 meses que pueden disparar el reloj de aviso. El dato lo aporta el cliente (no hay dependencia externa). Ver ADR-0008.
_Avoid_: Transacción, movimiento.

**Partes relacionadas**:
Personas ligadas a un sujeto evaluado más allá del Beneficiario Controlador (socios, avales, beneficiarios, propietarios reales). El cliente las declara; se screenean contra todos los tipos de lista y su evidencia se liga al expediente. Mismo patrón que el [[beneficiario-controlador]]: declaradas, no resueltas de forma recursiva.
_Avoid_: Relacionados, terceros.

**Validación de identidad**:
La verificación de la identidad del sujeto evaluado (RFC, CURP, 69/69-B, y por integración INE/biometría/RENAPO). Se obtiene integrando un proveedor externo tras un puerto, no construyéndola; el resultado se adjunta al expediente. Proveedor a definir. Ver ADR-0009.
_Avoid_: KYC (ambiguo), onboarding.

**Tipo de lista**:
Clasificación de la fuente contra la que coincide un sujeto evaluado, con consecuencia AML distinta: `sanciones` (OFAC/ONU/UE/UK → no operar/reportar), `bloqueados-UIF` (LPB → bloqueo legal vinculante, la más severa en MX), `PEP` (dispara EDD y aprobación, NO rechazo), `EFOS-EDOS` (SAT 69-B, factor de riesgo fiscal, no rechazo automático). La acción sugerida y el nivel de riesgo se derivan de la matriz `(banda de confianza × tipo de lista)`. Ver ADR-0004.

**Política de corte**:
La regla versionada del operador que convierte el score continuo de OpenSanctions en bandas (candidato fuerte / zona gris → `requiere-verificación` / `descartado-por-nombre`). La fija el operador, no el cliente, y su versión se sella en cada expediente. Ver ADR-0001.

**Banda de confianza**:
El grado de resolución del dictamen, no un score numérico. Cuatro estados: `resuelto-real` (nombre + RFC + fecha de nacimiento coinciden), `resuelto-homónimo` (nombre coincide pero RFC/fecha no), `descartado-por-nombre` (score bajo, sin coincidencia relevante), `requiere-verificación` (score alto pero faltan RFC/fecha para resolver). El producto siempre emite dictamen; la banda dice qué tan firme es y qué dato falta para endurecerla.
_Avoid_: Indeterminado (demasiado vago; usar `requiere-verificación` o `descartado-por-nombre` según el score).
