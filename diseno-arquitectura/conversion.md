# Conversión de obras a cartas (proceso periódico)

## Objetivo

Evitar conversiones manuales y garantizar que una obra sólo sea convertida después de finalizar su semana de calificación.

## Flujo

```text
CRON/Scheduler
  |
  v
Buscar artworks:
  qualification_started_at >= 7 dias
  conversion_request = true
  conversion_status = pending/failed reintentable
  |
  v
Tomar trabajo
  |
  v
Marcar processing
  |
  v
Obtener ratings + emociones + comentarios
  |
  v
Ejecutar CardConversionService
  |
  +--> attack
  +--> defense
  +--> hp
  +--> speed
  +--> rarity
  |
  v
Crear Card
  |
  v
Marcar converted
  |
  v
Notificación
```

## Estados

```text
(creación) -> pending    [conversionRequest = true, u omitido]
(creación) -> skipped    [conversionRequest = false desde la creación: no hay período que dar]
pending -> processing -> converted
                  \-> failed
pending -> skipped        [conversionRequest pasó a false por una edición, antes del corte semanal]
```

## Idempotencia

Debe existir una protección de base de datos mediante `UNIQUE(artwork_id)` en `cards`. El servicio debe utilizar transacciones para impedir que dos instancias del scheduler creen la misma carta.

## Frecuencia del scheduler

El scheduler se dispara a las 00:00, diariamente, mediante el Scheduler nativo de NestJS (`@nestjs/schedule`).

## Disparo manual (Admin, demo)

La rutina de conversión ([flujo](#flujo), [estados](#estados) e [idempotencia](#idempotencia)) se expone también como método invocable independientemente del disparador de Cron, y un endpoint de administración (`POST /conversion/run`, [CU30](/casos-de-uso/cu30-disparar-el-ciclo-de-conversion-manualmente.md)) la ejecuta bajo demanda. No es una segunda implementación: es la misma lógica, sólo que disparada por una request en lugar de por el reloj — útil para demos donde no tiene sentido esperar a que termine una semana de calificación real.
