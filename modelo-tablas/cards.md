# cards

| Campo | Tipo | Reglas |
|---|---|---|
| id | bigint | PK |
| artwork_id | bigint | FK artworks.id, UNIQUE |
| attack | integer | NOT NULL, >= 0 |
| defense | integer | NOT NULL, >= 0 |
| hp | integer | NOT NULL, >= 0 |
| speed | integer | NOT NULL, >= 0 |
| rarity | enum | NOT NULL |
| deleted | boolean | NOT NULL, default false |
| created_at | timestamp | NOT NULL |
| updated_at | timestamp | nullable |

## Enum `rarity`

```text
common
uncommon
rare
epic
legendary
```

## Constraints

- `UNIQUE (artwork_id)` garantiza una carta por obra.
- `CHECK ("attack" >= 0 AND "defense" >= 0 AND "hp" >= 0 AND "speed" >= 0)`.
- `rarity` limitada al enum.
