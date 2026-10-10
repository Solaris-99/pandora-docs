# Módulo `scheduler`

## Servicio: `ConversionSchedulerService`

`handleCron()` (`@Cron`, dispara diariamente), `runConversionCycle()` (la rutina real, invocable independientemente del reloj — ver [diseño de arquitectura, disparo manual](/diseno-arquitectura/conversion.md#disparo-manual-admin-demo)).

## Controlador de soporte: `ConversionSchedulerController`

Expone `POST /conversion/run` delegando directamente en `runConversionCycle()`.
