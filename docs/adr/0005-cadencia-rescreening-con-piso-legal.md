# Cadencia de rescreening gobernada por costo, con piso legal por debajo del tier

El rescreening separa los cruces por costo:

- **Cruces locales (SAT 69-B hospedada por el operador, LPB subida por el cliente) → se corren en cada refresco de lista**, sin costo marginal. Cubre dos de los cuatro tipos de lista, incluido el más severo (bloqueados-UIF).
- **Re-consulta a OpenSanctions (sanciones + PEP) → se gobierna por una cadencia configurable por tier** (p. ej. mensual básico, semanal premium), porque cada consulta cuesta (€0.10 con API; ≈0 con self-hosted).

Sobre esa cadencia se impone un **piso legal mínimo igual para todos los tiers**: ningún plan, ni el más barato, re-consulta OpenSanctions menos seguido que el piso. Los tiers solo compran mayor frecuencia, nunca menos que el piso.

**Por qué:** mantiene el margen sano (la tesis del brief: el costo real es OpenSanctions, no la IA) y escala de API por consulta a self-hosted sin rediseño, porque el acceso a OpenSanctions ya está tras un puerto. El piso evita la exposición legal de dejar a un cliente de plan básico descubierto demasiado tiempo, que dañaría su cumplimiento y la reputación del producto.

**Consecuencia:** cada dictamen sella la versión de cada lista y la fecha contra la que se evaluó, así el expediente prueba la cadencia aplicada. No se cachea el veredicto de sanciones como sustituto de re-consultar; el rescreening es una re-consulta real gobernada por la cadencia. El valor numérico del piso se calibra con el especialista PLD (pendiente, ver brief riesgo del especialista legal).
