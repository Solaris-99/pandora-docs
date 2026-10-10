# CU15 — Conversión de obra a carta

**Actor técnico:** CRON / sistema
**Precondición:** Obra elegible, periodo de una semana terminado y `conversion_request` habilitado.
**Caso de prueba:** [CP06 — Conversión de obra a carta](/casos-de-prueba/cp06-conversion-de-obra-a-carta.md)

## Flujo principal

1. El CRON se ejecuta periódicamente.
2. Consulta obras cuyo período de calificación expiró.
3. Filtra obras que solicitaron conversión.
4. Filtra obras cuyo `conversion_status` indica que todavía están pendientes.
5. Obtiene calificaciones, emociones y comentarios necesarios.
6. Ejecuta el algoritmo de conversión.
7. Determina ataque, defensa, HP, velocidad y rareza.
8. Crea la carta asociada.
9. Actualiza el estado de conversión.
10. Genera una notificación de fin de calificación/conversión cuando corresponda.

## Alternativo

- **A1:** Obra no elegible -> se ignora.
- **A2:** Ya existe carta -> marcar como convertida sin duplicar.
- **A3:** Error al calcular -> registrar error y mantener estado reintentable.
- **A4:** Error de DB durante transacción -> rollback.

## Observación

NestJS cuenta con un Scheduler, utilizar esto antes que el propio cron o scheduler. Ver también [CU30](cu30-disparar-el-ciclo-de-conversion-manualmente.md), que dispara esta misma rutina bajo demanda.
