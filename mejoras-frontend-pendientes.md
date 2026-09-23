# Pedidos del frontend al backend

> Ítems detectados durante la implementación del frontend que requieren un cambio de contrato o un endpoint nuevo — no son deuda técnica del lado del frontend, sino mejoras a coordinar con el backend. Movidos acá desde `frontend-todo.md` (que sólo debería contener limitaciones/decisiones puramente del cliente).

## Búsqueda de tags

`GET /api/v1/tags` (api-conventions.md §10) devuelve el catálogo completo sin paginar ni filtrar. El selector de tags al subir/editar una obra (`TagsInput`, `features/artworks/components/TagsInput.tsx`) ahora es un combobox buscable, pero el filtrado ocurre **en el cliente** sobre la lista completa ya descargada — funciona bien mientras el catálogo de tags sea chico, pero no escala si llega a crecer mucho (miles de tags). Si eso pasa, convendría un `GET /tags?search=<query>` (con paginación estándar) para no tener que traer todo el catálogo a cada carga de la página.

## Filtro `rarity` en el explorador de obras

`GET /artworks` (§9.4) no acepta un filtro por `rarity`, a diferencia del explorador de cartas (`GET /cards`, que sí lo tiene). El diseño original (`pandora-diseño-general.md`) lo mencionaba como filtro deseado para obras; sigue pendiente porque la rareza es un atributo de la Carta, no de la Obra, y una obra no siempre tiene carta generada — habría que definir qué hacer con las obras sin `conversionStatus: "converted"` al filtrar por rareza.

## Selector de cartas propias sin filtro `owned`

Tanto el selector de cartas de batalla (`BattleCardPicker`) como el chequeo de estado de favoritos (`useFavoriteCardIds`) necesitan la lista de cartas que el usuario **posee**, pero `GET /users/me/cards` (§13.4) siempre devuelve el catálogo completo (con `owned: false` para las no poseídas), sin un filtro `owned=true`. Ambos casos hoy piden `limit=100` y filtran en el cliente — un `owned=true` en `GET /users/me/cards` eliminaría la necesidad de ese límite pragmático y sería correcto sin importar cuántas cartas tenga el catálogo.

## `isFavorited` ausente en el detalle de obra/carta

Ni `GET /artworks/:id` (§9.3) ni `GET /cards/:id` (§13.1) exponen si el ítem ya está destacado por el usuario autenticado — a diferencia de `isFollowing` en el perfil de usuario (§8.1). El frontend resuelve esto pidiendo la lista completa de destacados propios (`GET /users/me/favorites/artworks|cards`, `limit=100`) y comprobando membership en el cliente (mismo problema de límite pragmático que el punto anterior). Agregar `isFavorited`/`favorited` a ambos detalles evitaría esto por completo, igual que ya se hizo con `isFollowing`.

## Notificaciones sin referencia al recurso afectado

Las notificaciones de moderación (`artwork_removed`, `comment_removed`, y ahora también `artwork_restored`/`comment_restored`/`user_unbanned`/`appeal_resolved` de §18.7) sólo traen `title`/`content` como texto libre — no incluyen el `id` del recurso afectado. Esto le impide al frontend armar un link directo de "tu obra fue eliminada" hacia el formulario de apelación (`POST /artworks/:id/appeals`); hoy `MyAppealsPage` le pide el ID al usuario manualmente (visible en el texto de la notificación). Un campo opcional en la notificación (p. ej. `data: { targetType, targetId }`) resolvería esto.

## Cookie `httpOnly` para el `refreshToken` (opcional, hardening)

`api-conventions.md` §5.4 recomienda evitar `localStorage` para el `refreshToken` por exposición a XSS, y sugiere `httpOnly` cookie o almacenamiento cifrado. El frontend no puede lograr `httpOnly` unilateralmente — requiere que el backend setee la cookie vía `Set-Cookie` en `/auth/login`, `/auth/register` y `/auth/refresh`. Si se prioriza este hardening para el cliente web, sería un modo opcional (la app Android seguiría necesitando el token en el body JSON).
