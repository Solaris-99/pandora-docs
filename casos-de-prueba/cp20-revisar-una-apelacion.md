# CP20 — Revisar una apelación (CU28)

**Caso de uso:** [CU28 — Revisar una apelación](/casos-de-uso/cu28-revisar-una-apelacion.md)

- **Precondición:** apelación `pending`; revisor distinto de quien resolvió el reporte original (salvo moderador único).
- **Pasos:** abrir listado (excluye conflictos de revisor) → abrir detalle → aprobar o rechazar → si aprueba, revertir la sanción (mismo mecanismo que [CU26](/casos-de-uso/cu26-reabrir-un-reporte-resuelto.md)) → notificar al apelante en ambos casos.
- **Resultado esperado:** apelación `approved`/`rejected`; si `approved`, mismo efecto que [CU26](/casos-de-uso/cu26-reabrir-un-reporte-resuelto.md).
- **Casos negativos:** revisor en conflicto con más de un moderador disponible → 403 (A1) · ya resuelta por otro → 409 (A2) · aprobación compite con reapertura directa del mismo reporte → se considera igualmente aprobada, no es error (A3) · sin rol de moderación → 403 (A4).
