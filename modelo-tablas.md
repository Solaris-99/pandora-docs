# Pandora — Tablas y Diagrama de Tablas

> Documento derivado de `pandora-diseño-general.md` (que se mantiene como referencia consolidada). Cubre el esquema físico de base de datos: tablas, columnas, tipos, constraints, enums e índices recomendados. El modelo conceptual (sin tipos de dato) vive en `pandora-clases-dominio.md`; la realización en entidades TypeORM (con operaciones) en `pandora-clases-diseno.md`.
>
> No se incluye diagrama gráfico en esta versión — la lista de relaciones (sección 3) es su equivalente textual (notación entidad–relación en texto).

La definición inicial incluye usuarios, artworks, comentarios, ratings, cartas, inventario, paquetes, reportes, follows, favoritos y tags. Se agregan además las estructuras necesarias para notificaciones, moderación, verificación de email y recuperación de contraseña.

[Diagrama](https://dbdiagram.io/d/Pandora-6a9ee57bff72c756bcfca7b5)

---


# 1. Tablas

## 1.1 users

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
| created_at | timestamp | NOT NULL |
| updated_at | timestamp | nullable |

### Constraints

- `PRIMARY KEY (id)`
- `UNIQUE (username)`
- `UNIQUE (email)`
- `CHECK (char_length(username) <= 30)`
- `CHECK (char_length(email) <= 50)`
- `password_hash` puede ser NULL sólo para proveedores de autenticación externa.

---

## 1.2 roles / user_roles

Se separan usuarios y roles para evitar una única columna rígida y permitir crecimiento futuro.

### roles

| Campo | Tipo | Reglas |
|---|---|---|
| id | bigint | PK |
| name | enum/varchar | UNIQUE, NOT NULL |
| description | varchar | nullable |

### Enum `role_name`

```text
user
moderator
admin
```

### user_roles

| Campo | Tipo | Reglas |
|---|---|---|
| user_id | bigint | FK users.id, NOT NULL |
| role_id | bigint | FK roles.id, NOT NULL |

**PK compuesta:** `(user_id, role_id)`. Evita roles duplicados y permite que un usuario posea más de un rol.

---

## 1.3 artworks

| Campo                    | Tipo         | Reglas                  |
| ------------------------ | ------------ | ----------------------- |
| id                       | bigint       | PK                      |
| title                    | varchar(50)  | NOT NULL                |
| description              | varchar(300) | nullable                |
| author_id                | bigint       | FK users.id, NOT NULL   |
| image_original_url       | varchar      | NOT NULL                |
| image_medium_url         | varchar      | NOT NULL                |
| image_thumbnail_url      | varchar      | NOT NULL                |
| image_public_id          | varchar      | nullable (id de Cloudinary) |
| conversion_request       | boolean      | NOT NULL, default true  |
| conversion_status        | enum         | NOT NULL                |
| qualification_started_at | timestamp    | NOT NULL                |
| deleted                  | boolean      | NOT NULL, default false |
| created_at               | timestamp    | NOT NULL                |
| updated_at                | timestamp    | nullable                |

### Enum `conversion_status`

```text
pending
processing
converted
skipped
failed
```

Interpretación: `pending` todavía no procesada · `processing` tomada por un proceso de conversión · `converted` ya generó su carta · `skipped` no debe convertirse · `failed` ocurrió un error y puede quedar disponible para reintento controlado.

### Constraints

- `title` obligatorio y máximo 50 caracteres.
- `description` máximo 300 caracteres.
- `author_id` debe referenciar un usuario existente.

---

## 1.4 artwork_comments

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

### Constraints

- FK a `users`, FK a `artworks`.
- `comment` no puede estar vacío después de trim.
- Longitud máxima: 300.

---

## 1.5 artwork_ratings

| Campo | Tipo | Reglas |
|---|---|---|
| id | bigint | PK |
| artwork_id | bigint | FK artworks.id, NOT NULL |
| voter_id | bigint | FK users.id, NOT NULL |
| stars | smallint | NOT NULL |
| emotion | enum | NOT NULL |
| created_at | timestamp | NOT NULL |
| updated_at | timestamp | nullable |

### Enum `emotion`

```text
joy
fear
anger
serenity
grief
awe
```

### Constraints

- `CHECK (stars BETWEEN 1 AND 5)`.
- `UNIQUE (artwork_id, voter_id)`.
- No se admite valoración sobre obra eliminada (regla de dominio, validada en servicio).

---

## 1.6 cards

| Campo | Tipo | Reglas |
|---|---|---|
| id | bigint | PK |
| artwork_id | bigint | FK artworks.id, UNIQUE |
| attack | integer | NOT NULL, >= 0 |
| defense | integer | NOT NULL, >= 0 |
| hp | integer | NOT NULL, >= 0 |
| speed | integer | NOT NULL, >= 0 |
| rarity | enum | NOT NULL |
| deleted | boolean | NOT NULL, default false |
| created_at | timestamp | NOT NULL |
| updated_at | timestamp | nullable |

### Enum `rarity`

```text
common
uncommon
rare
epic
legendary
```

### Constraints

- `UNIQUE (artwork_id)` garantiza una carta por obra.
- `CHECK ("attack" >= 0 AND "defense" >= 0 AND "hp" >= 0 AND "speed" >= 0)`.
- `rarity` limitada al enum.

---

## 1.7 user_cards

| Campo | Tipo | Reglas |
|---|---|---|
| id | bigint | PK |
| user_id | bigint | FK users.id |
| card_id | bigint | FK cards.id |
| copies | integer | NOT NULL, default 0 |

### Constraints

- `UNIQUE (user_id, card_id)`.
- `CHECK (copies >= 0)`.

El contador se incrementa al recibir repetidas.

---

## 1.8 card_packs

| Campo | Tipo | Reglas |
|---|---|---|
| id | bigint | PK |
| user_id | bigint | FK users.id, UNIQUE |
| amount | integer | NOT NULL, default 0 |
| last_regeneration_at | timestamp | nullable |
| updated_at | timestamp | nullable |

### Constraints

- `UNIQUE (user_id)` para representar un inventario de paquetes por usuario.
- `CHECK (amount BETWEEN 0 AND 10)`.

### Regeneración

```text
elapsed_minutes = floor(now - last_regeneration_at)
regenerated = min(elapsed_minutes, 10 - amount)
```

Cálculo realizado al consultar/modificar los paquetes, en vez de almacenar eventos de regeneración.

---

## 1.9 tags

| Campo | Tipo | Reglas |
|---|---|---|
| id | bigint | PK |
| name | varchar(30) | NOT NULL, UNIQUE |

### Constraints

- `UNIQUE(name)`.
- Normalizado a minúsculas.

---

## 1.10 artwork_tags

| Campo | Tipo | Reglas |
|---|---|---|
| artwork_id | bigint | FK artworks.id |
| tag_id | bigint | FK tags.id |

**PK compuesta:** `(artwork_id, tag_id)`.

---

## 1.11 follows

| Campo | Tipo | Reglas |
|---|---|---|
| follower_id | bigint | FK users.id |
| followed_id | bigint | FK users.id |
| created_at | timestamp | NOT NULL |

**PK compuesta:** `(follower_id, followed_id)`.

### Constraints

- Prohibir seguimiento de uno mismo: `follower_id <> followed_id`.
- Una relación no puede duplicarse.

---

## 1.12 favorite_cards

| Campo | Tipo | Reglas |
|---|---|---|
| user_id | bigint | FK users.id |
| card_id | bigint | FK cards.id |
| created_at | timestamp | NOT NULL |

**PK compuesta:** `(user_id, card_id)`.

---

## 1.13 favorite_artworks

| Campo | Tipo | Reglas |
|---|---|---|
| user_id | bigint | FK users.id |
| artwork_id | bigint | FK artworks.id |
| created_at | timestamp | NOT NULL |

**PK compuesta:** `(user_id, artwork_id)`.

---

## 1.14 reports (artwork_reports / comment_reports)

El sistema de reportes soporta tanto obras como comentarios, en dos tablas paralelas con la misma forma.

### artwork_reports

| Campo | Tipo | Reglas |
|---|---|---|
| id | bigint | PK |
| artwork_id | bigint | FK artworks.id, NOT NULL |
| reporter_id | bigint | FK users.id, NOT NULL |
| reason | enum | NOT NULL |
| comment | varchar(300) | nullable |
| status | enum | NOT NULL |
| resolved_by | bigint | FK users.id, nullable |
| resolution | enum | nullable |
| created_at | timestamp | NOT NULL |
| resolved_at | timestamp | nullable |

### comment_reports

| Campo | Tipo | Reglas |
|---|---|---|
| id | bigint | PK |
| comment_id | bigint | FK artwork_comments.id, NOT NULL |
| reporter_id | bigint | FK users.id, NOT NULL |
| reason | enum | NOT NULL |
| comment | varchar(300) | nullable |
| status | enum | NOT NULL |
| resolved_by | bigint | FK users.id, nullable |
| resolution | enum | nullable |
| created_at | timestamp | NOT NULL |
| resolved_at | timestamp | nullable |

### Enum `report_reason`

```text
content_inappropriate
content_violent
content_sexual
illicit
copyright
plagiarism
spam
harassment
other
```

### Enum `report_status`

```text
pending
in_review
resolved
rejected
overturned
```

`overturned` se alcanza únicamente desde `resolved` (reapertura por admin o apelación aprobada), y sólo cuando la `resolution` original fue punitiva. `rejected` está reservado para un futuro mecanismo de auto-rechazo (p. ej. deduplicación) que ningún flujo actual produce todavía.

### Enum `report_resolution`

```text
dismissed
warning
content_removed
comment_removed
user_banned
```

### Constraints y reglas

- `reporter_id` debe ser usuario autenticado.
- `resolved_by` debe corresponder a moderador/admin.
- Un reporte resuelto debe tener `resolved_at` y `resolved_by`.
- Evitar reportes duplicados idénticos por el mismo usuario y objetivo cuando exista uno pendiente.
- Toda acción de moderación debe quedar auditada.

---

## 1.15 notifications

### Enum `notification_type`

```text
user_follow
follow_upload
qualification_ended
artwork_converted
artwork_comment
artwork_vote_summary
comment_removed
artwork_removed
user_warned
user_banned
artwork_restored
comment_restored
user_unbanned
appeal_resolved
```

El conjunto no es exhaustivo y puede ampliarse.

### Tabla

| Campo | Tipo | Reglas |
|---|---|---|
| id | bigint | PK |
| user_id | bigint | FK users.id, NOT NULL |
| type | enum | NOT NULL |
| title | varchar(100) | NOT NULL |
| content | varchar(500) | NOT NULL |
| is_read | boolean | NOT NULL, default false |
| reference_id | bigint | nullable |
| metadata | jsonb | nullable |
| created_at | timestamp | NOT NULL |

### Constraints

- FK de `user_id` a `users`.
- `title` y `content` no pueden estar vacíos.
- Índice sobre `(user_id, is_read, created_at)` para consultas eficientes.
- `reference_id`/`metadata` son de propósito general (apilado de votos, referencia a recurso de moderación) — ver `pandora-clases-diseno.md` §12.

---

## 1.16 email_verification_tokens

| Campo | Tipo | Reglas |
|---|---|---|
| id | bigint | PK |
| user_id | bigint | FK users.id |
| token_hash | varchar | NOT NULL, UNIQUE |
| expires_at | timestamp | NOT NULL |
| used_at | timestamp | nullable |
| created_at | timestamp | NOT NULL |

### Reglas

- No se almacena el token de verificación en texto plano (hash SHA-256).
- Expiración obligatoria — 24 horas.
- Un token usado no puede reutilizarse.

---

## 1.17 password_reset_tokens

Misma estructura que `email_verification_tokens`. Tabla separada en lugar de reutilizar esa tabla, para no mezclar dos propósitos distintos (activar una cuenta vs. recuperar acceso a una) bajo el mismo modelo.

| Campo | Tipo | Reglas |
|---|---|---|
| id | bigint | PK |
| user_id | bigint | FK users.id |
| token_hash | varchar | NOT NULL, UNIQUE |
| expires_at | timestamp | NOT NULL |
| used_at | timestamp | nullable |
| created_at | timestamp | NOT NULL |

### Reglas

- No se almacena el token de recuperación en texto plano (mismo hash que el resto de los tokens opacos del sistema).
- Expiración obligatoria — **1 hora**, más corta que las 24 horas de `email_verification_tokens` por ser un token más sensible.
- Un token usado no puede reutilizarse.
- Al emitir un token nuevo para un usuario, cualquier token previo sin usar de ese usuario se invalida.

---

## 1.18 refresh_tokens

| Campo | Tipo | Reglas |
|---|---|---|
| id | bigint | PK |
| user_id | bigint | FK users.id |
| token_hash | varchar | NOT NULL, UNIQUE |
| expires_at | timestamp | NOT NULL |
| revoked_at | timestamp | nullable |
| created_at | timestamp | NOT NULL |

### Reglas

- No se almacena el refresh token en texto plano (hash SHA-256).
- Se rota en cada uso: el token consumido queda `revoked_at` y se emite uno nuevo.
- Se revoca íntegramente al cerrar sesión y, en bloque, al restablecer la contraseña del usuario.

---

## 1.19 oauth_accounts

Para Google OAuth (y, a futuro, otros proveedores).

| Campo | Tipo | Reglas |
|---|---|---|
| id | bigint | PK |
| user_id | bigint | FK users.id |
| provider | varchar | NOT NULL |
| provider_user_id | varchar | NOT NULL |

**Constraint recomendado:** `UNIQUE(provider, provider_user_id)`.

---

## 1.20 appeals

Tabla única, no dividida por tipo de contenido: toda apelación remite a un único reporte ya resuelto (`report_target_type` + `report_id`, mismo discriminador de `report_target_type` usado en 1.14), y ese reporte ya indica mediante su `resolution` si la sanción fue sobre una obra, un comentario o un usuario baneado.

| Campo | Tipo | Reglas |
|---|---|---|
| id | bigint | PK |
| report_target_type | enum | NOT NULL — mismo discriminador que reports (`artwork`, `comment`) |
| report_id | bigint | NOT NULL — referencia lógica a `artwork_reports.id` o `comment_reports.id` según `report_target_type` |
| appellant_id | bigint | FK users.id, NOT NULL |
| reason | enum | NOT NULL |
| detail | varchar(500) | nullable |
| status | enum | NOT NULL, default `pending` |
| reviewed_by | bigint | FK users.id, nullable |
| reviewed_at | timestamp | nullable |
| created_at | timestamp | NOT NULL |

### Enum `appeal_reason`

```text
excessive_punishment
mistaken_identity
missing_context
false_report
other
```

### Enum `appeal_status`

```text
pending
approved
rejected
```

### Constraints y reglas

- `UNIQUE(report_target_type, report_id)`: un reporte sólo puede apelarse una vez, sin importar el resultado.
- Un baneo de usuario se representa igual que una apelación de obra/comentario: `report_target_type`/`report_id` apuntan al reporte cuya `resolution` fue `user_banned`; no existe un tercer valor de `report_target_type` para "usuario".
- `reviewed_by` y `reviewed_at` sólo se completan al resolver (`approved`/`rejected`).
- El usuario que revisa (`reviewed_by`) no puede ser el mismo que aparece en `resolved_by` del reporte referenciado, salvo que en ese momento exista exactamente un miembro de personal de moderación (`moderator` + `admin` combinados).

---

# 2. Diagrama de tablas (notación textual)

```text
users 1 ─── N artworks
users 1 ─── N artwork_comments
users 1 ─── N artwork_ratings
artworks 1 ─── N artwork_comments
artworks 1 ─── N artwork_ratings
artworks 1 ─── 1 cards
users N ─── N cards                    (user_cards)
users 1 ─── 1 card_packs
users N ─── N users                    (follows)
users N ─── N artworks                 (favorite_artworks)
users N ─── N cards                    (favorite_cards)
artworks N ─── N tags                  (artwork_tags)
users 1 ─── N notifications
artworks 1 ─── N artwork_reports
artwork_comments 1 ─── N comment_reports
users 1 ─── N reports (artwork_reports/comment_reports, como reporter/resolver)
users 1 ─── N email_verification_tokens
users 1 ─── N password_reset_tokens
users 1 ─── N refresh_tokens
users 1 ─── N oauth_accounts
users N ─── N roles                    (user_roles)
users 1 ─── N appeals                  (appellant_id)
users 1 ─── N appeals                  (reviewed_by, nullable)
artwork_reports  1 ─── 0..1 appeals    (report_target_type = artwork)
comment_reports  1 ─── 0..1 appeals    (report_target_type = comment)
```

---

# 3. Índices recomendados

Para PostgreSQL se recomienda considerar, como mínimo:

```text
users(email)
users(username)
artworks(author_id, created_at)
artworks(deleted, created_at)
artwork_comments(artwork_id, created_at)
artwork_ratings(artwork_id)
user_cards(user_id, card_id)
card_packs(user_id)
follows(follower_id)
follows(followed_id)
notifications(user_id, is_read, created_at)
artwork_reports(status, created_at)
comment_reports(status, created_at)
appeals(status, created_at)
appeals(report_target_type, report_id)  -- ya cubierto por el UNIQUE
email_verification_tokens(token_hash)   -- ya cubierto por el UNIQUE
password_reset_tokens(token_hash)       -- ya cubierto por el UNIQUE
refresh_tokens(token_hash)              -- ya cubierto por el UNIQUE
```

Los índices exactos deben revisarse mediante consultas reales y `EXPLAIN ANALYZE` cuando el volumen de datos sea representativo. Esta revisión es un ítem pendiente de la Fase 9/Hardening (ver `pandora-trazabilidad.md`).

---

## Documentos relacionados

- `pandora-clases-dominio.md` — modelo conceptual del que deriva este esquema.
- `pandora-clases-diseno.md` — entidades TypeORM (con operaciones) que mapean estas tablas.
- `pandora-requerimientos.md` — reglas de negocio que motivan cada constraint.
