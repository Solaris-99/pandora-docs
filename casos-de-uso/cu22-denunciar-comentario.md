# CU22 — Denunciar comentario

**Actor principal:** Usuario autenticado.

## Flujo principal

1. El usuario visualiza un comentario.
2. Pulsa "Denunciar".
3. Selecciona el motivo.
4. Opcionalmente añade detalle.
5. Se crea el reporte asociado al comentario.
6. Se notifica el registro exitoso.

## Alternativo

- **A1:** Comentario eliminado -> impedir o resolver automáticamente según estado.
- **A2:** Motivo inválido -> 400.
- **A3:** Reporte duplicado -> rechazar o consolidar.
