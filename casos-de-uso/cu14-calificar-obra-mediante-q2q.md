# CU14 — Calificar obra mediante Q2Q

**Actor principal:** Usuario autenticado
**Precondición:** La obra está dentro de su semana de calificación.
**Caso de prueba:** [CP05 — Calificar obra mediante Q2Q](/casos-de-prueba/cp05-calificar-obra-mediante-q2q.md)

## Flujo principal

1. El usuario abre el detalle de una obra.
2. El backend informa si la calificación sigue habilitada.
3. El usuario selecciona entre 1 y 5 estrellas.
4. Selecciona una emoción permitida.
5. Se registra o actualiza la valoración.
6. Se confirma la acción.

## Alternativo

- **A1:** Período expirado -> no se acepta valoración; comentarios ([CU13](cu13-comentar-obra.md)) siguen disponibles.
- **A2:** Usuario no autenticado -> se requiere login.
- **A3:** Emoción inválida -> 400.
- **A4:** Estrellas fuera de rango -> 400.
- **A5:** Usuario intenta duplicar valoración -> actualizar existente o rechazar según la política elegida. Se recomienda actualización.
- **A6:** Obra eliminada/bloqueada -> no permitir valoración.

## Observaciones

Publicar un comentario sobre la obra es un caso de uso independiente ([CU13](cu13-comentar-obra.md)); ya no es un paso opcional de este flujo.
