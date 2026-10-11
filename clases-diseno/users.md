# Módulo `users`

## Entidades

- **User**: `id`, `username: string`, `email: string`, `password_hash: string | null`, `email_verified: boolean`, `status: UserStatus`, `bio: string | null`, `avatar_url: string | null`, `roles: Role[]` (`ManyToMany`), `created_at`, `updated_at`.
- **Follow**: `follower_id`, `followed_id`, `created_at`.
- **EmailChangeToken** (CU07): `id`, `user_id`, `new_email: string`, `token_hash: string`, `expires_at: Date`, `used_at: Date | null`, `created_at`. Relación `ManyToOne` a `User` (`onDelete: CASCADE`). Mismo patrón que `EmailVerificationToken` (token opaco hasheado, un solo uso, con expiración), pero además guarda el email nuevo propuesto — recién se aplica a `User.email` al confirmarse.

## Servicio: `UsersService`

| Operación | Responsabilidad |
|---|---|
| `findByEmail` / `findByUsername` / `findById` | Búsquedas base (con roles cargados). |
| `create` / `save` / `updateStatus` | Persistencia de alta y cambios de estado. |
| `getPublicProfile(targetId, currentId?)` | Perfil público + contadores de seguidores/seguidos + `isFollowing` si hay sesión. |
| `updateProfile(userId, dto)` | Edita username/bio/avatar propios, valida unicidad de username. |
| `changePassword(userId, dto)` (CU08) | Verifica `currentPassword` contra el hash almacenado, hashea y guarda `newPassword`, revoca los `refresh_tokens` activos de la cuenta (mismo efecto que `AuthService.resetPassword`). 409 `NO_LOCAL_PASSWORD` si la cuenta es sólo-Google (`password_hash` nulo). |
| `requestEmailChange(userId, dto)` / `confirmEmailChange(token)` (CU07) | Dos pasos: la solicitud valida unicidad/distinción contra el email actual e invalida cualquier `EmailChangeToken` pendiente previo antes de emitir uno nuevo y enviar el correo (vía `MailService`); la confirmación recién copia `new_email` a `User.email`, re-chequeando unicidad por si el email fue tomado por otra cuenta mientras tanto. |
| `followUser` / `unfollowUser` | Alta/baja idempotente de `Follow`, dispara notificación `user_follow`. |
| `getFollowers` / `getFollowing` | Listados paginados. |
| `count()` | Total de usuarios registrados — usado como divisor en la fórmula de rareza (ver [requerimientos §5.7](/requerimientos/reglas-07-rarezas.md)). |

Nota de dependencias: `UsersService` inyecta directamente el repositorio de `RefreshToken` (entidad del módulo `auth`) para la revocación de `changePassword`, en vez de depender de `AuthService` — evita un ciclo de módulos, ya que `AuthModule` depende de `UsersModule` (no al revés). Mismo motivo por el que `AuthController` (dentro de `AuthModule`, que sí puede depender de `UsersModule`) expone la confirmación pública del cambio de email (`POST /auth/confirm-email-change`) delegando en `UsersService.confirmEmailChange()`.

## DTOs

`UpdateProfileDto`, `ChangePasswordDto`, `ChangeEmailDto`, `ConfirmEmailChangeDto` (entrada) · `PublicProfileResponseDto`, `UpdateProfileResponseDto`, `FollowListItemDto` (salida).
