# Módulo `notifications`

## Entidad: `Notification`

`id`, `user_id`, `type: NotificationType`, `title: string`, `content: string`, `is_read: boolean`, `reference_id: bigint | null`, `metadata: jsonb | null`, `created_at`.

`reference_id`/`metadata` son de propósito general: se reutilizan tanto para apilar votos (`{voters: [...]}`) como para referenciar el recurso afectado por una acción de moderación (`{targetType, targetId}`).

## Servicio: `NotificationsService`

`create(userId, type, title, content, data?)`, `findAll(userId, pagination)`, `markAsRead(id, userId)`, `markAllAsRead(userId)`, `unreadCount(userId)`. La reconstrucción de `data` en la vista de respuesta distingue por tipo de contenido de `metadata` (type guard sobre `targetType`).
