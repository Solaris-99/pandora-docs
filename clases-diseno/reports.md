# Módulo `reports`

## Entidades

- **ArtworkReport**: `id`, `artwork_id`, `reporter_id`, `reason: ReportReason`, `comment: string | null`, `status: ReportStatus`, `resolved_by: bigint | null`, `resolution: ReportResolution | null`, `created_at`, `resolved_at`.
- **CommentReport**: misma forma, referenciando `comment_id`.

## Servicio: `ReportsService`

`reportArtwork(reporterId, artworkId, dto)`, `reportComment(reporterId, commentId, dto)` — validan duplicados pendientes antes de crear.
