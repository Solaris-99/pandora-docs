# reports (artwork_reports / comment_reports)

El sistema de reportes soporta tanto obras como comentarios, en dos tablas paralelas con la misma forma.

## artwork_reports

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

## comment_reports

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

## Enum `report_reason`

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

## Enum `report_status`

```text
pending
in_review
resolved
rejected
overturned
```

`overturned` se alcanza únicamente desde `resolved` (reapertura por admin o apelación aprobada), y sólo cuando la `resolution` original fue punitiva. `rejected` está reservado para un futuro mecanismo de auto-rechazo (p. ej. deduplicación) que ningún flujo actual produce todavía.

## Enum `report_resolution`

```text
dismissed
warning
content_removed
comment_removed
user_banned
```

## Constraints y reglas

- `reporter_id` debe ser usuario autenticado.
- `resolved_by` debe corresponder a moderador/admin.
- Un reporte resuelto debe tener `resolved_at` y `resolved_by`.
- Evitar reportes duplicados idénticos por el mismo usuario y objetivo cuando exista uno pendiente.
- Toda acción de moderación debe quedar auditada.
