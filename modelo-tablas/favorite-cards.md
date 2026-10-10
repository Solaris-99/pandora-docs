# favorite_cards

| Campo | Tipo | Reglas |
|---|---|---|
| user_id | bigint | FK users.id |
| card_id | bigint | FK cards.id |
| created_at | timestamp | NOT NULL |

**PK compuesta:** `(user_id, card_id)`.
