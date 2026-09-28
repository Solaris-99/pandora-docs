# Pandora — Diseño de Componentes

Este documento cubre los componentes de la capa API: endpoints REST propuestos, guards de autorización, validaciones, convenciones de respuesta y el contrato de estado de una carta. El contrato HTTP exacto (payloads, respuestas de ejemplo, códigos de error) vive en [convenciones de API](/api-conventions.md); este documento es su resumen a nivel de diseño.

---

# 1. Endpoints REST propuestos

Los endpoints siguientes son una propuesta derivada de los requerimientos y casos de uso (ver [requerimientos](/requerimientos.md), [casos de uso](/casos-de-uso.md)). Los nombres pueden adaptarse, pero se recomienda mantener una convención uniforme.

## 1.1 Auth

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

## 1.2 Users / Profiles

| Método | Endpoint | Uso |
|---|---|---|
| GET | `/users/:id` | Perfil público |
| PATCH | `/users/me` | Editar perfil propio |
| GET | `/users/me/followers` | Seguidores |
| GET | `/users/me/following` | Seguidos |
| POST | `/users/:id/follow` | Seguir |
| DELETE | `/users/:id/follow` | Dejar de seguir |

## 1.3 Artworks

| Método | Endpoint | Uso |
|---|---|---|
| POST | `/artworks` | Crear obra |
| GET | `/artworks` | Explorar y filtrar |
| GET | `/artworks/:id` | Detalle |
| PATCH | `/artworks/:id` | Editar obra propia |
| DELETE | `/artworks/:id` | Eliminación lógica |
| GET | `/users/me/artworks` | Obras propias |
| GET | `/artworks/:id/comments` | Comentarios |
| POST | `/artworks/:id/comments` | Crear comentario |

### Query params sugeridos para `GET /artworks`

```text
page / cursor
limit
search
from
until
authorId
tag
sort
qualificationOnly
converted
rarity
```

Para búsquedas que crezcan en complejidad, conviene encapsular parámetros en un DTO específico de filtros en lugar de construir SQL directamente desde query strings.

## 1.4 Ratings

| Método | Endpoint | Uso |
|---|---|---|
| PUT | `/artworks/:id/rating` | Crear o actualizar rating del usuario |
| DELETE | `/artworks/:id/rating` | Retirar rating, si la regla lo permite |
| GET | `/artworks/:id/ratings` | Resumen o datos de calificación |

No se debe exponer información que permita manipular el proceso de generación de estadísticas si ésta no corresponde a la interfaz pública.

## 1.5 Cards

| Método | Endpoint | Uso |
|---|---|---|
| GET | `/cards` | Explorar cartas |
| GET | `/cards/:id` | Detalle público |
| GET | `/users/me/cards` | Colección propia |
| GET | `/users/me/cards/:id` | Estado de posesión/detalle |
| POST | `/cards/:id/favorite` | Destacar carta |
| DELETE | `/cards/:id/favorite` | Quitar destacado |

## 1.6 Packages

| Método | Endpoint | Uso |
|---|---|---|
| GET | `/users/me/packages` | Consultar cantidad/estado |
| POST | `/users/me/packages/open` | Abrir paquete |

La operación de apertura debe ser atómica.

## 1.7 Battle

| Método | Endpoint | Uso |
|---|---|---|
| GET | `/battle/opponent` | Obtener una carta rival aleatoria |

No se requieren endpoints para guardar resultados. No se recomienda implementar `/battle/start`, `/battle/result` o `/battle/history` mientras la batalla siga siendo puramente cliente-side.

## 1.8 Favorites

| Método | Endpoint | Uso |
|---|---|---|
| POST | `/artworks/:id/favorite` | Destacar obra |
| DELETE | `/artworks/:id/favorite` | Quitar destacado |
| GET | `/users/me/favorites/artworks` | Obras destacadas |
| GET | `/users/me/favorites/cards` | Cartas destacadas |

## 1.9 Reports

| Método | Endpoint | Uso |
|---|---|---|
| POST | `/artworks/:id/reports` | Reportar obra |
| POST | `/comments/:id/reports` | Reportar comentario |
| GET | `/moderation/reports` | Listar reportes |
| GET | `/moderation/reports/:id` | Detalle |
| POST | `/moderation/reports/:id/resolve` | Resolver reporte |

## 1.10 Notifications

