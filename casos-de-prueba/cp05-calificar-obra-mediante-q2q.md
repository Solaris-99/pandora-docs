# CP05 — Calificar obra mediante Q2Q (CU14)

**Caso de uso:** [CU14 — Calificar obra mediante Q2Q](/casos-de-uso/cu14-calificar-obra-mediante-q2q.md)

- **Precondición:** obra dentro de su semana de calificación; usuario autenticado.
- **Pasos:** abrir detalle → backend confirma que la calificación sigue habilitada → elegir estrellas (1–5) y emoción → registrar o actualizar la valoración.
- **Resultado esperado:** valoración persistida (una por usuario/obra); si ya existía, se actualiza en vez de duplicarse.
- **Casos negativos:** período expirado (A1) · sin sesión (A2) · emoción inválida → 400 (A3) · estrellas fuera de rango → 400 (A4) · obra eliminada → rechazar (A6).
