# oauth_accounts

Para Google OAuth (y, a futuro, otros proveedores).

| Campo | Tipo | Reglas |
|---|---|---|
| id | bigint | PK |
| user_id | bigint | FK users.id |
| provider | varchar | NOT NULL |
| provider_user_id | varchar | NOT NULL |

**Constraint recomendado:** `UNIQUE(provider, provider_user_id)`.
