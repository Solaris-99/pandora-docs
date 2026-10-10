# CP22 — Disparar el ciclo de conversión manualmente (Admin) (CU30)

**Caso de uso:** [CU30 — Disparar el ciclo de conversión manualmente (Admin)](/casos-de-uso/cu30-disparar-el-ciclo-de-conversion-manualmente.md)

- **Precondición:** actor `admin`.
- **Pasos:** disparar la ejecución manual → el backend ejecuta `runConversionCycle()` (la misma rutina que [CU15](/casos-de-uso/cu15-conversion-de-obra-a-carta.md)) → devuelve el resumen.
- **Resultado esperado:** `{processed, converted, failed, skipped}`; mismo efecto que una corrida real del CRON.
- **Casos negativos:** sin obras elegibles → resumen en cero, no error (A1) · actor sin `admin` → 403 (A2).
- **Verificación en vivo realizada:** confirmado por HTTP real; rechazo 403 para un usuario sin rol `admin` también verificado en vivo.
