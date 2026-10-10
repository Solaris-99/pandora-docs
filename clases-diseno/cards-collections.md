# Módulo `cards` (+ `collections`)

## Entidades

- **Card**: `id`, `artwork_id` (único, `OneToOne` con `Artwork`), `attack` / `defense` / `hp` / `speed: number`, `rarity: Rarity`, `deleted: boolean`, `created_at`, `updated_at`.
- **UserCard** (módulo `collections`): `id`, `user_id`, `card_id`, `copies: number`.

## Servicio: `CardsService`

| Operación | Responsabilidad |
|---|---|
| `findAll(query, currentUserId?)` | Explorador de cartas, con filtro `owned` (requiere sesión). |
| `findOne(id, currentUserId?)` | Detalle; oculta `stats` si `copies = 0` o no hay sesión. |
| `ensureRarityCatalogForAuthor(authorId)` | Garantiza que un autor tenga al menos una carta de cada rareza (autoreparación del catálogo mínimo). |
| `createForArtwork(artworkId, rarity, statsOverride?)` | CU29: crea la carta de una obra ya convertida; completa estadísticas faltantes vía `computeStatsForRarity`. |
| `removeByArtwork(artworkId)` | Cascada lógica al eliminar la obra de origen. |

## Servicio: `CardCatalogSeederService`

`onApplicationBootstrap()`: en cada arranque, delega en `ensureRarityCatalogForAuthor` bajo la cuenta de sistema `pandora`.

## DTOs

`QueryCardsDto` (entrada) · `CardResponseDto` (con `CardArtworkSummaryDto`, `CardStatsDto` anidados) (salida).
