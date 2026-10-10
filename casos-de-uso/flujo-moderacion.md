# Flujo consolidado de moderación (CU21–CU23, CU26–CU28)

Los casos de uso de denuncias, moderación, reapertura y apelaciones no son independientes entre sí: comparten un único flujo de negocio con dos posibles caminos de reversión (reapertura directa por admin, o apelación revisada por otro moderador/admin).

```text
Usuario denuncia (CU21/CU22)
      |
      v
Reporte pendiente
      |
      v
Moderador revisa (CU23)
      |
      +---- descartar
      +---- advertir
      +---- eliminar contenido
      +---- banear usuario
      |
      v
Reporte resuelto
      |
      v
Auditoría + notificación
      |
      v
¿Sanción punitiva? (content_removed / comment_removed / user_banned)
      |
      +---- No (dismissed/warning) -> no hay nada que reabrir ni apelar
      |
      v
Sí:
      +---- Admin reabre directamente (CU26) ──────┐
      |                                             v
      +---- Afectado apela (CU27) ──► Apelación pending
                    │                                │
                    v                                │
          Otro moderador/admin revisa (CU28)         │
          (no quien resolvió, salvo único)            │
                    │                                │
          +---------+---------+                       │
          |                   |                       │
        rechaza            aprueba                     │
          |                   |                       │
          v                   v                       v
   No se modifica     Revertir sanción  <──────────────┘
   nada, se notifica  (restaurar obra/comentario
   al apelante         o desbanear) + reporte -> overturned
                        + notificar al afectado
                        + notificar al apelante
```
