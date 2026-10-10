# appeals

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

## Enum `appeal_reason`

```text
excessive_punishment
mistaken_identity
missing_context
false_report
other
```

## Enum `appeal_status`

```text
pending
approved
rejected
```

## Constraints y reglas

- `UNIQUE(report_target_type, report_id)`: un reporte sólo puede apelarse una vez, sin importar el resultado.
- Un baneo de usuario se representa igual que una apelación de obra/comentario: `report_target_type`/`report_id` apuntan al reporte cuya `resolution` fue `user_banned`; no existe un tercer valor de `report_target_type` para "usuario".
- `reviewed_by` y `reviewed_at` sólo se completan al resolver (`approved`/`rejected`).
- El usuario que revisa (`reviewed_by`) no puede ser el mismo que aparece en `resolved_by` del reporte referenciado, salvo que en ese momento exista exactamente un miembro de personal de moderación (`moderator` + `admin` combinados).
