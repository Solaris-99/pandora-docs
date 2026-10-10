# CP10 — Seguir usuario (CU19)

**Caso de uso:** [CU19 — Seguir usuario](/casos-de-uso/cu19-seguir-usuario.md)

- **Pasos:** abrir perfil ajeno → pulsar "Seguir" → se crea el `Follow` → se notifica al usuario seguido.
- **Resultado esperado:** relación creada; notificación `user_follow` generada.
- **Casos negativos:** relación ya existente → idempotente, no error (A1) · intento de seguirse a sí mismo → rechazar (A2) · usuario objetivo inexistente/baneado → rechazar (A3).
