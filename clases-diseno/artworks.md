# Módulo `artworks`

## Entidad: `Artwork`

`id`, `title: string`, `description: string | null`, `author_id`, `author: User`, `image_original_url` / `image_medium_url` / `image_thumbnail_url: string`, `image_public_id: string`, `conversion_request: boolean`, `conversion_status: ConversionStatus`, `qualification_started_at: Date`, `deleted: boolean`, `tags: Tag[]` (`ManyToMany`), `created_at`, `updated_at`.

## Servicio: `ArtworksService`

| Operación | Responsabilidad |
|---|---|
| `create(authorId, dto, file)` | Sube imagen a Cloudinary, crea la obra en `pending`, dispara notificación a seguidores. |
| `createByAdmin(adminId, dto, file)` | CU29: crea obra + carta directamente, `conversion_status = converted`, sin notificar seguidores. |
| `findOne(id, currentUserId?)` | Detalle público + `isFavorited` si hay sesión. |
| `findAll(query)` / `findMine(authorId, pagination)` | Explorador con filtros (búsqueda, autor, fechas, tag, rareza, calificación) y listado propio. |
| `update` / `remove` | Edición por el autor; eliminación lógica por autor o moderador. CU10-A1/CU11-A1: rechazan con 409 `ARTWORK_ALREADY_CONVERTED` cuando `conversion_status = converted` — pero sólo si quien actúa es el autor; un moderador puede eliminar una obra ya convertida sin esta restricción. |
| `moderationRemove(id)` / `restore(id)` | Eliminación/restauración por moderación (sin validar ownership; el guard ya restringe el acceso), con cascada lógica a comentarios/valoraciones/carta/favoritos. |

## DTOs

`CreateArtworkDto`, `UpdateArtworkDto`, `AdminUploadCardDto`, `QueryArtworksDto` (entrada) · `ArtworkResponseDto` (con `ArtworkImageDto`, `QualificationDto` anidados) (salida).
