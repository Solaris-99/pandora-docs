# artwork_ratings

| Campo | Tipo | Reglas |
|---|---|---|
| id | bigint | PK |
| artwork_id | bigint | FK artworks.id, NOT NULL |
| voter_id | bigint | FK users.id, NOT NULL |
| stars | smallint | NOT NULL |
| emotion | enum | NOT NULL |
| created_at | timestamp | NOT NULL |
| updated_at | timestamp | nullable |

## Enum `emotion`

```text
joy
fear
anger
serenity
grief
awe
```

## Constraints

- `CHECK (stars BETWEEN 1 AND 5)`.
- `UNIQUE (artwork_id, voter_id)`.
- No se admite valoración sobre obra eliminada (regla de dominio, validada en servicio).
