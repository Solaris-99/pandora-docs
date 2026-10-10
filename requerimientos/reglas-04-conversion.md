# 5.4 Conversión

1. El CRON/scheduler busca obras cuyo período de calificación terminó.
2. Sólo procesa obras elegibles para conversión.
3. Sólo procesa obras que aún no fueron convertidas.
4. Se calcula las estadísticas base: `( b * e ) * (r + s)` donde b es una base dependiendo del stat, r es la rareza, s es el promedio de estrellas y e es el bonus de la emoción. Ver las tablas en [5.10](reglas-10-estadisticas-base-emociones-y-rarezas.md).
5. Guarda la carta generada.
6. Actualiza el estado de conversión de la obra.
7. El proceso debe ser idempotente.
8. Un admin puede disparar el mismo ciclo manualmente, fuera del horario del CRON (pensado para demos) — no es un proceso distinto, reutiliza exactamente la misma rutina.
