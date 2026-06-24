# Tenancy de dos niveles: Cuenta → Sujeto obligado

El multi-tenant se modela en dos niveles: la **Cuenta** es la frontera de auth, facturación y metering; el **Sujeto obligado** es la frontera legal y de aislamiento de datos (padrón, expedientes y LPB). Una Cuenta contiene uno o más sujetos obligados: un despacho es una cuenta con N sujetos obligados; un sujeto obligado directo es una cuenta con uno solo (él mismo).

**Por qué:** cubre el escenario despacho (factura consolidada, muchos clientes) y el directo (joyería sola) con un único modelo sin casos especiales. Pone el aislamiento donde lo pone la ley —en el sujeto obligado, responsable del expediente y dueño de su LPB confidencial—, mientras la facturación/metering vive en la Cuenta, que es donde el despacho la quiere consolidada.

**Consecuencia:** la LPB se sube y se aísla **por sujeto obligado**, nunca compartida dentro de una cuenta-despacho (la notaría no ve la LPB ni el padrón de la joyería de la misma cuenta). El metering se mide por cuenta para cobrar, pero es atribuible por sujeto obligado. Cambiar esta jerarquía después implica re-particionar todos los datos, por eso se fija desde el inicio.