| Método | Endpoint | Uso |
|---|---|---|
| GET | `/notifications` | Listar notificaciones |
| GET | `/notifications/unread-count` | Contador no leídas |
| PATCH | `/notifications/:id/read` | Marcar una como leída |
| POST | `/notifications/read-all` | Marcar todas como leídas |

## 1.11 Tags

| Método | Endpoint | Uso |
|---|---|---|
| GET | `/tags` | Consultar catálogo (soporta `?search=`) |

La administración de tags puede restringirse a administrador si dejan de ser libres.

## 1.12 Administración y Apelaciones

| Método | Endpoint | Uso | Auth |
|---|---|---|---|
| POST | `/roles/moderators/:userId` | Otorgar rol `moderator` | Admin |
| DELETE | `/roles/moderators/:userId` | Revocar rol `moderator` | Admin |
| POST | `/moderation/reports/:id/reopen` | Reabrir un reporte resuelto (reversión directa) | Admin |
| POST | `/artworks/:id/appeals` | Apelar la eliminación de una obra propia | Auth |
| POST | `/comments/:id/appeals` | Apelar la eliminación de un comentario propio | Auth |
| POST | `/users/me/ban-appeal` | Apelar el propio baneo | Auth (permitido para cuenta baneada) |
| GET | `/users/me/appeals` | Consultar apelaciones propias | Auth (permitido para cuenta baneada) |
| GET | `/moderation/appeals` | Listar apelaciones pendientes (excluye las que el revisor no puede resolver) | Moderator/Admin |
| GET | `/moderation/appeals/:id` | Detalle de una apelación | Moderator/Admin |
| POST | `/moderation/appeals/:id/resolve` | Aprobar o rechazar una apelación | Moderator/Admin |
| POST | `/artworks/admin-upload` | Cargar directamente obra + carta ya convertida (CU21) | Admin |
| POST | `/conversion/run` | Disparar manualmente el ciclo de conversión (CU22), pensado para demos | Admin |

`POST /moderation/reports/:id/reopen` es el único endpoint de `/moderation/*` restringido a `admin` exclusivamente; el resto de `/moderation/*` acepta `moderator` o `admin` por igual. `POST /artworks/admin-upload` y `POST /conversion/run` también son exclusivos de `admin`.

---

# 2. Guards y autorización

La arquitectura NestJS separa autenticación de autorización mediante guards componibles.

## 2.1 `JwtAuthGuard`

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

## 2.2 `RolesGuard`

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

## 2.3 `OwnershipGuard` (verificación en servicio)

No todo se resuelve con roles. El backend debe verificar propiedad del recurso. Ejemplos:

- editar una obra -> el usuario debe ser su autor;
- eliminar una obra propia -> autor o moderador;
- destacar una obra -> debe ser propia;
- iniciar batalla con carta -> debe poseerla.

## 2.4 `VerifiedEmailGuard`

Se utiliza en funcionalidades que requieren cuenta plenamente habilitada. Ejemplo:

```text
POST /artworks
POST /artworks/:id/comments
PUT  /artworks/:id/rating
POST /artworks/:id/reports
```

## 2.5 Rate limiting

Implementado con `@nestjs/throttler` (Fase 9/Hardening), por IP, en memoria del proceso:

- Límite global por defecto (100/min) como red de seguridad general, vía guard global (`AppThrottlerGuard`).
- Override estricto (5/min, `@Throttle` por endpoint) en: login, registro, reenvío de verificación, recuperación de contraseña (`forgot-password`/`reset-password`) y `refresh`.
- Override moderado (10/min) en endpoints de escritura sensibles a spam: reportes, comentarios, valoraciones.

Detalle completo (valores exactos, comportamiento del error 429) en [convenciones de API](/api-conventions.md) §4.1.

## 2.6 `BannedUserGuard`

Un usuario baneado no debería poder utilizar funcionalidades de escritura aunque posea un JWT válido — pero sí debe poder **autenticarse**, porque necesita un token para apelar su propio baneo. Por eso el chequeo de estado `banned` no vive en `JwtStrategy`/`AuthService` (eso impediría obtener un token en absoluto), sino en este guard, aplicado junto a `JwtAuthGuard` en prácticamente todos los endpoints protegidos, **excepto**:

- `POST /users/me/ban-appeal` y `GET /users/me/appeals` (para poder apelar y consultar el resultado);
- todo `/notifications/*` (para poder leer la notificación que confirma el desenlace de la apelación).

