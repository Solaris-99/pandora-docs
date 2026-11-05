# CU20 — Destacar obra o carta

**Actor principal:** Usuario autenticado.
**Caso de prueba:** [CP11 — Destacar obra o carta](/casos-de-prueba/cp11-destacar-obra-o-carta.md)

## Flujo principal

1. El usuario abre el detalle de una obra propia o de una carta que posee. En la web también puede hacerlo directamente desde "Mis obras" o "Colección".
2. Pulsa "Destacar".
3. Se registra la relación de favorito/destacado.
4. El elemento pasa a figurar en sus destacados, accesibles desde su perfil.

## Alternativo

- **A1:** El usuario no es propietario de la obra -> rechazar.
- **A2:** El usuario no posee la carta -> rechazar.
- **A3:** Ya está destacada -> operación idempotente.
- **A4:** El usuario pulsa nuevamente sobre un elemento destacado -> se quita de sus destacados.
