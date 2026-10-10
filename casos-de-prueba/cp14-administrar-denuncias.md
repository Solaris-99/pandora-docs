# CP14 — Administrar denuncias (CU23)

**Caso de uso:** [CU23 — Administrar denuncias](/casos-de-uso/cu23-administrar-denuncias.md)

- **Precondición:** actor con rol `moderator` o `admin`.
- **Pasos:** abrir panel de reportes pendientes → abrir un reporte → elegir acción (descartar/advertir/eliminar obra/eliminar comentario/banear) → aplicar → marcar `resolved` → auditar → notificar.
- **Resultado esperado:** reporte `resolved` con `resolved_by`/`resolved_at`/`resolution` completos; sanción aplicada; notificación al afectado cuando corresponde.
- **Casos negativos:** contenido ya eliminado → cerrar sin duplicar acción (A1) · usuario ya baneado (A2) · sin permisos → 403 (A3) · dos moderadores resolviendo simultáneamente → control de concurrencia (A4).
