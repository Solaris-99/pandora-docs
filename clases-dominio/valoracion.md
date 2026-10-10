# Valoración (Rating / Q2Q)

Un voto que un Usuario emite sobre una Obra durante su período de calificación: una cantidad de estrellas (1 a 5) y una emoción.

- **Atributos conceptuales:** estrellas, emoción, obra, votante.
- **Responsabilidades:** contribuir, junto con el resto de las Valoraciones de una Obra, al cálculo de las estadísticas y la rareza de la futura Carta.
- **Relaciones:** pertenece a 1 Obra · emitida por 1 Usuario.
- **Regla de unicidad:** un Usuario tiene como máximo una Valoración activa por Obra; volver a calificar actualiza la existente en vez de crear una nueva.
