# Módulo `battle`

## Servicio: `BattleService`

`getOpponent(ownedCardId)`: valida ownership de la carta propia, selecciona una carta rival válida al azar entre todas las cartas existentes, devuelve ambas con estadísticas completas (sin ocultamiento — ver [requerimientos §5.8](/requerimientos/reglas-08-batallas.md) y la clase de dominio [Carta](/clases-dominio/carta.md)).

Utilidad pura: `toOpponentCardView` (mapeo de `Card` a la forma de respuesta del rival, siempre con `stats` visible).
