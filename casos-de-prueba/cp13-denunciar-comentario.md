# CP13 — Denunciar comentario (CU22)

**Caso de uso:** [CU22 — Denunciar comentario](/casos-de-uso/cu22-denunciar-comentario.md)

- **Pasos:** seleccionar motivo → opcionalmente agregar detalle → registrar `CommentReport`.
- **Resultado esperado:** reporte creado en `pending`.
- **Casos negativos:** comentario ya eliminado (A1) · motivo inválido → 400 (A2) · reporte duplicado (A3).
