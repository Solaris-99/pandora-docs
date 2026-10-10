# CP09 — Jugar autobattler (CU18)

**Caso de uso:** [CU18 — Jugar autobattler](/casos-de-uso/cu18-jugar-autobattler.md)

- **Precondición:** usuario autenticado, posee al menos una carta.
- **Pasos:** seleccionar carta propia → solicitar rival → backend selecciona una carta rival válida al azar → cliente simula la batalla localmente.
- **Resultado esperado:** ambas cartas (propia y rival) con estadísticas completas; ningún resultado persiste en el backend.
- **Casos negativos:** carta no poseída → 403/400 (A1) · carta inexistente → 404 (A2) · sin cartas rivales disponibles → error controlado (A3).
