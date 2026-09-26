# Pandora — Clases de Diseño y Diagrama

Este documento describe las clases **tal como se realizan en el backend NestJS**: entidades TypeORM (atributos con tipo y relaciones de persistencia), servicios de dominio (con sus operaciones principales) y DTOs de entrada/salida, organizados por módulo.

# 1. Convención general

Cada módulo de NestJS agrupa tres tipos de clase de diseño:

- **Entidad** (`*.entity.ts`): clase TypeORM que mapea una tabla; define atributos tipados y relaciones (`@OneToMany`, `@ManyToOne`, etc.).
- **Servicio** (`*.service.ts`): clase inyectable que concentra la lógica de dominio y orquesta repositorios; es la única capa que accede a los repositorios directamente. Expone las operaciones que el/los controlador(es) del módulo invocan.
- **DTO** (`dto/*.dto.ts`): clases de contrato — de entrada (validadas con `class-validator`) o de salida (serializadas con `class-transformer`/`@Expose()`). Nunca se reutiliza una entidad como DTO de entrada.

Controladores y guards concretos (qué expone cada endpoint) viven en `pandora-diseno-componentes.md`; acá el foco es la clase y sus operaciones.

---

# 2. Módulo `auth`

## Entidades

- **EmailVerificationToken**: `id`, `user_id`, `token_hash: string`, `expires_at: Date`, `used_at: Date | null`, `created_at`. Relación `ManyToOne` a `User`.
- **PasswordResetToken**: misma forma que `EmailVerificationToken` (tabla separada).
- **RefreshToken**: `id`, `user_id`, `token_hash: string`, `expires_at: Date`, `revoked_at: Date | null`, `created_at`. Relación `ManyToOne` a `User`.
- **OAuthAccount**: `id`, `user_id`, `provider: string`, `provider_user_id: string`. Relación `ManyToOne` a `User`.

## Servicio: `AuthService`

| Operación | Responsabilidad |
|---|---|
| `register(dto)` | Valida unicidad, hashea contraseña (bcrypt), crea `User`, emite token de verificación y correo, devuelve par de tokens. |
| `login(dto)` | Verifica credenciales (`bcrypt.compare`), bloquea cuentas `suspended`, devuelve par de tokens. |
| `verifyEmail(token)` | Consume un `EmailVerificationToken`, activa la cuenta, devuelve par de tokens (auto-login). |
| `resendVerification(email)` | Invalida tokens previos, emite uno nuevo — respuesta neutra si el email no existe (anti-enumeración). |
| `forgotPassword(email)` | Igual patrón anti-enumeración; emite `PasswordResetToken` sólo si la cuenta tiene contraseña local. |
| `resetPassword(token, newPassword)` | Consume el token, actualiza `password_hash`, revoca todos los `RefreshToken` activos del usuario. |
| `handleGoogleAuth(googleUser)` / `loginWithGoogleIdToken(idToken)` | Vincula o crea cuenta a partir de un perfil de Google. |
| `refreshTokens(rawToken)` | Rota el refresh token (revoca el usado, emite uno nuevo). |
| `logout(rawToken)` | Revoca un refresh token puntual. |
| `getProfile(userId)` | Devuelve el perfil del usuario autenticado. |

Métodos privados clave: `hashToken()` (SHA-256, para tokens opacos), `issueRefreshToken()`.

## Estrategias y guards

- **JwtStrategy.validate(payload)**: resuelve el usuario desde el JWT, bloquea `suspended`, permite `banned` (necesita autenticarse para apelar).
- **GoogleStrategy.validate(...)**: mapea el perfil de Google a la forma interna esperada por `handleGoogleAuth`.
- **JwtAuthGuard**: respeta `@Public()`; delega en Passport; normaliza el error 401.
- **GoogleOAuthGuard**: delega en Passport (`AuthGuard('google')`).

## DTOs

`RegisterDto`, `LoginDto`, `VerifyEmailDto`, `ResendVerificationDto`, `ForgotPasswordDto`, `ResetPasswordDto`, `RefreshTokenDto`, `GoogleLoginDto` (entrada) · `TokenPairResponseDto`, `VerifyEmailResponseDto`, `MeResponseDto` (salida).

---

# 3. Módulo `users`

## Entidades

