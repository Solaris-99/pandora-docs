# users

| Campo | Tipo | Reglas |
|---|---|---|
| id | bigint/int | PK |
| username | varchar(30) | NOT NULL, UNIQUE |
| email | varchar(50) | NOT NULL, UNIQUE |
| password_hash | varchar | NULL para cuentas OAuth |
| email_verified | boolean | NOT NULL, default false |
| status | enum | active, suspended, banned, pending_verification |
| bio | varchar(300) | nullable |
| avatar_url | varchar | nullable |
| avatar_public_id | varchar | nullable — `public_id` de Cloudinary del avatar actual (CU31); null si nunca se subió uno por esta vía (incluye el avatar importado desde Google) |
| created_at | timestamp | NOT NULL |
| updated_at | timestamp | nullable |

## Constraints

- `PRIMARY KEY (id)`
- `UNIQUE (username)`
- `UNIQUE (email)`
- `CHECK (char_length(username) <= 30)`
- `CHECK (char_length(email) <= 50)`
- `password_hash` puede ser NULL sólo para proveedores de autenticación externa.
