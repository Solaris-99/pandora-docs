# CU19 — Seguir usuario

**Actor principal:** Usuario autenticado.

## Flujo principal

1. El usuario abre un perfil.
2. Pulsa "Seguir".
3. Se crea la relación.
4. Se genera una notificación al usuario seguido si corresponde.

## Alternativo

- **A1:** Ya existe la relación -> operación idempotente.
- **A2:** Usuario intenta seguirse a sí mismo -> rechazar.
- **A3:** Usuario objetivo inexistente/baneado -> rechazar según política.
