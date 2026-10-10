# Módulo `comments`

## Entidad: `Comment` (`artwork_comments`)

`id`, `author_id`, `artwork_id`, `comment: string`, `deleted: boolean`, `created_at`, `updated_at`.

## Servicio: `CommentsService`

`create(artworkId, authorId, dto)`, `findAllForArtwork(artworkId, pagination)`, `removeAllForArtwork` (cascada).
