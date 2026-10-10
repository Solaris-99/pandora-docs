# CU16 — Explorar colección/cartas

**Actor principal:** Invitado o usuario autenticado.

## Flujo principal

1. El usuario abre la sección de cartas.
2. Aplica filtros.
3. El sistema pagina los resultados.
4. El usuario selecciona una carta.
5. Se muestra información pública de la carta.
6. Si el usuario la posee, se muestran sus estadísticas completas.
7. Si no la posee, se ocultan las estadísticas según la regla del álbum.

## Alternativo

- **A1:** Carta no poseída -> estadística oculta.
- **A2:** Carta inexistente -> 404.
- **A3:** El usuario solicita más de una página -> devolver `nextCursor` o metadata equivalente.
