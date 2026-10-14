# Pedidos del frontend al backend

> Pedidos de cambio de contrato/endpoint hacia el backend, y decisiones o limitaciones del cliente web con valor de coordinación para mobile/backend. Antes esto último vivía separado en `frontend-todo.md` (deuda técnica puramente del cliente); se consolidó todo acá para no mantener dos listas. Los ítems resueltos quedan tachados con el detalle de cómo se resolvió, para que quede el rastro de qué se pidió.

## Seguridad / autenticación

### Cookie `httpOnly` para el `refreshToken` (pendiente, hardening opcional)

`api-conventions.md` §5.4 recomienda evitar `localStorage` para el `refreshToken` por exposición a XSS, y sugiere `httpOnly` cookie o almacenamiento cifrado. El frontend no puede lograr `httpOnly` unilateralmente — requiere que el backend setee la cookie vía `Set-Cookie` en `/auth/login`, `/auth/register` y `/auth/refresh`. Si se prioriza, sería un modo opcional (Android seguiría necesitando el token en el body JSON). Mientras tanto el `refreshToken` vive en `localStorage` (`persist` de Zustand, `features/auth/store.ts`) — un `localStorage` "cifrado" no es una alternativa real, ya que la clave de cifrado tendría que vivir en el mismo JS accesible al atacante.

### ~~Validación de composición de contraseña en el backend~~ — resuelto

El backend ahora replica exactamente la política que ya exigía el cliente (`passwordSchema`, `src/lib/schemas.ts`): 8–50 caracteres, con al menos una mayúscula, una minúscula, un número y un símbolo. Decorador compartido `IsStrongPassword()` (`src/common/decorators/is-strong-password.decorator.ts`), aplicado a `RegisterDto.password`, `ResetPasswordDto.newPassword` y `ChangePasswordDto.newPassword` — nunca a `currentPassword`, que sólo se compara contra el hash existente, no se re-valida contra la política actual (una cuenta creada antes de este cambio puede seguir autenticándose con una contraseña más corta/simple; sólo al cambiarla se le exige la política nueva). Ver [convenciones de API §4](/api-conventions.md).

### Flujo de Google web sin `refreshToken` (decisión de diseño del cliente)

El flujo de redirección (§7.3) no recibe `refreshToken` por diseño — no se quiere exponer un token de larga duración en la URL de redirección. Como consecuencia, esa sesión expira a los 15 minutos sin refresco silencioso y el usuario tiene que volver a autenticarse con Google. El flujo 7.4 (popup/fetch con `idToken`) daría sesión persistente, pero no se implementó por requerir integrar el SDK de Google Identity Services en el cliente web. El refresco de `accessToken` además es sólo reactivo (on-401, `src/lib/axios.ts`), no proactivo vía el `exp` del JWT — una mejora menor, no un bug.

### IDs serializados como string en algunas respuestas

El backend documenta todo id como número JSON (`api-conventions.md`), pero en la práctica a veces serializa bigints como string (`"id": "43"`). El frontend se defiende centralizando `idSchema = z.coerce.number()` (`src/lib/schemas.ts`) en vez de confiar en el tipo documentado — si esto persiste o reaparece, Android debería considerar una defensa equivalente al parsear JSON (un `Int`/`Long` estricto rompería donde el frontend web no rompe).

## Obras

### Explorador/Home sólo muestran obras en período de calificación

`ExplorePage` y el preview de obras recientes del Home filtran con `GET /artworks?qualificationOnly=true` (§9.4) — sólo obras con `conversionStatus: "pending"`, aún sin procesar por el ciclo de conversión. Una obra ya `converted`/`skipped` no pertenece a "explorar": vive en el catálogo de cartas. Relevante para que mobile replique el mismo recorte si tiene una pantalla equivalente.

### ~~Búsqueda de tags server-side~~ — resuelto en el backend, pendiente de migrar en el frontend

`GET /api/v1/tags?search=<query>` (§10) ya filtra server-side (`ILIKE` sobre `name`), devuelto como array plano sin paginación (catálogo todavía chico). `TagsInput` (subir/editar obra) sigue filtrando en el cliente sobre el `GET /tags` completo — no es urgente mientras el catálogo sea chico, pero falta migrarlo.

### Obra convertida ya no puede editarse ni eliminarse por su autor (`ARTWORK_ALREADY_CONVERTED`)

CU10-A1/CU11-A1: el backend responde 409 `ARTWORK_ALREADY_CONVERTED` si el autor intenta editar/eliminar una obra con `conversionStatus: "converted"` (un moderador/admin sí puede eliminarla en cualquier momento). El frontend lo gatea proactivamente — `ArtworkDetailPage` oculta editar/eliminar y `ArtworkEditPage` muestra un estado vacío en vez del 409 reactivo. Mobile debería aplicar la misma precondición en su propia UI de edición.

## Cartas y favoritos

