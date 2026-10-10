# CP18 — Reabrir un reporte resuelto (CU26)

**Caso de uso:** [CU26 — Reabrir un reporte resuelto](/casos-de-uso/cu26-reabrir-un-reporte-resuelto.md)

- **Precondición:** reporte `resolved` con `resolution` punitiva; actor `admin`.
- **Pasos:** abrir listado de resueltos → seleccionar reporte sancionado → confirmar reapertura → revertir la sanción → marcar `overturned` → notificar al afectado.
- **Resultado esperado:** contenido restaurado o usuario desbaneado; reporte en `overturned`.
- **Casos negativos:** ya reabierto o ya apelado y aprobado → 409 (A1) · resolución no punitiva → 400 (A2) · reporte inexistente/no resuelto → 404 (A3) · actor sin `admin` → 403 (A4).
