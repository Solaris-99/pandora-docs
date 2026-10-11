# CU31 — Subir avatar

**Actor principal:** Usuario autenticado, con email verificado.
**Precondición:** El usuario tiene sesión autenticada y su email verificado.
**Postcondición:** El avatar de la cuenta queda actualizado con la imagen subida.
**Caso de prueba:** todavía sin caso de prueba dedicado.

## Flujo principal

1. El usuario entra a la configuración de su perfil.
2. Elige "Cambiar avatar".
3. Selecciona una imagen (máximo 2 MB).
4. El backend valida el archivo y lo sube a Cloudinary.
5. El avatar anterior (si había uno subido así) se elimina de Cloudinary.
6. Se actualiza `avatar_url` de la cuenta con la nueva imagen.

## Alternativo

- **A1:** Archivo > 2 MB -> rechazar.
- **A2:** Formato no permitido (sólo PNG/JPG/JPEG/WEBP) -> rechazar.
- **A3:** Usuario sin email verificado -> 403.
- **A4:** Fallo de Cloudinary -> no se actualiza el avatar de la cuenta.

## Observaciones

Mismo mecanismo que [CU09](cu09-subir-obra.md) (subir obra): validación de tipo real de archivo (no sólo la extensión/`mimetype` declarado) y almacenamiento en Cloudinary, pero en su propia carpeta y con un límite de peso mucho menor — un avatar es una imagen chica mostrada en un único tamaño en toda la interfaz, a diferencia de una obra.

Distinto de [CU06](cu06-ver-editar-perfil.md): `PATCH /users/me` sigue aceptando `avatar_url` como texto libre (por ejemplo, el avatar importado automáticamente desde Google al autenticarse — ver flujo de Google en [interfaces entre sistemas](/interfaces-entre-sistemas.md)); este caso de uso es la vía recomendada para que el usuario suba su propia imagen, pasando por Cloudinary en lugar de apuntar a una URL externa arbitraria.