- **User**: `id`, `username: string`, `email: string`, `password_hash: string | null`, `email_verified: boolean`, `status: UserStatus`, `bio: string | null`, `avatar_url: string | null`, `roles: Role[]` (`ManyToMany`), `created_at`, `updated_at`.
- **Follow**: `follower_id`, `followed_id`, `created_at`.

## Servicio: `UsersService`

| Operación | Responsabilidad |
|---|---|
| `findByEmail` / `findByUsername` / `findById` | Búsquedas base (con roles cargados). |
| `create` / `save` / `updateStatus` | Persistencia de alta y cambios de estado. |
| `getPublicProfile(targetId, currentId?)` | Perfil público + contadores de seguidores/seguidos + `isFollowing` si hay sesión. |
| `updateProfile(userId, dto)` | Edita username/bio/avatar propios, valida unicidad de username. |
| `followUser` / `unfollowUser` | Alta/baja idempotente de `Follow`, dispara notificación `user_follow`. |
| `getFollowers` / `getFollowing` | Listados paginados. |
| `count()` | Total de usuarios registrados — usado como divisor en la fórmula de rareza (ver `pandora-requerimientos.md` §5.7). |

## DTOs

`UpdateProfileDto` (entrada) · `PublicProfileResponseDto`, `UpdateProfileResponseDto`, `FollowListItemDto` (salida).

---

# 4. Módulo `roles`

## Entidad: `Role`

`id`, `name: RoleName` (`user`/`moderator`/`admin`), `description: string | null`.

## Servicio: `RolesService`

`findByName`, `ensureDefaultRoles()` (auto-repara el catálogo de roles al arrancar), otorgar/revocar `moderator` sobre un usuario objetivo.

## Guard: `RolesGuard`

Lee metadata `@Roles(...)` (por método o por clase, con `getAllAndOverride`) y compara contra `request.user.roles`.

---

# 5. Módulo `artworks`

## Entidad: `Artwork`

`id`, `title: string`, `description: string | null`, `author_id`, `author: User`, `image_original_url` / `image_medium_url` / `image_thumbnail_url: string`, `image_public_id: string`, `conversion_request: boolean`, `conversion_status: ConversionStatus`, `qualification_started_at: Date`, `deleted: boolean`, `tags: Tag[]` (`ManyToMany`), `created_at`, `updated_at`.

## Servicio: `ArtworksService`

| Operación | Responsabilidad |
|---|---|
| `create(authorId, dto, file)` | Sube imagen a Cloudinary, crea la obra en `pending`, dispara notificación a seguidores. |
| `createByAdmin(adminId, dto, file)` | CU21: crea obra + carta directamente, `conversion_status = converted`, sin notificar seguidores. |
| `findOne(id, currentUserId?)` | Detalle público + `isFavorited` si hay sesión. |
| `findAll(query)` / `findMine(authorId, pagination)` | Explorador con filtros (búsqueda, autor, fechas, tag, rareza, calificación) y listado propio. |
| `update` / `remove` | Edición por el autor; eliminación lógica por autor o moderador. |
| `moderationRemove(id)` / `restore(id)` | Eliminación/restauración por moderación (sin validar ownership; el guard ya restringe el acceso), con cascada lógica a comentarios/valoraciones/carta/favoritos. |

## DTOs

`CreateArtworkDto`, `UpdateArtworkDto`, `AdminUploadCardDto`, `QueryArtworksDto` (entrada) · `ArtworkResponseDto` (con `ArtworkImageDto`, `QualificationDto` anidados) (salida).

---

# 6. Módulo `ratings`

## Entidad: `Rating`

`id`, `artwork_id`, `voter_id`, `stars: number (1–5)`, `emotion: Emotion`, `created_at`, `updated_at`.

## Servicio: `RatingsService`

`rate(artworkId, userId, dto)` (crea o actualiza), `unrate`, `getSummary(artworkId, currentUserId?)`, `getConversionAggregate(artworkId)` (promedio de estrellas, total de votos y emoción dominante — insumo de `ConversionService`), `removeAllForArtwork` (cascada), `notifyOwnerOfVote` (notificación apilable `artwork_vote_summary`).

---

# 7. Módulo `comments`

## Entidad: `Comment` (`artwork_comments`)

