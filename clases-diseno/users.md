# Módulo `users`

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
| `count()` | Total de usuarios registrados — usado como divisor en la fórmula de rareza (ver [requerimientos §5.7](/requerimientos/reglas-07-rarezas.md)). |

## DTOs

`UpdateProfileDto` (entrada) · `PublicProfileResponseDto`, `UpdateProfileResponseDto`, `FollowListItemDto` (salida).
