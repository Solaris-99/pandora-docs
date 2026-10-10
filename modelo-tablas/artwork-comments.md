# artwork_comments

| Campo | Tipo | Reglas |
|---|---|---|
| id | bigint | PK |
| author_id | bigint | FK users.id, NOT NULL |
| artwork_id | bigint | FK artworks.id, NOT NULL |
| comment | varchar(300) | NOT NULL |
| deleted | boolean | NOT NULL, default false |
| created_at | timestamp | NOT NULL |
| updated_at | timestamp | nullable |

Las obras continúan siendo comentables después de finalizar el período de calificación.

## Constraints

- FK a `users`, FK a `artworks`.
- `comment` no puede estar vacío después de trim.
- Longitud máxima: 300.
