# Lógica de selección del rival (batalla)

El endpoint de rival debe cumplir las restricciones de negocio sin asumir que se trata de una partida multijugador.

```text
GET /battle/opponent
        |
        v
Seleccionar carta válida aleatoria
        |
        v
Devolver datos necesarios
```

El endpoint no debe crear una "partida" persistente. La separación entre backend y cliente es especialmente importante en el juego: el backend mantiene únicamente la autoridad necesaria sobre inventario y selección del rival, mientras la simulación de batalla no se considera información de negocio persistente.
