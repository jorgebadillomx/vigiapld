# Validación de identidad por integración (puerto), no construcción

La validación de identidad (KYC de identificación del cliente) se resuelve **integrando un proveedor externo tras un puerto** (igual que OpenSanctions y el LLM en Bedrock), **no construyendo** OCR/biometría propios. Dos niveles: (1) barato y casi existente — validación de RFC, consulta 69/69-B (ya hospedada) y validación de formato CURP; (2) fuerte por integración — INE con OCR + biometría facial, CURP autoritativa vía RENAPO, etc., vía API de un proveedor KYC mexicano. El resultado se adjunta al expediente del sujeto evaluado, reforzando la primera obligación LFPIORPI ("identificar al cliente").

**Por qué:** construir identidad (OCR, biometría, conexiones RENAPO/IMSS/SEP) es un proyecto en sí mismo y nos saca de nuestra cuña — orquestamos y producimos el dictamen/expediente, no nos volvemos una empresa de identidad. KYC Systems la incluye, pero integrarla nos pone a paridad funcional sin diluir el foco ni el costo.

**Consecuencia:** **proveedor a definir** — el puerto es estable; la elección concreta (Truora u otros del mercado MX; comparar cobertura INE/RENAPO/biometría y precio) se decide después y se fakea en pruebas como cualquier dependencia externa. Nivel (1) en fase 1; biometría/INE por integración en fase 1 o 2 según demanda del canal.
