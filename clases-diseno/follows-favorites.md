# Módulo `follows` / `favorites`

- El modelo `Follow` vive físicamente en el módulo `users` (ver [`users`](users.md)); no hay un módulo `follows` separado con servicio propio.
- **Entidades** `FavoriteArtwork`, `FavoriteCard`: `user_id`, `artwork_id`/`card_id`, `created_at`.
- **Servicio `FavoritesService`**: `favoriteArtwork` / `unfavoriteArtwork` / `favoriteCard` / `unfavoriteCard` (idempotentes), `isArtworkFavorited` / `isCardFavorited` (usados por `ArtworksService`/`CardsService` al armar el detalle), `removeAllFavoritesForArtwork` / `removeAllFavoritesForCard` (cascada).
