# Módulo `storage`

## Servicio: `StorageService`

`uploadArtworkImage(file)` → `{publicId, originalUrl, mediumUrl, thumbnailUrl}` (sube a Cloudinary, genera transformaciones; carpeta `pandora/artworks`). `uploadAvatarImage(file)` (CU31) → `{publicId, avatarUrl}` — mismo pipeline, carpeta propia (`pandora/avatars`) y una única transformación cuadrada fija (256×256), ya que a diferencia de una obra el avatar siempre se muestra en un solo tamaño. `deleteImage(publicId)` (genérico a propósito: borra cualquier recurso por su `public_id`, lo use `ArtworksService` o `UsersService`; best-effort, no bloquea el flujo si Cloudinary falla).
