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

## Notificaciones sin referencia al recurso (detectado desde mobile)

`data: { targetType, targetId }` ([api-conventions §16.6](/api-conventions.md)) solo viaja en las notificaciones de moderación (`artwork_removed`, `comment_removed`, `artwork_restored`, `comment_restored`, `appeal_resolved`). El resto no trae ninguna referencia, así que el cliente no puede abrir el recurso al tocarlas:

| Tipo | Referencia que falta | Destino en la app |
|---|---|---|
| `artwork_comment`, `artwork_vote_summary`, `qualification_ended`, `artwork_converted`, `follow_upload` | `targetType: "artwork"`, `targetId` = id de la obra | Detalle de la obra |
| `user_follow` | `targetType: "user"`, `targetId` = id de quien siguió | Perfil de esa persona |
| `comment_restored` | `targetType: "comment"` ya viaja, pero sin el id de la obra a la que pertenece el comentario | Obra con el comentario |

Propuesta: agregar `data` a esos tipos (y, para comentarios, un `artworkId` opcional dentro de `data`). Es compatible hacia atrás: el campo ya es opcional. Mientras tanto, mobile solo enlaza las notificaciones que hoy traen `data`.

## Forma de la respuesta de apelaciones (detectado desde mobile)

[§18.3–18.5](/api-conventions.md) dicen que crear una apelación devuelve "la apelación creada, con `status: "pending"`", y que `GET /users/me/appeals` lista las propias, pero no documentan los campos de cada ítem. `AppealResponseDto` (clases de diseño) solo lista `id`, `reason`, `detail`, `status` y `report`. Mobile decodifica de forma tolerante (todo opcional salvo `id`) y muestra estado, motivo, texto y tipo de contenido. Para mostrar bien la lista falta documentar, como mínimo: `createdAt`, `reviewedAt` y, dentro de `report`, `targetType`, `targetId` y `resolution`; y confirmar los valores posibles de `status` (`pending` | `approved` | `rejected`).

## `createdAt` de comentarios adelantado 3 horas (detectado desde mobile)

Al comentar desde la app contra el backend de desarrollo, `POST /artworks/:id/comments` guardó `createdAt = 2026-10-14T21:43:51.585Z` cuando la hora UTC real era ≈ `18:43Z` (el campo `timestamp` de los errores del mismo servidor sí marca `18:44Z`). Parece una columna `timestamp` sin zona horaria o un `TZ` del proceso/DB distinto de UTC (el desfase coincide con UTC−3). El cliente muestra "Dentro de 2 horas" en comentarios recién creados. Revisar el tipo de columna (`timestamptz`) y el `TZ` de la base de datos para que todas las fechas se emitan en UTC real.

## `GET /artworks?converted=false` devuelve lo mismo que `converted=true` (detectado desde mobile)

Contra el backend de desarrollo, `GET /artworks?tag=fantasia&converted=false` y `...&converted=true` devuelven exactamente el mismo conjunto (`total: 9`, todas `conversionStatus: converted`), y sin el parámetro `total: 13`. [`api-conventions.md`](api-conventions.md) dice que `false` = "el resto". Causa probable: el DTO convierte el query string con `@Transform(({ value }) => Boolean(value))` (o `@Type(() => Boolean)`), con lo que la cadena `"false"` se evalúa como `true`. Debe parsearse `value === 'true'` / `'false'` de forma explícita. El mismo patrón puede afectar a `qualificationOnly` y a otros filtros booleanos de query. La app ya envía el valor correcto, por lo que el filtro "Sin carta" de Explorar no funcionará hasta que se corrija.
