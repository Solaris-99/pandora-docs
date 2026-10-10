# favorite_artworks

| Campo | Tipo | Reglas |
|---|---|---|
| user_id | bigint | FK users.id |
| artwork_id | bigint | FK artworks.id |
| created_at | timestamp | NOT NULL |

**PK compuesta:** `(user_id, artwork_id)`.
