# Rastro de auditoría como log de eventos append-only

El rastro de 10 años se modela como un **log de eventos append-only** (dictamen emitido, corrida de rescreening, ajuste manual, aviso disparado), nunca como filas mutables. Sobre ese log se proyectan dos vistas de lectura: el **Expediente** (la historia completa de un sujeto evaluado en el tiempo) y la **Constancia de corrida** (snapshot de qué se hizo en una fecha).

**Por qué:** el auditor del Art. 12 Bis pregunta por sujeto y por continuidad temporal ("muéstrame todo lo que supiste de Juan Pérez y cuándo"), no por el batch de un día. Un log append-only es la forma más limpia y defendible de garantizar la inmutabilidad que exige la ley: nada se sobrescribe, un ajuste manual es un evento nuevo con su justificación, autor y fecha. Evita duplicar datos entre snapshots.

**Consecuencia:** la inmutabilidad del PRD se cumple por construcción. La persistencia (SQL Server) almacena el log y materializa las dos proyecciones. Cambiar este modelo después de emitir expedientes sería muy costoso, por eso se fija desde el inicio.
