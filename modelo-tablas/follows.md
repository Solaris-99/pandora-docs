# follows

| Campo | Tipo | Reglas |
|---|---|---|
| follower_id | bigint | FK users.id |
| followed_id | bigint | FK users.id |
| created_at | timestamp | NOT NULL |

**PK compuesta:** `(follower_id, followed_id)`.

## Constraints

- Prohibir seguimiento de uno mismo: `follower_id <> followed_id`.
- Una relación no puede duplicarse.
