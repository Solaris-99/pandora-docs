# user_cards

| Campo | Tipo | Reglas |
|---|---|---|
| id | bigint | PK |
| user_id | bigint | FK users.id |
| card_id | bigint | FK cards.id |
| copies | integer | NOT NULL, default 0 |

## Constraints

- `UNIQUE (user_id, card_id)`.
- `CHECK (copies >= 0)`.

El contador se incrementa al recibir repetidas.
