# CP23 — Recuperar contraseña olvidada (CU04)

**Caso de uso:** [CU04 — Recuperar contraseña olvidada](/casos-de-uso/cu04-recuperar-contrasena-olvidada.md)

- **Precondición:** ninguna del lado del actor (invitado); la cuenta puede existir o no.
- **Pasos:** solicitar recuperación con un email → respuesta neutra siempre → si la cuenta existe y tiene contraseña local, se emite un token de un solo uso y se envía por correo → el usuario abre el enlace → envía la nueva contraseña → backend valida el token, actualiza la contraseña y revoca todos los `refresh_tokens` activos → el usuario inicia sesión con la contraseña nueva.
- **Resultado esperado:** contraseña actualizada; sesiones previas invalidadas; no se inicia sesión automáticamente.
- **Casos negativos:** email inexistente → mismo mensaje neutro, nada se envía (A1) · cuenta exclusivamente de Google → mismo mensaje neutro, nada se envía (A2) · token inválido/inexistente/ya usado → 400 (A3) · token expirado (> 1 hora) → 400 (A4) · contraseña nueva fuera de política → 400 (A5).
- **Verificación en vivo realizada:** confirmado por HTTP real end-to-end — token bogus rechazado (400), token válido aceptado (200), reintento del mismo token rechazado (400, ya usado), login con contraseña vieja rechazado (401), login con contraseña nueva exitoso (200), y confirmado en base de datos que el `refresh_token` emitido antes del reset quedó `revoked_at` mientras el emitido después del login post-reset permanece activo.
