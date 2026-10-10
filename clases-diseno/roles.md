# Módulo `roles`

## Entidad: `Role`

`id`, `name: RoleName` (`user`/`moderator`/`admin`), `description: string | null`.

## Servicio: `RolesService`

`findByName`, `ensureDefaultRoles()` (auto-repara el catálogo de roles al arrancar), otorgar/revocar `moderator` sobre un usuario objetivo.

## Guard: `RolesGuard`

Lee metadata `@Roles(...)` (por método o por clase, con `getAllAndOverride`) y compara contra `request.user.roles`.
