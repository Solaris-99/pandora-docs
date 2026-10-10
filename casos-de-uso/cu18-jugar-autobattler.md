# CU18 — Jugar autobattler

**Actor principal:** Usuario autenticado.
**Caso de prueba:** [CP09 — Jugar autobattler](/casos-de-prueba/cp09-jugar-autobattler.md)

## Flujo principal

1. El usuario abre el módulo de juego.
2. Selecciona una carta que posee.
3. El cliente solicita una carta rival.
4. El backend selecciona una carta rival válida de forma aleatoria.
5. Devuelve ambas cartas, propia y rival, con estadísticas completas para la simulación.
6. El cliente ejecuta la batalla.
7. El cliente muestra turnos, daño, vida restante y resultado.
8. La sesión termina.

## Alternativo

- **A1:** Usuario no posee la carta seleccionada -> 403/400.
- **A2:** Carta no existe -> 404.
- **A3:** No existen cartas válidas para rival -> error controlado.

## Observaciones

No almacenar resultado, replay, victoria, derrota ni historial de batallas en backend en esta versión.
