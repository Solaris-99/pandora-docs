# Endpoints — Administración y Apelaciones

| Método | Endpoint | Uso | Auth |
|---|---|---|---|
| POST | `/roles/moderators/:userId` | Otorgar rol `moderator` | Admin |
| DELETE | `/roles/moderators/:userId` | Revocar rol `moderator` | Admin |
| POST | `/moderation/reports/:id/reopen` | Reabrir un reporte resuelto (reversión directa) | Admin |
| POST | `/artworks/:id/appeals` | Apelar la eliminación de una obra propia | Auth |
| POST | `/comments/:id/appeals` | Apelar la eliminación de un comentario propio | Auth |
| POST | `/users/me/ban-appeal` | Apelar el propio baneo | Auth (permitido para cuenta baneada) |
| GET | `/users/me/appeals` | Consultar apelaciones propias | Auth (permitido para cuenta baneada) |
| GET | `/moderation/appeals` | Listar apelaciones pendientes (excluye las que el revisor no puede resolver) | Moderator/Admin |
| GET | `/moderation/appeals/:id` | Detalle de una apelación | Moderator/Admin |
| POST | `/moderation/appeals/:id/resolve` | Aprobar o rechazar una apelación | Moderator/Admin |
| POST | `/artworks/admin-upload` | Cargar directamente obra + carta ya convertida ([CU29](/casos-de-uso/cu29-cargar-carta-directamente.md)) | Admin |
| POST | `/conversion/run` | Disparar manualmente el ciclo de conversión ([CU30](/casos-de-uso/cu30-disparar-el-ciclo-de-conversion-manualmente.md)), pensado para demos | Admin |

`POST /moderation/reports/:id/reopen` es el único endpoint de `/moderation/*` restringido a `admin` exclusivamente; el resto de `/moderation/*` acepta `moderator` o `admin` por igual. `POST /artworks/admin-upload` y `POST /conversion/run` también son exclusivos de `admin`.
