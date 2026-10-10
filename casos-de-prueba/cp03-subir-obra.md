# CP03 — Subir obra (CU09)

**Caso de uso:** [CU09 — Subir obra](/casos-de-uso/cu09-subir-obra.md)

- **Precondición:** usuario autenticado, con email verificado.
- **Pasos:** completar título, imagen, y opcionalmente lore/tags/solicitud de conversión → backend valida campos y archivo → sube a Cloudinary → registra la obra con `qualification_started_at = now()`.
- **Resultado esperado:** obra creada en `conversion_status = pending`, visible en el explorador, notificando a seguidores del autor.
- **Casos negativos:** archivo > 25 MB (A1) · formato no permitido, incluyendo mimetype falsificado (A2) · título vacío o demasiado largo (A3) · fallo de Cloudinary no debe dejar una obra incompleta (A4) · usuario baneado o sin email verificado → 403 (A6).