`id`, `author_id`, `artwork_id`, `comment: string`, `deleted: boolean`, `created_at`, `updated_at`.

## Servicio: `CommentsService`

`create(artworkId, authorId, dto)`, `findAllForArtwork(artworkId, pagination)`, `removeAllForArtwork` (cascada).

---

# 8. Módulo `cards` (+ `collections`)

## Entidades

- **Card**: `id`, `artwork_id` (único, `OneToOne` con `Artwork`), `attack` / `defense` / `hp` / `speed: number`, `rarity: Rarity`, `deleted: boolean`, `created_at`, `updated_at`.
- **UserCard** (módulo `collections`): `id`, `user_id`, `card_id`, `copies: number`.

## Servicio: `CardsService`

| Operación | Responsabilidad |
|---|---|
| `findAll(query, currentUserId?)` | Explorador de cartas, con filtro `owned` (requiere sesión). |
| `findOne(id, currentUserId?)` | Detalle; oculta `stats` si `copies = 0` o no hay sesión. |
| `ensureRarityCatalogForAuthor(authorId)` | Garantiza que un autor tenga al menos una carta de cada rareza (autoreparación del catálogo mínimo). |
| `createForArtwork(artworkId, rarity, statsOverride?)` | CU21: crea la carta de una obra ya convertida; completa estadísticas faltantes vía `computeStatsForRarity`. |
| `removeByArtwork(artworkId)` | Cascada lógica al eliminar la obra de origen. |

## Servicio: `CardCatalogSeederService`

`onApplicationBootstrap()`: en cada arranque, delega en `ensureRarityCatalogForAuthor` bajo la cuenta de sistema `pandora`.

## DTOs

`QueryCardsDto` (entrada) · `CardResponseDto` (con `CardArtworkSummaryDto`, `CardStatsDto` anidados) (salida).

---

# 9. Módulo `packages`

## Entidad: `CardPack`

`id`, `user_id` (único), `amount: number (0–10)`, `last_regeneration_at: Date | null`, `updated_at`.

## Servicio: `PackagesService`

`getInventory(userId)` (aplica el cálculo de regeneración de `pandora-diseno-arquitectura.md` §6.2 antes de responder), `openPackage(userId)` (consumo atómico + sorteo de 5 cartas por rareza ponderada + alta/incremento en `UserCard`).

Utilidades puras: `rarity-roll.util` (sorteo ponderado 64/20/10/5/1), `packages-regen.util` (cálculo de minutos/paquetes regenerados).

---

# 10. Módulo `battle`

## Servicio: `BattleService`

`getOpponent(ownedCardId)`: valida ownership de la carta propia, selecciona una carta rival válida al azar entre todas las cartas existentes, devuelve ambas con estadísticas completas (sin ocultamiento — regla de negocio 5.8/1.7 de `pandora-clases-dominio.md`).

Utilidad pura: `toOpponentCardView` (mapeo de `Card` a la forma de respuesta del rival, siempre con `stats` visible).

---

# 11. Módulo `follows` / `favorites`

- El modelo `Follow` vive físicamente en el módulo `users` (ver sección 3); no hay un módulo `follows` separado con servicio propio.
- **Entidades** `FavoriteArtwork`, `FavoriteCard`: `user_id`, `artwork_id`/`card_id`, `created_at`.
- **Servicio `FavoritesService`**: `favoriteArtwork` / `unfavoriteArtwork` / `favoriteCard` / `unfavoriteCard` (idempotentes), `isArtworkFavorited` / `isCardFavorited` (usados por `ArtworksService`/`CardsService` al armar el detalle), `removeAllFavoritesForArtwork` / `removeAllFavoritesForCard` (cascada).

---

# 12. Módulo `notifications`

## Entidad: `Notification`

`id`, `user_id`, `type: NotificationType`, `title: string`, `content: string`, `is_read: boolean`, `reference_id: bigint | null`, `metadata: jsonb | null`, `created_at`.

`reference_id`/`metadata` son de propósito general: se reutilizan tanto para apilar votos (`{voters: [...]}`) como para referenciar el recurso afectado por una acción de moderación (`{targetType, targetId}`).

## Servicio: `NotificationsService`

