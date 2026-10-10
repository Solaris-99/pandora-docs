# notifications

## Enum `notification_type`

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

## Tabla

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

## Constraints

- FK de `user_id` a `users`.
- `title` y `content` no pueden estar vacíos.
- Índice sobre `(user_id, is_read, created_at)` para consultas eficientes.
- `reference_id`/`metadata` son de propósito general (apilado de votos, referencia a recurso de moderación) — ver [clases de diseño, módulo `notifications`](/clases-diseno/notifications.md).
