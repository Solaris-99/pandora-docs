# CU28 — Revisar una apelación

**Actor principal:** Moderador o Admin
**Precondición:** Existe una apelación en estado `pending`; el revisor no resolvió el reporte original que la apelación referencia, salvo que sea el único miembro de moderación (`moderator` + `admin`) en el sistema.
**Postcondición:** La apelación queda `approved` o `rejected`; si fue aprobada, la sanción original queda revertida igual que en [CU26](cu26-reabrir-un-reporte-resuelto.md).

## Flujo principal

1. El moderador abre el listado de apelaciones pendientes (el sistema excluye las que él mismo originó al resolver el reporte, salvo la excepción de moderador único).
2. Abre el detalle de una apelación: motivo, texto opcional, y el reporte/sanción que referencia.
3. Decide aprobar o rechazar.
4. Si aprueba: el sistema revierte la sanción original (mismo mecanismo que [CU26](cu26-reabrir-un-reporte-resuelto.md)) y notifica al afectado.
5. Si rechaza: no se modifica el contenido ni el reporte.
6. En ambos casos se notifica el resultado al apelante.

## Alternativo

- **A1:** El revisor resolvió el reporte original y hay más de un miembro de personal de moderación -> 403, debe revisarla otro moderador/admin.
- **A2:** La apelación ya fue resuelta por otro moderador -> 409.
- **A3:** La aprobación compite con una reapertura directa ([CU26](cu26-reabrir-un-reporte-resuelto.md)) del mismo reporte -> no es un error para la apelación: se considera igualmente aprobada, ya que el resultado deseado ya ocurrió.
- **A4:** Actor sin rol `moderator`/`admin` -> 403.
