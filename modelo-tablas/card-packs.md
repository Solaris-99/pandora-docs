# card_packs

| Campo | Tipo | Reglas |
|---|---|---|
| id | bigint | PK |
| user_id | bigint | FK users.id, UNIQUE |
| amount | integer | NOT NULL, default 0 |
| last_regeneration_at | timestamp | nullable |
| updated_at | timestamp | nullable |

## Constraints

- `UNIQUE (user_id)` para representar un inventario de paquetes por usuario.
- `CHECK (amount BETWEEN 0 AND 10)`.

## Regeneración

```text
elapsed_minutes = floor(now - last_regeneration_at)
regenerated = min(elapsed_minutes, 10 - amount)
```

Cálculo realizado al consultar/modificar los paquetes, en vez de almacenar eventos de regeneración.
