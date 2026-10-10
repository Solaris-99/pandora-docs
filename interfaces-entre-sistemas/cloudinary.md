# Cloudinary (almacenamiento de imágenes)

**Rol:** almacenamiento y transformación de las imágenes de obra (`RNF06`). Encapsulado íntegramente en `StorageService` (`src/storage/storage.service.ts`) — Adapter: ningún otro service llama al SDK de Cloudinary directamente.

**Configuración:** `CLOUDINARY_CLOUD_NAME`, `CLOUDINARY_API_KEY`, `CLOUDINARY_API_SECRET` (`src/config/image.config.ts`). Si falta alguna, `StorageService` queda en estado "no configurado" (loguea un `warn` al construirse) — el proceso arranca igual, pero `uploadArtworkImage()` responde `500 INTERNAL_SERVER_ERROR` explícito en vez de intentar la subida.

## Subida (`uploadArtworkImage`)

```text
ArtworksService.create()/update()
  │
  ▼
StorageService.uploadArtworkImage(file)
  │
  ▼
cloudinary.uploader.upload_stream(buffer)  — carpeta pandora/artworks
  │
  ▼
{ public_id, secure_url }
  │
  ├──► originalUrl = secure_url (tal cual la subió Cloudinary)
  ├──► mediumUrl    = cloudinary.url(public_id, width=1080, crop=limit, quality=auto, fetch_format=auto)
  └──► thumbnailUrl = cloudinary.url(public_id, width=300×300, crop=fill, gravity=auto, quality=auto, fetch_format=auto)
```

`mediumUrl`/`thumbnailUrl` son transformaciones *on-the-fly* del mismo `public_id` — no se generan ni almacenan archivos adicionales; Cloudinary las renderiza (y cachea) la primera vez que alguien pide esa URL.

## Borrado (`deleteArtworkImage`)

`cloudinary.uploader.destroy(publicId)`, invocado al reemplazar la imagen de una obra en una edición y al eliminarla. Es **best-effort**: un fallo se loguea (`this.logger.error`) pero nunca aborta la operación principal — la obra ya se guardó o se marcó eliminada independientemente de si Cloudinary logró borrar el recurso asociado.

## Límite conocido

La validación de tipo de archivo (`buildImageValidationPipe`) ocurre *antes* de llegar a Cloudinary, basada en el `mimetype` que reporta el cliente — no en un análisis de los primeros bytes del archivo. Cloudinary en sí no se usa como mecanismo de validación de contenido.
