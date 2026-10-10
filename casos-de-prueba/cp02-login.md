# CP02 — Login (CU03)

**Caso de uso:** [CU03 — Login](/casos-de-uso/cu03-login.md)

- **Precondición:** cuenta registrada.
- **Pasos:** enviar email/contraseña → backend aplica rate limit → busca el usuario → verifica credenciales → verifica estado de la cuenta → genera JWT.
- **Resultado esperado:** par de tokens (`accessToken`/`refreshToken`) y datos del usuario.
- **Casos negativos:** credenciales inválidas → 401 genérico (A1) · demasiados intentos → 429 (A2, verificado en vivo: 6ª petición en un minuto bloqueada) · cuenta baneada → sigue permitiendo login, pero bloquea escritura vía `BannedUserGuard` (A3, ver regla 6 de [requerimientos §5.11](/requerimientos/reglas-11-apelaciones-y-administracion.md)) · cuenta sin verificar → mensaje específico (A4) · error de Google → mensaje genérico (A5).
