# Obra (Artwork)

La unidad central de contenido de la plataforma: una pieza de arte digital publicada por un Usuario, que opcionalmente evoluciona a una Carta.

- **Atributos conceptuales:** título, descripción/lore (opcional), imagen (original + versiones optimizadas), si solicita conversión a carta, estado de conversión, fecha de inicio del período de calificación, si fue eliminada.
- **Responsabilidades:** exponerse en el explorador público mientras no esté eliminada; permanecer calificable durante exactamente una semana desde su publicación; permanecer comentable indefinidamente; convertirse en, como máximo, una Carta.
- **Relaciones:** pertenece a 1 Usuario (autor) · tiene N Comentarios · tiene N Valoraciones · está asociada a N Tags · genera 0..1 Carta · es destacada por N Usuarios · es denunciada mediante N Reportes.
- **Regla de estado clave:** una Obra eliminada nunca aparece como contenido público activo, pero conserva su historial (eliminación lógica).
