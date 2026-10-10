# Carta (Card)

El resultado de convertir una Obra al finalizar su período de calificación (o de crearla directamente como admin). Representa la pieza coleccionable y jugable.

- **Atributos conceptuales:** ataque, defensa, vida, velocidad, rareza, obra de origen.
- **Responsabilidades:** ser coleccionada (con copias) por los Usuarios que la obtienen; participar como carta propia o como rival en una Batalla; ocultar sus estadísticas ante quien no la posee.
- **Relaciones:** se genera a partir de exactamente 1 Obra (relación 1 a 1, máximo una Carta por Obra) · es poseída por N Usuarios, cada uno con su propio contador de copias · puede ser destacada por N Usuarios.
