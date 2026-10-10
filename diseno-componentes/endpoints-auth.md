# Endpoints — Auth

| Método | Endpoint | Uso | Auth |
|---|---|---|---|
| POST | `/auth/register` | Registrar usuario local | Público |
| POST | `/auth/login` | Login local | Público + rate limit |
| GET | `/auth/google` | Iniciar OAuth Google | Público |
| GET | `/auth/google/callback` | Callback Google | Público |
| POST | `/auth/verify-email` | Confirmar email | Público |
| POST | `/auth/resend-verification` | Reenviar challenge | Público + rate limit |
| POST | `/auth/forgot-password` | Solicitar enlace de recuperación de contraseña | Público + rate limit |
| POST | `/auth/reset-password` | Establecer una nueva contraseña con el enlace recibido | Público + rate limit |
| POST | `/auth/refresh` | Renovar sesión, si se utiliza refresh token | Auth/refresh + rate limit |
| POST | `/auth/logout` | Cerrar sesión | Auth |
| GET | `/auth/me` | Consultar usuario autenticado | Auth |
