# Resend (correo transaccional)

**Rol:** envío de los dos correos transaccionales del sistema — verificación de email (`RF02`/`RNF07`) y recuperación de contraseña. Encapsulado en `MailService` (`src/mail/mail.service.ts`) — Adapter, mismo patrón que `StorageService`.

**Configuración:** `RESEND_API_KEY`, `MAIL_FROM` (`src/config/mail.config.ts`). Sin `RESEND_API_KEY`, `MailService` opera en **modo LOCAL/LOGGER**: en vez de llamar a la API de Resend, imprime el contenido simulado del correo (destinatario, usuario, token, URL) en los logs — pensado para desarrollar sin depender de una cuenta real de Resend.

## Correos enviados

| Correo | Disparado por | Enlace incluido | Vigencia del token |
|---|---|---|---|
| Verificación de cuenta | `AuthService.register()`, `resendVerification()` | `${FRONTEND_URL}/verify-email?token=...` | 24 horas |
| Recuperación de contraseña | `AuthService.forgotPassword()` | `${FRONTEND_URL}/reset-password?token=...` | 1 hora |

Ambos enlaces apuntan al frontend (no directamente a un endpoint del backend): es el frontend quien lee el `token` de la URL y lo manda en el body de `POST /auth/verify-email` / `POST /auth/reset-password` — el correo en sí nunca llama al backend.

## Manejo de fallos

Un error de la API de Resend (o una excepción de red) se loguea pero **nunca** se propaga como excepción hacia quien llamó — `register()`/`resendVerification()`/`forgotPassword()` responden éxito igual, sea que el correo se haya enviado, haya fallado, o esté en modo LOCAL/LOGGER. Esto es consistente con el patrón anti-enumeración de `AuthService` ([clases de diseño, módulo `auth`](/clases-diseno/auth.md)): `forgotPassword()` responde el mismo mensaje neutro exista o no la cuenta, así que un fallo de Resend tampoco puede filtrar esa información por la vía de "la respuesta tardó más" o "devolvió otro código".

**Cuenta creada por Google, sin contraseña local:** `forgotPassword()` sobre esa cuenta no emite `PasswordResetToken` (no hay contraseña local que recuperar) pero igual responde el mensaje neutro — no hay forma de distinguir desde afuera "no existe la cuenta" de "existe pero es sólo-Google".
