# Pedidos del frontend al backend

> Ítems detectados durante la implementación del frontend que requieren un cambio de contrato o un endpoint nuevo — no son deuda técnica del lado del frontend, sino mejoras a coordinar con el backend. Movidos acá desde `frontend-todo.md` (que sólo debería contener limitaciones/decisiones puramente del cliente).

## ~~Búsqueda de tags~~ — resuelto

`GET /api/v1/tags?search=<query>` (api-conventions.md §10) ya filtra server-side (`ILIKE` sobre `name`). Se mantuvo como array plano sin envoltorio de paginación — el catálogo de tags sigue siendo chico, así que no se agregó paginación real todavía; si el catálogo crece mucho eso sí quedaría pendiente.

## ~~Filtro `rarity` en el explorador de obras~~ — resuelto

`GET /artworks?rarity=<rareza>` (§9.4) ya filtra por la rareza de la carta ya generada (join contra `cards`). Decisión sobre el caso "obra sin carta convertida": simplemente no matchea el filtro — no hay un valor "sin rareza" explícito.

## ~~Selector de cartas propias sin filtro `owned`~~ — resuelto

`GET /users/me/cards?owned=true` (§13.4) ya filtra a sólo las cartas que el usuario posee (`copies > 0`), tanto en `GET /cards` como en `GET /users/me/cards`. Sin token, `owned=true` devuelve una lista vacía en vez de ignorarse.

## ~~`isFavorited` ausente en el detalle de obra/carta~~ — resuelto

`GET /artworks/:id` (§9.3) y `GET /cards/:id` (§13.1) ya exponen `isFavorited` (mismo patrón que `isFollowing`): sólo presente con un token válido, ausente en petición anónima. No se agregó a los listados (`GET /artworks`, `GET /cards`), sólo al detalle — se consideró fuera de alcance de este pedido.

## ~~Notificaciones sin referencia al recurso afectado~~ — resuelto

Las notificaciones de moderación ahora traen un campo opcional `data: { targetType, targetId }` (§16.6) en `artwork_removed`, `comment_removed`, `artwork_restored`, `comment_restored` (`targetType: "artwork"|"comment"`) y `appeal_resolved` (`targetType: "appeal"`). `user_warned`/`user_banned`/`user_unbanned` quedaron fuera a propósito: el flujo de apelación de baneo (`POST /users/me/ban-appeal`) siempre es sobre la propia cuenta, no necesita un id.

## ~~`GET /artworks/qualification` rechaza los filtros que documenta §9.5~~ — resuelto

El atajo dedicado `GET /artworks/qualification` fue eliminado (tanto de la API como de la documentación) en vez de arreglado: sólo aceptaba `page`/`limit` mientras el contrato prometía el set completo de filtros de `/artworks`, y el frontend (`ExplorePage`, preview de obras recientes en Home) ya usaba `GET /artworks?qualificationOnly=true` en su lugar. Ese filtro sobre el explorador general sigue siendo la única vía — ya aceptaba correctamente el set completo de filtros y sigue documentado en §9.4.

## Cookie `httpOnly` para el `refreshToken` (opcional, hardening)

`api-conventions.md` §5.4 recomienda evitar `localStorage` para el `refreshToken` por exposición a XSS, y sugiere `httpOnly` cookie o almacenamiento cifrado. El frontend no puede lograr `httpOnly` unilateralmente — requiere que el backend setee la cookie vía `Set-Cookie` en `/auth/login`, `/auth/register` y `/auth/refresh`. Si se prioriza este hardening para el cliente web, sería un modo opcional (la app Android seguiría necesitando el token en el body JSON).
