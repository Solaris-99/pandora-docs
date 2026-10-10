# Módulo `storage`

## Servicio: `StorageService`

`uploadArtworkImage(file)` → `{publicId, originalUrl, mediumUrl, thumbnailUrl}` (sube a Cloudinary, genera transformaciones). `deleteArtworkImage(publicId)` (best-effort, no bloquea el flujo si Cloudinary falla).