### ~~`owned=true` / `isFavorited`~~ — resueltos en el backend, pendientes de migrar en el frontend

`GET /cards`/`GET /users/me/cards?owned=true` (§13.4) ya filtra a sólo las cartas que el usuario posee, e `isFavorited` ya está expuesto en `GET /artworks/:id` y `GET /cards/:id` (§9.3/§13.1, mismo patrón que `isFollowing`, sólo en el detalle, no en los listados). El frontend todavía no los consume: `BattleCardPicker` sigue pidiendo `useMyCards({ page: 1, limit: 100 })` y filtrando `owned` en el cliente, y `useFavoriteArtworkIds`/`useFavoriteCardIds` (`features/favorites/hooks.ts`) siguen resolviendo "¿está destacado?" pidiendo la lista completa de destacados propios (mismo límite de 100) y comprobando membership — ambos son el mismo patrón pragmático, pendiente de simplificar.

## Batalla

### ~~Reglas de la batalla local~~ — resuelto

La fórmula (dodge/crítico/reducción de daño) quedó acordada y documentada en `battle-formula.md` (2026-10-07); [requerimientos §5.8](/requerimientos/reglas-08-batallas.md) define que la simulación es local a cada cliente. `features/battle/simulate.ts` en el frontend es la implementación de referencia. Si se agrega algo más elaborado (multi-golpe, efectos por rareza/emoción, etc.), hay que actualizar primero `battle-formula.md` y después cada cliente.

## Notificaciones y apelaciones

### ~~Notificaciones sin referencia al recurso afectado~~ — resuelto en el backend, pendiente de migrar en el frontend

Las notificaciones de moderación ya traen `data: { targetType, targetId }` (§16.6) en `artwork_removed`/`comment_removed`/`artwork_restored`/`comment_restored`/`appeal_resolved`. El frontend todavía no lo usa: una notificación de contenido eliminado no permite deep-link hasta la apelación — `MyAppealsPage`/`AppealContentDialog` le siguen pidiendo el tipo y el ID manualmente al usuario. Pendiente reemplazar ese diálogo por un botón directo cuando la notificación traiga esos datos.

No hay mecanismo de push: el contador de no leídas (`useUnreadCount`) hace polling en un intervalo corto en vez de recibir un evento. Si se agrega push/WebSocket en el backend más adelante, ambos clientes podrían dejar de hacer polling.

## Cuenta

### ~~Cambiar contraseña (CU08)~~ — resuelto y consumido

`PATCH /users/me/password` (§8.6). El frontend agregó una sección "Cambiar contraseña" en `SettingsProfilePage` (`useChangePassword`). Como no hay un campo en el usuario de sesión que indique si la cuenta tiene contraseña local (login sólo-Google), el formulario se muestra siempre y `NO_LOCAL_PASSWORD` (409) se maneja reactivo, igual que `INVALID_CREDENTIALS`.

### ~~Cambiar email (CU07)~~ — resuelto y consumido

`POST /users/me/email` + `POST /auth/confirm-email-change` (§7.6/§8.7). El frontend agregó una sección "Cambiar email" en `SettingsProfilePage` (`useRequestEmailChange`) y una página pública nueva, `ConfirmEmailChangePage`, para confirmar desde el link del correo (mismo patrón que `VerifyEmailPage`/`ResetPasswordPage`). El email de sesión no se actualiza hasta confirmarse.

### ~~Subir avatar (CU31)~~ — resuelto y consumido

`POST /users/me/avatar` (§8.8, multipart, campo `avatar`, máx. 2 MB). El frontend agregó una sección "Avatar" en `SettingsProfilePage` (`useUploadAvatar`), gateada en el cliente a `user.emailVerified` para no depender de que el 403 `USER_NOT_VERIFIED` llegue recién al enviar.

## Administración

### Campos de stats opcionales: vacío vs. cero

`AdminCardUploadForm` usa `z.preprocess` para distinguir "vacío" de "0" en `attack`/`defense`/`hp`/`speed` (`src/features/admin/api.ts`): un input vacío se envía como `undefined` (el backend calcula el valor por rareza) en vez de coercionar a `0`. Si mobile tiene una pantalla equivalente de carga admin, debería omitir el campo en vez de mandar `0` por el mismo motivo.

## Mobile (detectado desde la app)

### Verificación de email y reset de contraseña desde la app (opcional, no bloquea)

Los correos de verificación y recuperación ([interfaces entre sistemas, correos enviados](/interfaces-entre-sistemas/resend.md#correos-enviados)) enlazan a `{FRONTEND_URL}/verify-email?token=...` y `{FRONTEND_URL}/reset-password?token=...`. Mobile los abre en el navegador (el portal web hace el `POST` correspondiente), sin cambios de contrato. Si se quisiera abrirlos directo en la app habría que, del lado web/backend: publicar `/.well-known/assetlinks.json` en el dominio del frontend (con el SHA-256 de la firma de la app) y confirmar que las rutas se mantengan estables.
