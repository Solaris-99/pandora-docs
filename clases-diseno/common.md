# Módulo `common` (transversal)

- **Guards:** `AppThrottlerGuard` (extiende `ThrottlerGuard`, castellaniza el mensaje 429), `BannedUserGuard`, `VerifiedEmailGuard`.
- **Middleware:** `loggingMiddleware` (ver [diseño de arquitectura, observabilidad](/diseno-arquitectura/observabilidad.md)).
- **Filtro global:** `AllExceptionsFilter` (normaliza toda excepción a `{statusCode, code, message, errors?, timestamp, path}`).
- **Pipe:** `buildImageValidationPipe(fileIsRequired)` (tipo real de archivo + tamaño máximo).
- **Decoradores:** `@SanitizeText()` (remueve HTML de campos de texto libre), `@ExposeId()` (normaliza bigint→number en respuestas), `@Public()`, `@CurrentUser()`, `@Roles(...)`.
