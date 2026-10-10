# CU23 — Administrar denuncias

**Actor principal:** Moderador.

## Flujo principal

1. El moderador abre el panel.
2. El sistema devuelve reportes pendientes, paginados y ordenados por prioridad/fecha.
3. El moderador abre un reporte.
4. Se muestra el contenido denunciado y contexto suficiente.
5. Selecciona una acción.
6. El sistema aplica la acción.
7. Marca el reporte como resuelto.
8. Registra auditoría.
9. Genera notificación cuando corresponda.

## Acciones

- descartar;
- advertir al autor;
- eliminar obra;
- eliminar comentario;
- banear usuario.

## Alternativo

- **A1:** El contenido ya fue eliminado -> permitir cerrar el reporte sin duplicar la acción.
- **A2:** Usuario ya baneado -> registrar el reporte como resuelto/afectado.
- **A3:** Moderador sin permisos -> 403.
- **A4:** Dos moderadores resuelven simultáneamente -> utilizar control de concurrencia/estado para evitar doble aplicación.

## Observaciones

La eliminación de obra aquí es una acción de moderación, distinta de la que ejecuta el propio autor en [CU11](cu11-eliminar-obra.md): puede aplicarse en cualquier momento, no sólo durante el período de calificación.
