# CP06 — Conversión de obra a carta (CU15)

**Caso de uso:** [CU15 — Conversión de obra a carta](/casos-de-uso/cu15-conversion-de-obra-a-carta.md)

- **Precondición:** obra con período de calificación terminado y `conversion_request = true`.
- **Pasos:** el scheduler busca obras elegibles → obtiene valoraciones/emociones → ejecuta el algoritmo de conversión → crea la `Card` → marca la obra `converted` → notifica.
- **Resultado esperado:** exactamente una `Card` por obra elegible; `conversion_status = converted`.
- **Casos negativos:** obra no elegible → se ignora (A1) · carta ya existente → se marca `converted` sin duplicar, protegido por `UNIQUE(artwork_id)` (A2) · error de cálculo → estado reintentable (A3) · error de transacción → rollback (A4).
- **Cobertura automatizada:** `conversion-scheduler.service.spec.ts` cubre los 5 escenarios anteriores explícitamente (ciclo sin candidatos, conversión exitosa, carta duplicada, claim perdido ante otra ejecución concurrente, y fallo con marcado `failed`).
