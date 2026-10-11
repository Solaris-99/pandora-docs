# Pendientes del backend

> Trabajo detectado a partir de los casos de uso agregados en [casos-de-uso](/casos-de-uso/README.md) (CU05, CU07, CU08, CU10, CU11) que todavía no tienen contrato, endpoint o validación implementados. Lo que requiere coordinación con los clientes ya está en `_todo-frontend.md` y `_todo_mobile.md`; acá sólo queda lo que resuelve el backend por su cuenta.

## Endpoint para cambiar contraseña (CU08)

No existe todavía un endpoint para que un usuario autenticado cambie su propia contraseña — `/auth/forgot-password`/`/auth/reset-password` son sólo para quien no puede iniciar sesión. [CU08](/casos-de-uso/cu08-cambiar-contrasena.md) describe el flujo: contraseña actual + nueva contraseña en dos campos (nueva/repetir).

Pendiente:
- `PATCH /users/me/password` (Auth), payload `{currentPassword, newPassword}`.
- Verificar `currentPassword` contra el hash almacenado antes de aceptar la nueva.
- Misma política de longitud que el registro (RNF, ver [requerimientos §5.1](/requerimientos/reglas-01-usuarios.md)).
- Por consistencia con [CU04](/casos-de-uso/cu04-recuperar-contrasena-olvidada.md), revocar los demás `refresh_tokens` activos de la cuenta al confirmar el cambio.
- No aplica si la cuenta no tiene `password_hash` (sólo Google) -> 400/409.

## Endpoint para cambiar email (CU07)

No existe todavía un endpoint para que un usuario autenticado cambie su propio email. [CU07](/casos-de-uso/cu07-cambiar-email.md) describe el flujo: el email nuevo no se aplica hasta confirmarse (mismo mecanismo que [CU02](/casos-de-uso/cu02-verificar-email.md)), y el email anterior sigue siendo el activo (login, notificaciones) mientras tanto.

Pendiente:
- `POST /users/me/email` (Auth), payload `{newEmail}` — valida formato y unicidad contra `users.email` antes de aceptar.
- Cambio de esquema en [`users`](/modelo-tablas/users.md): agregar `pending_email` (nullable), o una tabla análoga a [`email_verification_tokens`](/modelo-tablas/email-verification-tokens.md) que además guarde el email nuevo propuesto.
- Reutilizar el mecanismo de token opaco de un solo uso (hash SHA-256, expiración) ya usado por verificación de email y recuperación de contraseña, en vez de crear uno nuevo desde cero.
- Al confirmarse el token: copiar `pending_email` a `email`, limpiar `pending_email`.
- Si se solicita un nuevo cambio antes de confirmar el anterior, invalidar el token pendiente previo (mismo patrón que `password_reset_tokens`).

## Job de limpieza de cuentas no verificadas (CU05)

No existe todavía el proceso periódico que describe [CU05](/casos-de-uso/cu05-limpieza-de-cuentas-no-verificadas.md). Hoy una cuenta `pending_verification` que nunca confirma su email queda así indefinidamente.

Pendiente:
- Job con `@Cron`, mismo mecanismo que `ConversionSchedulerService` (ver [módulo `scheduler`](/clases-diseno/scheduler.md)), corriendo diariamente a las 00:00.
- Busca `users` con `email_verified = false` (o `status = pending_verification`) y `created_at` anterior a `now() - 72h`.
- A diferencia del resto del sistema, **no** es un borrado lógico: la cuenta se elimina físicamente, con cascada a sus tokens de verificación.
- Si falla la eliminación de una cuenta puntual, registrar el error y seguir con el resto del lote (no abortar el ciclo completo).

## Validar período de calificación al editar/eliminar obra (CU10/CU11)

[CU10](/casos-de-uso/cu10-editar-obra.md) y [CU11](/casos-de-uso/cu11-eliminar-obra.md) agregan una precondición que no está explícita todavía en `ArtworksService.update`/`remove` (ver [módulo `artworks`](/clases-diseno/artworks.md)): una obra ya convertida en carta (`conversion_status = converted`) no debería poder editarse ni eliminarse por su autor.

Pendiente:
- Confirmar si `update`/`remove` ya rechazan este caso; si no, agregar el chequeo (409 si `conversion_status = converted`) antes de aplicar el cambio.
- Este rechazo es específico del autor — no debe afectar la eliminación por moderación (`moderationRemove`), que puede aplicarse en cualquier momento.
