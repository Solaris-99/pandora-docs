# CU27 — Apelar una sanción

**Actor principal:** Usuario autenticado afectado (autor de la obra/comentario eliminado, o usuario baneado apelando su propia cuenta)
**Precondición:** Existe un reporte resuelto con sanción punitiva sobre su obra, comentario o cuenta, y esa sanción todavía no fue apelada.
**Postcondición:** Se registra una apelación en estado `pending`, visible para el personal de moderación.
**Caso de prueba:** [CP19 — Apelar una sanción](/casos-de-prueba/cp19-apelar-una-sancion.md)

## Flujo principal

1. El usuario ve que su obra o comentario fue eliminado, por la notificación que recibe, o que su cuenta fue suspendida, por el aviso que muestra la aplicación.
2. Pulsa "Apelar" en esa notificación o aviso.
3. Selecciona un motivo predefinido (sanción excesiva o incorrecta, identidad equivocada, falta de contexto, reporte falso, otro).
4. Opcionalmente agrega un texto explicando su caso.
5. Envía la apelación.
6. El sistema la registra en estado `pending`.

## Alternativo

- **A1:** Ya existe una apelación previa para esa misma sanción -> 409, no se admite una segunda.
- **A2:** No existe una sanción punitiva apelable sobre ese contenido/cuenta -> 404.
- **A3:** El usuario no es el autor del contenido ni el usuario baneado -> 403.
- **A4:** El usuario apelando su propio baneo está autenticado pero baneado -> se le permite igual: apelar el propio baneo, consultar sus apelaciones y sus notificaciones son las únicas operaciones de escritura/lectura que una cuenta baneada conserva.

## Observaciones

La notificación no indica si la sanción ya fue apelada, así que la opción sigue visible y un segundo intento termina en A1. Las notificaciones creadas antes de que el backend incluyera la referencia al contenido (`data`) no ofrecen la opción.
