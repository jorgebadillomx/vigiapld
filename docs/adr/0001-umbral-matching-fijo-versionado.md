# Umbral de matching fijo y versionado, con zona gris

El score de OpenSanctions `/match` necesita un corte para convertir un match crudo en candidato del dictamen. Decidimos que el umbral lo **fija y versiona el operador** (no el cliente/tenant), con tres bandas: corte alto → candidato fuerte; zona gris → `requiere-verificación` aun con nombre completo; bajo → `descartado-por-nombre`. La versión de la política de corte se registra en cada expediente.

**Por qué:** el sujeto obligado chico no sabe ni debe calibrar un umbral de matching difuso; trasladárselo le pasa el riesgo y debilita la defensa en auditoría. Un umbral fijo y versionado es lo más defendible ("usamos la política de corte vN, registrada en el expediente"). La zona gris evita el binario que produce los dos errores caros (falsos positivos / falsos negativos), empujando lo ambiguo a verificación humana.

**Consecuencia:** el apetito de riesgo (EBR) ajusta el nivel de riesgo y la acción sugerida, **no** el umbral de matching — quedan desacoplados a propósito. Los valores numéricos exactos de los cortes se calibran en build y se versionan; cambiarlos retroactivamente sobre expedientes ya emitidos no se permite (la política queda sellada en el expediente).
