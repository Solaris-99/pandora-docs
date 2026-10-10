# artwork_tags

| Campo | Tipo | Reglas |
|---|---|---|
| artwork_id | bigint | FK artworks.id |
| tag_id | bigint | FK tags.id |

**PK compuesta:** `(artwork_id, tag_id)`.
