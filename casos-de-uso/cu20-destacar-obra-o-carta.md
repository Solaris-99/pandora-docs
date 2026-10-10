# CU20 — Destacar obra o carta

**Actor principal:** Usuario autenticado.

## Flujo principal

1. El usuario abre su perfil.
2. Selecciona una obra propia o una carta que posee.
3. Pulsa "Destacar".
4. Se registra la relación de favorito/destacado.
5. El perfil muestra el elemento en su sección correspondiente.

## Alternativo

- **A1:** El usuario no es propietario de la obra -> rechazar.
- **A2:** El usuario no posee la carta -> rechazar.
- **A3:** Ya está destacada -> operación idempotente.
