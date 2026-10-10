# Reporte (Report)

Una denuncia sobre una Obra o un Comentario, presentada por un Usuario y eventualmente resuelta por un moderador o admin. Conceptualmente una única clase, aunque a nivel de datos se distingue por el tipo de objetivo (ver [modelo de tablas](/modelo-tablas/reports.md)).

- **Atributos conceptuales:** objetivo denunciado (Obra o Comentario), denunciante, motivo, comentario opcional, estado (`pending`/`in_review`/`resolved`/`rejected`/`overturned`), resolución aplicada (si fue resuelto), quién lo resolvió, cuándo.
- **Responsabilidades:** disparar el flujo de moderación; registrar auditoría de quién tomó qué acción y cuándo; habilitar, si su resolución fue punitiva, que el afectado presente una Apelación o que un admin lo reabra directamente.
- **Relaciones:** referencia 1 Obra o 1 Comentario · presentado por 1 Usuario (denunciante) · resuelto por 0..1 Usuario (moderador/admin) · referenciado por 0..1 Apelación.
