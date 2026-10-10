# CU21 — Denunciar obra

**Actor principal:** Usuario autenticado.
**Caso de prueba:** [CP12 — Denunciar obra](/casos-de-prueba/cp12-denunciar-obra.md)

## Flujo principal

1. El usuario abre una obra.
2. Pulsa "Denunciar".
3. Selecciona un motivo predefinido.
4. Opcionalmente agrega comentario.
5. Se registra el reporte.
6. Se confirma al usuario.

## Alternativo

- **A1:** Motivo inválido -> rechazar.
- **A2:** Obra inexistente -> 404.
- **A3:** Obra ya eliminada -> informar que no requiere una nueva acción o permitir reporte según política.
- **A4:** Usuario intenta realizar múltiples reportes idénticos -> aplicar regla anti-spam/duplicados.
