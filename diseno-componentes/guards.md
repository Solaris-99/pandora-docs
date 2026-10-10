# Guards y autorización

La arquitectura NestJS separa autenticación de autorización mediante guards componibles.

## `JwtAuthGuard`

Protege rutas que requieren sesión autenticada. Ejemplo conceptual de rutas protegidas:

```text
POST /artworks
PUT /artworks/:id/rating
POST /artworks/:id/comments
POST /artworks/:id/reports
POST /users/:id/follow
POST /users/me/packages/open
GET  /notifications
```

## `RolesGuard`

Comprueba que el usuario tenga un rol requerido. Ejemplo:

```text
GET  /moderation/reports
POST /moderation/reports/:id/resolve
GET  /moderation/appeals
POST /moderation/appeals/:id/resolve
```

requieren `moderator` o `admin`. En cambio:

```text
POST   /roles/moderators/:userId
DELETE /roles/moderators/:userId
POST   /moderation/reports/:id/reopen
POST   /artworks/admin-upload
POST   /conversion/run
```

requieren exclusivamente `admin`.

## `OwnershipGuard` (verificación en servicio)

No todo se resuelve con roles. El backend debe verificar propiedad del recurso. Ejemplos:

- editar una obra -> el usuario debe ser su autor;
- eliminar una obra propia -> autor o moderador;
- destacar una obra -> debe ser propia;
- iniciar batalla con carta -> debe poseerla.

## `VerifiedEmailGuard`

Se utiliza en funcionalidades que requieren cuenta plenamente habilitada. Ejemplo:

```text
POST /artworks
POST /artworks/:id/comments
PUT  /artworks/:id/rating
POST /artworks/:id/reports
```

## Rate limiting

Implementado con `@nestjs/throttler` (Fase 9/Hardening), por IP, en memoria del proceso:

- Límite global por defecto (100/min) como red de seguridad general, vía guard global (`AppThrottlerGuard`).
- Override estricto (5/min, `@Throttle` por endpoint) en: login, registro, reenvío de verificación, recuperación de contraseña (`forgot-password`/`reset-password`) y `refresh`.
- Override moderado (10/min) en endpoints de escritura sensibles a spam: reportes, comentarios, valoraciones.

Detalle completo (valores exactos, comportamiento del error 429) en [convenciones de API](/api-conventions.md) §4.1.

## `BannedUserGuard`

Un usuario baneado no debería poder utilizar funcionalidades de escritura aunque posea un JWT válido — pero sí debe poder **autenticarse**, porque necesita un token para apelar su propio baneo. Por eso el chequeo de estado `banned` no vive en `JwtStrategy`/`AuthService` (eso impediría obtener un token en absoluto), sino en este guard, aplicado junto a `JwtAuthGuard` en prácticamente todos los endpoints protegidos, **excepto**:

- `POST /users/me/ban-appeal` y `GET /users/me/appeals` (para poder apelar y consultar el resultado);
- todo `/notifications/*` (para poder leer la notificación que confirma el desenlace de la apelación).

El estado `suspended` es distinto: sigue bloqueado directamente en `JwtStrategy`/`AuthService`, sin ninguna excepción — la suspensión no forma parte del sistema de apelaciones.