El estado `suspended` es distinto: sigue bloqueado directamente en `JwtStrategy`/`AuthService`, sin ninguna excepción — la suspensión no forma parte del sistema de apelaciones.

---

# 3. Validaciones

## 3.1 Validación general

Toda entrada debe:

1. validarse en el backend aunque ya se valide en frontend;
2. normalizarse cuando corresponda;
3. limitar longitud y tamaño;
4. rechazar tipos inesperados;
5. **sanitizar contenido apropiadamente** — implementado: todo campo de texto libre (título/descripción de obra, tags, comentarios, bio, `comment`/`detail` de reportes y apelaciones) remueve marcado HTML antes de persistir, defensa contra XSS almacenado independiente del cliente que lo renderice (detalle en [convenciones de API](/api-conventions.md) §4.2);
6. evitar concatenación directa en consultas SQL — garantizado en todo el proyecto vía TypeORM (queries parametrizadas / query builder, sin interpolación directa de strings).

## 3.2 Usuario

```text
username: 1–30 caracteres, sólo letras/números/guión bajo, único
email: formato válido, máximo 50 según RNF adoptado, único
password: política de complejidad definida por implementación (mínimo 6, máximo 50)
```

La contraseña debe almacenarse siempre como hash.

## 3.3 Obras

```text
title: requerido, <= 50
description: opcional, <= 300
tags: lista acotada y normalizada
image: <= 25 MB, PNG/JPG/JPEG/WEBP
```

## 3.4 Comentarios

```text
comment: requerido después de trim, <= 300
```

Deben aplicarse medidas anti-spam y rate limiting (ver 2.5).

## 3.5 Ratings

```text
stars: integer 1..5
emotion: valor perteneciente al enum
```

Además debe verificarse que la obra esté dentro de su período activo.

## 3.6 Reportes

```text
reason: enum
comment: <= 300
```

No aceptar resolución enviada desde un cliente normal. Las resoluciones son acciones administrativas.

## 3.7 DTOs

Los DTOs representan contratos explícitos de entrada. Catálogo (no exhaustivo) por módulo:

```text
RegisterDto, LoginDto, VerifyEmailDto, ResendVerificationDto,
ForgotPasswordDto, ResetPasswordDto, RefreshTokenDto, GoogleLoginDto
UpdateProfileDto
CreateArtworkDto, UpdateArtworkDto, AdminUploadCardDto, QueryArtworksDto
CreateCommentDto
RateArtworkDto
QueryCardsDto
QueryTagsDto
CreateReportDto, ResolveReportDto, ReopenReportDto
CreateAppealDto, ResolveAppealDto, QueryAppealsDto
```

No se reutilizan automáticamente entidades TypeORM como DTOs de entrada. Esto ayuda a:

- controlar qué campos puede modificar el cliente;
- aplicar validaciones;
- evitar mass assignment;
- separar modelo de persistencia y contrato de API.

---

# 4. Convenciones de API

## 4.1 Respuestas de éxito

Cada endpoint devuelve directamente el recurso (o una página de recursos) serializado según su DTO de salida — ver el contrato completo en [convenciones de API](/api-conventions.md).

## 4.2 Errores

```json
{
  "statusCode": 400,
  "code": "VALIDATION_ERROR",
  "message": "La obra no cumple los requisitos de validación.",
  "errors": [{ "field": "title", "message": "..." }],
  "timestamp": "2026-...",
  "path": "/api/v1/artworks"
}
```

No devolver stack traces ni detalles internos al cliente. Normalizado centralmente por `AllExceptionsFilter` (ver [clases de diseño](/clases-diseno.md) §19).

## 4.3 Paginación

`page`/`limit` para listados de la API (galería, comentarios, notificaciones, reportes — RNF15), con respuesta `{items, total, page, limit, totalPages}`.

---

# 5. Estado de una carta en el cliente

La API de colección permite distinguir como mínimo:

```text
owned = true/false
copies = integer
stats = objeto completo | null
```

Esto permite que el frontend renderice:

- carta completa si es poseída;
- imagen/obra visible con estadísticas desconocidas (`stats: null`) si no es poseída.

La ocultación es una regla de presentación que además protege el backend: la API no envía información privada de forma innecesaria cuando el usuario no posee la carta (ver [clases de dominio](/clases-dominio.md) §1.7–1.8 para la regla de negocio, y [clases de diseño](/clases-diseno.md) §8 para `toCardView`).

