# roles / user_roles

Se separan usuarios y roles para evitar una única columna rígida y permitir crecimiento futuro.

## roles

| Campo | Tipo | Reglas |
|---|---|---|
| id | bigint | PK |
| name | enum/varchar | UNIQUE, NOT NULL |
| description | varchar | nullable |

## Enum `role_name`

```text
user
moderator
admin
```

## user_roles

| Campo | Tipo | Reglas |
|---|---|---|
| user_id | bigint | FK users.id, NOT NULL |
| role_id | bigint | FK roles.id, NOT NULL |

**PK compuesta:** `(user_id, role_id)`. Evita roles duplicados y permite que un usuario posea más de un rol.
