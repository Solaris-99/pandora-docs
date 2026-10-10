# Colección (relación Usuario–Carta)

No es una entidad autónoma sino la relación de posesión entre un Usuario y una Carta, con un contador de copias.

- **Atributos conceptuales:** usuario, carta, cantidad de copias poseídas.
- **Responsabilidades:** determinar si las estadísticas completas de una Carta son visibles para ese Usuario (visible sólo si copias > 0); incrementarse al recibir una copia repetida desde un Paquete.
