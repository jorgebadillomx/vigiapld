# Separar resolución de identidad de consecuencia AML

El núcleo determinístico separa dos decisiones que el PRD trataba como una sola "coincidencia real":

1. **Resolución de identidad** — ¿es el mismo sujeto? Produce la banda de confianza (real / homónimo / requiere-verificación / descartado-por-nombre) a partir de nombre + RFC + fecha de nacimiento.
2. **Consecuencia AML** — ¿qué hacer? La acción sugerida y el nivel de riesgo se derivan de una matriz `(banda de confianza × tipo de lista)`, no del binario coincidió/no.

Los tipos de lista canónicos son `sanciones`, `bloqueados-UIF`, `PEP`, `EFOS-EDOS`, y **no** son equivalentes en consecuencia.

**Por qué:** las listas tienen consecuencias legales distintas. Sanciones y bloqueados-UIF → no operar/reportar. Un **PEP no es un flag criminal**: dispara debida diligencia reforzada (EDD) y aprobación, no rechazo — tratarlo como sancionado es un error grave de producto. EFOS-EDOS es un asunto fiscal (factureras) que es factor de riesgo, no un "no operar" automático. Meter todo en un binario produce dictámenes incorrectos.

**Consecuencia:** el LLM solo redacta; la matriz determinística decide acción y riesgo. Cambiar la semántica (p. ej. que PEP escale a rechazo en algún giro) es una decisión de la matriz, registrable, pero la separación identidad↔consecuencia se mantiene.
