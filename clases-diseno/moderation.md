# Módulo `moderation`

## Entidad: `Appeal`

`id`, `report_target_type: 'artwork' | 'comment'`, `report_id`, `appellant_id`, `reason: AppealReason`, `detail: string | null`, `status: AppealStatus`, `reviewed_by: bigint | null`, `reviewed_at`, `created_at`.

## Servicio: `ModerationService`

`listReports`, `getReportDetail`, `resolveReport(moderatorId, targetType, id, action)` (aplica la sanción — advertencia/eliminación/baneo/descarte — y notifica), `reopenReport(targetType, id)` (CU26, sólo admin), `listAppeals`, `getAppealDetail`, `resolveAppeal(reviewerId, id, action)` (CU28). Método privado `notify(...)` centraliza el envío de notificaciones de moderación con `data: {targetType, targetId}`.

## Servicio: `AppealsService`

`appealArtworkRemoval(userId, artworkId, dto)`, `appealCommentRemoval(userId, commentId, dto)`, y el flujo de apelación de baneo (`POST /users/me/ban-appeal`, expuesto desde `users`).
