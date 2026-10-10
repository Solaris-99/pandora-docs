# Notificación (Notification)

Un registro persistente de un acontecimiento relevante para un Usuario.

- **Atributos conceptuales:** destinatario, tipo de evento, título, contenido, si fue leída.
- **Eventos que la generan:** usuario seguido; nuevo upload de un usuario seguido; fin de período de calificación; obra convertida; comentario nuevo sobre una obra; comentario/obra eliminados por moderación (y su reversión); resumen de actividad de votos; advertencia o baneo; resultado de una apelación. El catálogo no es cerrado y puede ampliarse.
- **Relaciones:** pertenece a exactamente 1 Usuario (destinatario).
