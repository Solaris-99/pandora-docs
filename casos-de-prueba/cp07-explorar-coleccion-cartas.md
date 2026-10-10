# CP07 — Explorar colección/cartas (CU16)

**Caso de uso:** [CU16 — Explorar colección/cartas](/casos-de-uso/cu16-explorar-coleccion-cartas.md)

- **Precondición:** ninguna (detalle público accesible sin sesión; posesión requiere sesión).
- **Pasos:** abrir la sección de cartas → aplicar filtros → paginar → abrir detalle de una carta.
- **Resultado esperado:** si el usuario posee la carta (`copies > 0`), se muestran estadísticas completas; si no, `stats: null`.
- **Casos negativos:** carta no poseída → estadísticas ocultas, no error (A1) · carta inexistente → 404 (A2).
