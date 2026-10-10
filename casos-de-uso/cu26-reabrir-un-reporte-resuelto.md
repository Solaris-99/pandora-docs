# CU26 — Reabrir un reporte resuelto

**Actor principal:** Admin
**Precondición:** Existe un reporte con `status = resolved` y una `resolution` punitiva (`content_removed`, `comment_removed` o `user_banned`).
**Postcondición:** La sanción original queda revertida (contenido restaurado o usuario desbaneado) y el reporte pasa a `status = overturned`.
**Caso de prueba:** [CP18 — Reabrir un reporte resuelto](/casos-de-prueba/cp18-reabrir-un-reporte-resuelto.md)

## Flujo principal

1. El admin abre el listado de reportes resueltos.
2. Selecciona un reporte con sanción aplicada.
3. Confirma la reapertura.
4. El sistema revierte el efecto de la sanción original.
5. El sistema marca el reporte como `overturned`.
6. Se notifica al usuario afectado (obra/comentario restaurado, o cuenta desbaneada).

## Alternativo

- **A1:** El reporte ya fue reabierto o ya tiene una apelación aprobada en paralelo -> 409 (nada que revertir dos veces).
- **A2:** La `resolution` del reporte no es punitiva (`dismissed`/`warning`) -> 400, no hay nada que revertir.
- **A3:** El reporte no existe o no está resuelto -> 404.
- **A4:** Actor sin rol `admin` (un moderador sin `admin` no puede reabrir) -> 403.
