# Módulo `ratings`

## Entidad: `Rating`

`id`, `artwork_id`, `voter_id`, `stars: number (1–5)`, `emotion: Emotion`, `created_at`, `updated_at`.

## Servicio: `RatingsService`

`rate(artworkId, userId, dto)` (crea o actualiza), `unrate`, `getSummary(artworkId, currentUserId?)`, `getConversionAggregate(artworkId)` (promedio de estrellas, total de votos y emoción dominante — insumo de `ConversionService`), `removeAllForArtwork` (cascada), `notifyOwnerOfVote` (notificación apilable `artwork_vote_summary`).
