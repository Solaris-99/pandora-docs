# email_verification_tokens

| Campo | Tipo | Reglas |
|---|---|---|
| id | bigint | PK |
| user_id | bigint | FK users.id |
| token_hash | varchar | NOT NULL, UNIQUE |
| expires_at | timestamp | NOT NULL |
| used_at | timestamp | nullable |
| created_at | timestamp | NOT NULL |

## Reglas

- No se almacena el token de verificación en texto plano (hash SHA-256).
- Expiración obligatoria — 24 horas.
- Un token usado no puede reutilizarse.
