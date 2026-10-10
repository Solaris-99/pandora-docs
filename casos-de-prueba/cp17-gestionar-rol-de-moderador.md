# CP17 — Gestionar rol de moderador (CU25)

**Caso de uso:** [CU25 — Gestionar rol de moderador](/casos-de-uso/cu25-gestionar-rol-de-moderador.md)

- **Precondición:** actor con rol `admin`.
- **Pasos:** seleccionar usuario objetivo → otorgar o revocar `moderator` → confirmar.
- **Resultado esperado:** roles del usuario objetivo actualizados.
- **Casos negativos:** operación repetida → idempotente (A1) · usuario objetivo inexistente → 404 (A2) · actor sin rol `admin` → 403 (A3) · intento de otorgar/revocar `admin` por esta vía → fuera de alcance (A4).
