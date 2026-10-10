# CU05 — Limpieza de cuentas no verificadas

**Actor técnico:** CRON / sistema
**Precondición:** Existen cuentas en estado pendiente de verificación (`email_verified = false`) con más de 72 horas desde su creación.
**Postcondición:** Esas cuentas quedan eliminadas.
**Caso de prueba:** todavía sin caso de prueba dedicado.

## Flujo principal

1. Un job corre periódicamente, todos los días a las 00:00.
2. Busca cuentas con `email_verified = false` y `created_at` anterior a `now() - 72h`.
3. Elimina cada cuenta encontrada.
4. Registra el resultado del ciclo (cantidad eliminada).

## Alternativo

- **A1:** No hay cuentas elegibles -> no se elimina nada, no es un error.
- **A2:** Falla la eliminación de una cuenta puntual -> se registra el error y se continúa con el resto, sin abortar el ciclo completo.

## Observaciones

Esta no es una eliminación lógica: la cuenta y sus datos asociados se eliminan físicamente. No tiene sentido conservar un registro de una cuenta que nunca llegó a confirmarse. Relacionado con [CU01](cu01-registrarse.md) (Registrarse) y [CU02](cu02-verificar-email.md) (Verificar email).
