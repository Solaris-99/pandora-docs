# CU13 — Comentar obra

**Actor principal:** Usuario autenticado
**Precondición:** La obra existe y no fue eliminada.
**Postcondición:** El comentario queda registrado y visible en la obra.

## Flujo principal

1. El usuario abre el detalle de una obra.
2. Escribe un comentario.
3. Lo envía.
4. El backend valida y sanitiza el contenido.
5. Se registra el comentario.
6. Se genera una notificación al autor de la obra, si corresponde.

## Alternativo

- **A1:** Comentario vacío (después de trim) o excede longitud -> 400.
- **A2:** Obra inexistente o eliminada -> 404.
- **A3:** Usuario baneado -> 403.

## Observaciones

Disponible independientemente de si el período de calificación de la obra ([CU14](cu14-calificar-obra-mediante-q2q.md)) sigue activo o ya terminó.
