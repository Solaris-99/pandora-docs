# Módulo `scheduler`

## Servicio: `ConversionSchedulerService`

`handleCron()` (`@Cron`, dispara diariamente), `runConversionCycle()` (la rutina real, invocable independientemente del reloj — ver [diseño de arquitectura, disparo manual](/diseno-arquitectura/conversion.md#disparo-manual-admin-demo)).

## Controlador de soporte: `ConversionSchedulerController`

Expone `POST /conversion/run` delegando directamente en `runConversionCycle()`.

## Servicio: `AccountCleanupSchedulerService` (CU05)

`handleCron()` (`@Cron`, mismo horario que `ConversionSchedulerService`: 00:00 diariamente), `runCleanupCycle()` (rutina real, invocable independientemente del reloj — mismo patrón de separación que la de conversión).

Busca `User` con `email_verified = false` y `created_at` anterior a 72 horas, y los elimina **físicamente** (no es un soft-delete, a diferencia del resto del sistema — no tiene sentido conservar una cuenta que nunca llegó a confirmarse). No hace falta cascadear a mano: todas las FK hacia `users.id` ya están declaradas `ON DELETE CASCADE`/`SET NULL` (tokens de auth, obras, follows, favoritos, notificaciones, etc. — ver [clases de diseño §users](/clases-diseno/users.md)), así que un `DELETE` simple por repositorio alcanza. Si falla la eliminación de una cuenta puntual, se registra el error y se sigue con el resto del lote (no aborta el ciclo completo).
