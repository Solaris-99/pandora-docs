# email_change_tokens

| Campo | Tipo | Reglas |
|---|---|---|
| id | bigint | PK |
| user_id | bigint | FK users.id, ON DELETE CASCADE |
| new_email | varchar(50) | NOT NULL — el email propuesto, todavía sin aplicar a `users.email` |
| token_hash | varchar | NOT NULL, UNIQUE |
| expires_at | timestamp | NOT NULL |
| used_at | timestamp | nullable |
| created_at | timestamp | NOT NULL |

## Reglas

- No se almacena el token de confirmación en texto plano (hash SHA-256) — mismo patrón que `email_verification_tokens`/`password_reset_tokens`.
- Expiración obligatoria — 24 horas.
- Un token usado no puede reutilizarse.
- `new_email` sólo se copia a `users.email` al confirmarse (CU07); hasta entonces el email anterior sigue siendo el activo.
- Solicitar un nuevo cambio antes de confirmar el anterior invalida (marca `used_at`) cualquier token pendiente previo del mismo usuario.
