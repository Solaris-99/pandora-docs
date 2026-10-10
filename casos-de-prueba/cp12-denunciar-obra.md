# CP12 — Denunciar obra (CU21)

**Caso de uso:** [CU21 — Denunciar obra](/casos-de-uso/cu21-denunciar-obra.md)

- **Pasos:** seleccionar motivo predefinido → opcionalmente agregar comentario → registrar el reporte.
- **Resultado esperado:** `ArtworkReport` en estado `pending`.
- **Casos negativos:** motivo inválido (A1) · obra inexistente → 404 (A2) · obra ya eliminada (A3) · reportes duplicados idénticos → regla anti-spam (A4).
