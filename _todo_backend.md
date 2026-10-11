# Pendientes del backend

> Trabajo detectado a partir de los casos de uso agregados en [casos-de-uso](/casos-de-uso/README.md) (CU05, CU07, CU08, CU10, CU11) que todavía no tienen contrato, endpoint o validación implementados. Lo que requiere coordinación con los clientes ya está en `_todo-frontend.md` y `_todo_mobile.md`; acá sólo queda lo que resuelve el backend por su cuenta.

## ~~Endpoint para cambiar contraseña (CU08)~~ — resuelto

`PATCH /users/me/password` (`UsersController` → `UsersService.changePassword`), payload `{currentPassword, newPassword}`. Verifica `currentPassword` contra el hash almacenado (`401 INVALID_CREDENTIALS` si no coincide), misma política de longitud que el registro para `newPassword`, `409 NO_LOCAL_PASSWORD` si la cuenta es sólo-Google (`password_hash` nulo), y revoca los demás `refresh_tokens` activos de la cuenta al confirmar — mismo mecanismo que `resetPassword` (CU04). Ver [clases de diseño §users](/clases-diseno/users.md) y [convenciones de API §8.6](/api-conventions.md).

## ~~Endpoint para cambiar email (CU07)~~ — resuelto

Dos endpoints: `POST /users/me/email` (solicita el cambio — valida formato/unicidad/distinción contra el email actual, invalida cualquier solicitud pendiente previa y envía el correo de confirmación a la dirección **nueva**) y `POST /auth/confirm-email-change` (público, token por email — recién ahí se aplica `User.email`). Se optó por la tabla análoga a `email_verification_tokens` (nueva entidad `EmailChangeToken`/`email_change_tokens`, con `new_email` propio) en vez de agregar `pending_email` a `users` — reutiliza el mismo mecanismo de token opaco hasheado de un solo uso. Ver [modelo de tablas](/modelo-tablas/email-change-tokens.md), [clases de diseño §users](/clases-diseno/users.md) y [convenciones de API §7.6/§8.7](/api-conventions.md).

## ~~Job de limpieza de cuentas no verificadas (CU05)~~ — resuelto

`AccountCleanupSchedulerService` (nuevo, mismo módulo `scheduler` que `ConversionSchedulerService`), `@Cron` diario a las 00:00. Busca `User` con `email_verified = false` y `created_at` anterior a 72h, y los elimina físicamente por repositorio (sin cascada manual: todas las FK hacia `users.id` ya estaban declaradas `ON DELETE CASCADE`/`SET NULL`). Si falla una cuenta puntual, se registra el error y se sigue con el resto del lote. Ver [clases de diseño §scheduler](/clases-diseno/scheduler.md).

## ~~Validar período de calificación al editar/eliminar obra (CU10/CU11)~~ — resuelto

No lo rechazaban. `ArtworksService.update()`/`remove()` ahora responden `409 ARTWORK_ALREADY_CONVERTED` cuando `conversion_status = converted`. En `remove()` el chequeo es específico del autor (`!isModerator`): un moderador/admin puede seguir eliminando una obra ya convertida en cualquier momento, igual que `moderationRemove`, que no se tocó. Ver [clases de diseño §artworks](/clases-diseno/artworks.md) y [convenciones de API §9.2](/api-conventions.md).
