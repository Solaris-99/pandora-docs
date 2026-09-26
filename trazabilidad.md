# Pandora — Documento de Trazabilidad

Este documento conecta [requerimientos](/requerimientos.md) con los [casos de uso](/casos-de-uso.md) que los realizan y los [componentes](/diseno-componentes.md) concretos que los implementan, y agrega una traza temporal de cuándo se construyó cada capacidad.

---

# 1. Trazabilidad de requerimientos funcionales

| RF | Descripción corta | Caso(s) de uso | Endpoint(s) / componente principal |
|---|---|---|---|
| RF01 | Registro | CU01 | `POST /auth/register` |
| RF02 | Verificación de email | CU01 | `POST /auth/verify-email`, `POST /auth/resend-verification` |
| RF03 | Login | CU02 | `POST /auth/login`, `GET /auth/google`, `POST /auth/google` |
| RF03b | Recuperación de contraseña | CU23 | `POST /auth/forgot-password`, `POST /auth/reset-password` |
| RF04 | Perfil | CU16 | `GET /users/:id`, `PATCH /users/me` |
| RF05 | Publicación de obra | CU03 | `POST /artworks` |
| RF06 | Gestión de obras | CU03, CU16 | `GET /users/me/artworks`, `PATCH /artworks/:id`, `DELETE /artworks/:id` |
| RF07 | Explorador de obras | CU04 | `GET /artworks` |
| RF08 | Detalle de obra | CU04 | `GET /artworks/:id` |
| RF09 | Comentarios | *(sin CU dedicado — flujo cubierto dentro de CU04/CU05)* | `GET/POST /artworks/:id/comments` |
| RF10 | Q2Q | CU05 | `PUT /artworks/:id/rating`, `GET /artworks/:id/ratings` |
| RF11 | Conversión a carta | CU06, CU22 | `ConversionSchedulerService.runConversionCycle`, `POST /conversion/run` |
| RF12 | Estadísticas de carta | CU06 | entidad `Card` |
| RF13 | Colección | CU07 | `GET /cards`, `GET /users/me/cards` |
| RF14 | Detalle de carta | CU07 | `GET /cards/:id` |
| RF15 | Paquetes | CU08 | `POST /users/me/packages/open` |
| RF16 | Regeneración de paquetes | CU08 | `GET /users/me/packages` |
| RF17 | Autobattler | CU09 | `GET /battle/opponent` |
| RF18 | Seguimiento | CU10 | `POST/DELETE /users/:id/follow` |
| RF19 | Favoritos | CU11 | `POST/DELETE /artworks/:id/favorite`, `POST/DELETE /cards/:id/favorite` |
| RF20 | Notificaciones | CU15 | `GET /notifications`, `PATCH /notifications/:id/read` |
| RF21 | Reportes de obras | CU12 | `POST /artworks/:id/reports` |
| RF22 | Reportes de comentarios | CU13 | `POST /comments/:id/reports` |
| RF23 | Moderación | CU14 | `GET /moderation/reports`, `POST /moderation/reports/:id/resolve` |
| RF24 | Estados de contenido | *(transversal — aplica a todas las entidades con `deleted`/`status`)* | soft delete + enums de estado en cada módulo |
| RF25 | Gestión de moderadores | CU17 | `POST/DELETE /roles/moderators/:userId` |
| RF26 | Reapertura de reportes | CU18 | `POST /moderation/reports/:id/reopen` |
| RF27 | Apelaciones | CU19, CU20 | `POST /artworks/:id/appeals`, `POST /comments/:id/appeals`, `POST /users/me/ban-appeal`, `GET/POST /moderation/appeals*` |
| RF28 | Carga directa de carta (Admin) | CU21 | `POST /artworks/admin-upload` |
| RF29 | Disparo manual de conversión (Admin) | CU22 | `POST /conversion/run` |

---

# 2. Trazabilidad de requerimientos no funcionales

| RNF | Descripción corta | Mecanismo / componente | Estado |
|---|---|---|---|
| RNF01 | Peso/formato de obras | `buildImageValidationPipe` (`pandora-clases-diseno.md` §19) | Implementado |
| RNF02 | Hash de contraseñas | `bcrypt` en `AuthService.register`/`resetPassword` | Implementado |
| RNF03 | Rate limit de login | `AppThrottlerGuard` + `@Throttle` (`pandora-diseno-componentes.md` §2.5) | Implementado (Fase 9) |
| RNF04 | Roles | `RolesGuard` | Implementado |
| RNF05 | Usabilidad | Responsabilidad de los clientes (web/Android), fuera del alcance del backend | N/A backend |
| RNF06 | Storage de imágenes | `StorageService` (Cloudinary) | Implementado |
| RNF07 | Verificación de email | `EmailVerificationToken` + `AuthService` | Implementado |
| RNF08 | Seguridad de endpoints | Combinación `JwtAuthGuard` + `RolesGuard` + `BannedUserGuard` + `VerifiedEmailGuard` + `AppThrottlerGuard` + sanitización + ownership checks en servicio | Implementado |
| RNF09 | Sanidad de campos | `@SanitizeText()` + límites de longitud en DTOs (`pandora-diseno-componentes.md` §3.1) | Implementado (Fase 9) |
| RNF10 | Manejo de errores | `AllExceptionsFilter` | Implementado |
| RNF11 | Integridad | Transacciones en `ConversionSchedulerService` y `PackagesService` | Implementado |
| RNF12 | Idempotencia de conversión | `UNIQUE(artwork_id)` en `cards` + claim atómico (`UPDATE ... WHERE conversion_status IN (...)`) | Implementado |
| RNF13 | Auditoría de moderación | `resolved_by`/`resolved_at` en reportes, `reviewed_by`/`reviewed_at` en apelaciones | Implementado |
| RNF14 | Observabilidad | `loggingMiddleware` + `AllExceptionsFilter` + logs puntuales en `MailService`/`StorageService`/`ConversionSchedulerService` | Implementado (Fase 9) |
| RNF15 | Paginación | `PaginationDto`/`PaginatedResponseDto` en galería, comentarios, notificaciones y reportes | Implementado |

