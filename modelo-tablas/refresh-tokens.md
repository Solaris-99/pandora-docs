# refresh_tokens

| Campo | Tipo | Reglas |
|---|---|---|
| id | bigint | PK |
| user_id | bigint | FK users.id |
| token_hash | varchar | NOT NULL, UNIQUE |
| expires_at | timestamp | NOT NULL |
| revoked_at | timestamp | nullable |
| created_at | timestamp | NOT NULL |

## Reglas

- No se almacena el refresh token en texto plano (hash SHA-256).
- Se rota en cada uso: el token consumido queda `revoked_at` y se emite uno nuevo.
- Se revoca íntegramente al cerrar sesión y, en bloque, al restablecer la contraseña del usuario.
