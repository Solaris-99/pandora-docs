# CU30 — Disparar el ciclo de conversión manualmente (Admin)

**Actor principal:** Admin
**Precondición:** El actor posee el rol `admin`.
**Postcondición:** Toda obra elegible cuyo período de calificación ya terminó queda procesada, igual que si hubiera corrido el CRON.
**Caso de prueba:** [CP22 — Disparar el ciclo de conversión manualmente (Admin)](/casos-de-prueba/cp22-disparar-el-ciclo-de-conversion-manualmente.md)

## Flujo principal

1. El admin dispara la ejecución manual (pensado para demos, sin esperar al horario del CRON).
2. El backend ejecuta exactamente la misma rutina que usa el scheduler ([CU15](cu15-conversion-de-obra-a-carta.md)): busca obras elegibles, las convierte y marca las no solicitadas como `skipped`.
3. Devuelve un resumen: cantidad procesada, convertida, fallida y marcada como `skipped`.

## Alternativo

- **A1:** No hay obras elegibles -> se devuelve el resumen en cero, no es un error.
- **A2:** Actor sin rol `admin` -> 403.

## Observaciones

No es un proceso alternativo al CRON: es el mismo `runConversionCycle` ([CU15](cu15-conversion-de-obra-a-carta.md)), sólo que disparado bajo demanda en lugar de por el reloj.