Ítems de Fase 9/Hardening todavía **pendientes** a la fecha de este documento: métricas, revisión de queries e índices, revisión de seguridad.

---

# 3. Trazabilidad temporal (orden histórico de implementación)

## Fase 1 — Fundaciones

Proyecto NestJS, PostgreSQL, TypeORM, configuración por entorno, migraciones, estructura de módulos, manejo global de errores, validación global, autenticación JWT, roles.

## Fase 2 — Usuarios

Registro, verificación email, login, recuperación de contraseña, Google OAuth, perfiles, follows.

## Fase 3 — Obras

Tags, Cloudinary, creación, edición/eliminación, explorador, comentarios.

## Fase 4 — Q2Q

Ratings, emociones, período de una semana, restricciones de cierre.

## Fase 5 — Cartas

Algoritmo de conversión, `conversion_status`, Cron/Scheduler (rutina de conversión), cartas, colección.

## Fase 6 — Paquetes

Inventario de paquetes, regeneración de 1 minuto, apertura, distribución de cartas.

## Fase 7 — Juego

Endpoint de rival, simulador local, UI de batalla.

## Fase 8 — Moderación y notificaciones

Reportes de obras, reportes de comentarios, resolución, auditoría, notificaciones.

## Fase 9 — Hardening

- rate limiting (**implementado** — RNF03);
- sanitización (**implementado** — RNF09);
- tests (**ampliado**: cobertura agregada sobre `JwtStrategy`, `JwtAuthGuard`, utils puras de mapeo de respuesta — `toArtworkView`/`toCardView`/`toOpponentCardView` —, `buildImageValidationPipe`, `GoogleStrategy` y el middleware de logging; antes sin ningún test propio — ver `pandora-casos-de-prueba.md`);
- logs (**implementado** — RNF14);
- métricas (pendiente);
- revisión de queries e índices (pendiente);
- revisión de seguridad (pendiente).

## Extensión — Favoritos, borrado con cascada y rol de admin/apelaciones

Construidos después de la Fase 8, a pedido directo y fuera del orden de fases original: favoritos de obras/cartas, borrado lógico de obras con cascada a comentarios/valoraciones/carta, validación real de contenido de archivo, notificación apilable de votos (RF19–RF21 y RF25–RF27). El detalle de diseño e implementación de cada uno vive en `docs/mejoras-post-fase-8.md` y `docs/mejoras-admin-apelaciones.md`.

## Extensión — Herramientas de admin para demo (carga directa de carta y disparo manual de conversión)

También construidas fuera del orden de fases original, a pedido directo: carga directa de obra+carta por un admin sin pasar por el período de calificación (RF28, CU21) y disparo manual del ciclo de conversión que normalmente corre por CRON (RF29, CU22). Ninguna de las dos requirió cambios de modelo de datos — reutilizan las mismas tablas/columnas ya definidas (`artworks.conversion_status`, `cards`) y, en el caso del disparo manual, exactamente la misma rutina de conversión de la Fase 5. El contrato HTTP completo vive en `docs/shared/api-conventions.md` §18.8–18.9.

## Extensión — Recuperación de contraseña

Agregada después de la implementación inicial de la Fase 2, a pedido directo (RF03b, CU23). Reutiliza exactamente el mismo mecanismo que la verificación de email: token opaco de un solo uso, hasheado con SHA-256 antes de persistir, con expiración explícita — sólo que en tabla propia (`password_reset_tokens`) para no mezclar los dos propósitos, y con una vigencia más corta (1 hora en vez de 24) por ser un token más sensible. Sigue el mismo patrón anti-enumeración que el reenvío de verificación: responde con el mismo mensaje neutro exista o no la cuenta, y también para una cuenta creada exclusivamente vía Google (sin contraseña local, nada que recuperar). Efecto adicional al actualizar la contraseña: revoca todas las sesiones (`refresh_tokens`) activas de la cuenta. El contrato HTTP completo vive en `docs/shared/api-conventions.md` §7.5.

