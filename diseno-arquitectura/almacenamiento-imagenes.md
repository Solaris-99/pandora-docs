# Almacenamiento de imágenes (Cloudinary)

Se utiliza **Cloudinary** para las imágenes de obras.

## Flujo

```text
Cliente
  |
  v
Backend valida metadata
  |
  v
Subida a Cloudinary
  |
  +--> original
  +--> thumbnail
  +--> medium
  |
  v
Backend guarda URLs / public_id
  |
  v
Artwork disponible
```

## Consideraciones

- No guardar blobs de imagen directamente en PostgreSQL.
- Guardar identificadores/URLs de Cloudinary.
- Utilizar transformaciones para thumbnails.
- Mantener acceso público sólo a los recursos que correspondan.
- Definir política de borrado de Cloudinary al eliminar una obra.
- No confiar sólo en la extensión del archivo: verificar MIME/type y contenido real (firma binaria).
