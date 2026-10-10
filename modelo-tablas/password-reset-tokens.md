# password_reset_tokens

Misma estructura que `email_verification_tokens`. Tabla separada en lugar de reutilizar esa tabla, para no mezclar dos propósitos distintos (activar una cuenta vs. recuperar acceso a una) bajo el mismo modelo.

| Campo | Tipo | Reglas |
|---|---|---|
| id | bigint | PK |
| user_id | bigint | FK users.id |
| token_hash | varchar | NOT NULL, UNIQUE |
| expires_at | timestamp | NOT NULL |
| used_at | timestamp | nullable |
| created_at | timestamp | NOT NULL |

## Reglas

- No se almacena el token de recuperación en texto plano (mismo hash que el resto de los tokens opacos del sistema).
- Expiración obligatoria — **1 hora**, más corta que las 24 horas de `email_verification_tokens` por ser un token más sensible.
- Un token usado no puede reutilizarse.
- Al emitir un token nuevo para un usuario, cualquier token previo sin usar de ese usuario se invalida.
