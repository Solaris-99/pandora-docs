# CU11 — Eliminar obra

**Actor principal:** Usuario autenticado
**Precondición:** El usuario es autor de la obra; la obra está dentro de su período de calificación (todavía no fue convertida en carta).
**Postcondición:** La obra queda eliminada (eliminación lógica).

## Flujo principal

1. El usuario entra a una obra propia.
2. Pulsa "Eliminar".
3. Confirma la acción.
4. El backend verifica que sea el autor y que la obra todavía esté en período de calificación.
5. Se marca la obra como eliminada (eliminación lógica).

## Alternativo

- **A1:** La obra ya fue convertida -> rechazar, no se puede eliminar una obra que ya es carta.
- **A2:** El usuario no es el autor -> 403.
- **A3:** La obra ya estaba eliminada -> idempotente o 404, según política.

## Observaciones

Distinto de la eliminación por moderación ([CU23](cu23-administrar-denuncias.md)): esta la ejecuta el propio autor y sólo está disponible durante el período de calificación; la de moderación la ejecuta un moderador y puede aplicarse en cualquier momento.
