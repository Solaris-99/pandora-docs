# artworks

| Campo                    | Tipo         | Reglas                  |
| ------------------------ | ------------ | ----------------------- |
| id                       | bigint       | PK                      |
| title                    | varchar(50)  | NOT NULL                |
| description              | varchar(300) | nullable                |
| author_id                | bigint       | FK users.id, NOT NULL   |
| image_original_url       | varchar      | NOT NULL                |
| image_medium_url         | varchar      | NOT NULL                |
| image_thumbnail_url      | varchar      | NOT NULL                |
| image_public_id          | varchar      | nullable (id de Cloudinary) |
| conversion_request       | boolean      | NOT NULL, default true  |
| conversion_status        | enum         | NOT NULL                |
| qualification_started_at | timestamp    | NOT NULL                |
| deleted                  | boolean      | NOT NULL, default false |
| created_at               | timestamp    | NOT NULL                |
| updated_at                | timestamp    | nullable                |

## Enum `conversion_status`

```text
pending
processing
converted
skipped
failed
```

Interpretación: `pending` todavía no procesada · `processing` tomada por un proceso de conversión · `converted` ya generó su carta · `skipped` no debe convertirse · `failed` ocurrió un error y puede quedar disponible para reintento controlado.

## Constraints

- `title` obligatorio y máximo 50 caracteres.
- `description` máximo 300 caracteres.
- `author_id` debe referenciar un usuario existente.