`create(userId, type, title, content, data?)`, `findAll(userId, pagination)`, `markAsRead(id, userId)`, `markAllAsRead(userId)`, `unreadCount(userId)`. La reconstrucción de `data` en la vista de respuesta distingue por tipo de contenido de `metadata` (type guard sobre `targetType`).

---

# 13. Módulo `reports`

## Entidades

- **ArtworkReport**: `id`, `artwork_id`, `reporter_id`, `reason: ReportReason`, `comment: string | null`, `status: ReportStatus`, `resolved_by: bigint | null`, `resolution: ReportResolution | null`, `created_at`, `resolved_at`.
- **CommentReport**: misma forma, referenciando `comment_id`.

## Servicio: `ReportsService`

`reportArtwork(reporterId, artworkId, dto)`, `reportComment(reporterId, commentId, dto)` — validan duplicados pendientes antes de crear.

---

# 14. Módulo `moderation`

## Entidad: `Appeal`

`id`, `report_target_type: 'artwork' | 'comment'`, `report_id`, `appellant_id`, `reason: AppealReason`, `detail: string | null`, `status: AppealStatus`, `reviewed_by: bigint | null`, `reviewed_at`, `created_at`.

## Servicio: `ModerationService`

`listReports`, `getReportDetail`, `resolveReport(moderatorId, targetType, id, action)` (aplica la sanción — advertencia/eliminación/baneo/descarte — y notifica), `reopenReport(targetType, id)` (CU18, sólo admin), `listAppeals`, `getAppealDetail`, `resolveAppeal(reviewerId, id, action)` (CU20). Método privado `notify(...)` centraliza el envío de notificaciones de moderación con `data: {targetType, targetId}`.

## Servicio: `AppealsService`

`appealArtworkRemoval(userId, artworkId, dto)`, `appealCommentRemoval(userId, commentId, dto)`, y el flujo de apelación de baneo (`POST /users/me/ban-appeal`, expuesto desde `users`).

---

# 15. Módulo `tags`

## Entidad: `Tag`

`id`, `name: string` (único, normalizado a minúsculas).

## Servicio: `TagsService`

`findAllView(query?)` (filtro `search` server-side vía `ILIKE`), `findOrCreateByNames(names)` (normaliza, deduplica y crea dinámicamente los tags nuevos).

---

# 16. Módulo `storage`

## Servicio: `StorageService`

`uploadArtworkImage(file)` → `{publicId, originalUrl, mediumUrl, thumbnailUrl}` (sube a Cloudinary, genera transformaciones). `deleteArtworkImage(publicId)` (best-effort, no bloquea el flujo si Cloudinary falla).

---

# 17. Módulo `conversion`

## Servicio: `ConversionService`

`determineRarity(totalVotes, totalRegisteredUsers)`, `convert(input)` → `{attack, defense, hp, speed, rarity}`. Ambos puros, sin efectos secundarios.

## Función pura: `computeStatsForRarity(rarity, averageStars?, multipliers?)`

Usada tanto por la conversión real como por el sembrado de catálogo (rareza sin votos reales de por medio).

---

# 18. Módulo `scheduler`

## Servicio: `ConversionSchedulerService`

`handleCron()` (`@Cron`, dispara diariamente), `runConversionCycle()` (la rutina real, invocable independientemente del reloj — ver `pandora-diseno-arquitectura.md` §5.6).

## Controlador de soporte: `ConversionSchedulerController`

Expone `POST /conversion/run` delegando directamente en `runConversionCycle()`.

---

# 19. Módulo `common` (transversal)

- **Guards:** `AppThrottlerGuard` (extiende `ThrottlerGuard`, castellaniza el mensaje 429), `BannedUserGuard`, `VerifiedEmailGuard`.
- **Middleware:** `loggingMiddleware` (ver `pandora-diseno-arquitectura.md` §3).
- **Filtro global:** `AllExceptionsFilter` (normaliza toda excepción a `{statusCode, code, message, errors?, timestamp, path}`).
- **Pipe:** `buildImageValidationPipe(fileIsRequired)` (tipo real de archivo + tamaño máximo).
- **Decoradores:** `@SanitizeText()` (remueve HTML de campos de texto libre), `@ExposeId()` (normaliza bigint→number en respuestas), `@Public()`, `@CurrentUser()`, `@Roles(...)`.

