# CU25 — Gestionar rol de moderador

**Actor principal:** Admin
**Precondición:** El actor posee el rol `admin`.
**Postcondición:** El usuario objetivo gana o pierde el rol `moderator`.
**Caso de prueba:** [CP17 — Gestionar rol de moderador](/casos-de-prueba/cp17-gestionar-rol-de-moderador.md)

## Flujo principal

1. El admin abre el perfil público del usuario objetivo.
2. Pulsa "Otorgar rol de moderador" o "Quitar rol de moderador", según corresponda.
3. Se otorga o revoca el rol `moderator`.
4. El sistema actualiza los roles del usuario objetivo.
5. Se confirma la acción.

## Alternativo

- **A1:** El usuario ya tiene (u otorgar) o ya no tiene (revocar) el rol -> operación idempotente, no es un error.
- **A2:** Usuario objetivo inexistente -> 404.
- **A3:** Actor sin rol `admin` -> 403.
- **A4:** El rol `admin` en sí mismo no se otorga ni revoca por este medio -> fuera de alcance de este caso de uso. La acción no se ofrece en el perfil de otro admin ni en el propio.

## Observaciones

Disponible sólo en el portal web: la app Android no incluye herramientas de administración.
