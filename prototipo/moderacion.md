# Moderación y apelaciones

Gestión de denuncias por el equipo de moderación, y apelación de sanciones por parte de los usuarios.

## Cola de denuncias

**Ruta:** `/moderation` | **Actor:** Moderador

Listado paginado de denuncias de obras y comentarios, con filtro por estado. Filtrando por resueltas, un admin llega a las que puede reabrir.

**Casos de uso:** [CU23 — Administrar denuncias](/casos-de-uso/cu23-administrar-denuncias.md) · [CU26 — Reabrir un reporte resuelto](/casos-de-uso/cu26-reabrir-un-reporte-resuelto.md)

| Escritorio | Móvil |
|---|---|
| ![Cola de denuncias en escritorio](img/desktop/moderacion-denuncias.webp ':size=520') | ![Cola de denuncias en móvil](img/mobile/moderacion-denuncias.webp ':size=180') |

---

## Denuncia de obra

**Ruta:** `/moderation/artwork/:id` | **Actor:** Moderador

Muestra la obra denunciada, el motivo y el comentario del denunciante. Las acciones son descartar, advertir, eliminar la obra o banear al usuario.

**Casos de uso:** [CU21 — Denunciar obra](/casos-de-uso/cu21-denunciar-obra.md) · [CU23 — Administrar denuncias](/casos-de-uso/cu23-administrar-denuncias.md)

| Escritorio | Móvil |
|---|---|
| ![Denuncia de obra en escritorio](img/desktop/moderacion-denuncia-obra.webp ':size=520') | ![Denuncia de obra en móvil](img/mobile/moderacion-denuncia-obra.webp ':size=180') |

---

## Denuncia de comentario

**Ruta:** `/moderation/comment/:id` | **Actor:** Moderador

Igual que la anterior pero para un comentario, con enlace a la obra como contexto. Eliminar la obra se reemplaza por eliminar el comentario.

**Casos de uso:** [CU22 — Denunciar comentario](/casos-de-uso/cu22-denunciar-comentario.md) · [CU23 — Administrar denuncias](/casos-de-uso/cu23-administrar-denuncias.md)

| Escritorio | Móvil |
|---|---|
| ![Denuncia de comentario en escritorio](img/desktop/moderacion-denuncia-comentario.webp ':size=520') | ![Denuncia de comentario en móvil](img/mobile/moderacion-denuncia-comentario.webp ':size=180') |

---

## Denuncia resuelta: reabrir

**Ruta:** `/moderation/artwork/:id` | **Actor:** Admin

Denuncia ya resuelta con una sanción, con su resolución a la vista. Sólo un admin ve la acción de reabrirla, que revierte la sanción.

**Casos de uso:** [CU26 — Reabrir un reporte resuelto](/casos-de-uso/cu26-reabrir-un-reporte-resuelto.md)

| Escritorio | Móvil |
|---|---|
| ![Denuncia resuelta: reabrir en escritorio](img/desktop/moderacion-denuncia-resuelta.webp ':size=520') | ![Denuncia resuelta: reabrir en móvil](img/mobile/moderacion-denuncia-resuelta.webp ':size=180') |

---

## Mis apelaciones

**Ruta:** `/appeals` | **Actor:** Usuario autenticado

Listado de las apelaciones propias con su motivo y estado. Recuerda que el contenido eliminado se apela desde su notificación.

**Casos de uso:** [CU27 — Apelar una sanción](/casos-de-uso/cu27-apelar-una-sancion.md)

| Escritorio | Móvil |
|---|---|
| ![Mis apelaciones en escritorio](img/desktop/mis-apelaciones.webp ':size=520') | ![Mis apelaciones en móvil](img/mobile/mis-apelaciones.webp ':size=180') |

---

## Diálogo: apelar contenido eliminado

**Ruta:** `/notifications` | **Actor:** Usuario autenticado

Se abre con el botón «Apelar» de la notificación de eliminación, que ya identifica la obra o el comentario: sólo pide el motivo y un detalle opcional.

**Casos de uso:** [CU27 — Apelar una sanción](/casos-de-uso/cu27-apelar-una-sancion.md)

| Escritorio | Móvil |
|---|---|
| ![Diálogo: apelar contenido eliminado en escritorio](img/desktop/dialogo-apelar-contenido.webp ':size=520') | ![Diálogo: apelar contenido eliminado en móvil](img/mobile/dialogo-apelar-contenido.webp ':size=180') |

---

## Cuenta suspendida

**Ruta:** `/appeals` | **Actor:** Usuario baneado

Una cuenta baneada puede iniciar sesión y navegar, pero no realizar acciones de escritura. Un aviso permanente en todo el sitio ofrece apelar y consultar sus apelaciones. La captura muestra el caso sin apelaciones previas.

**Casos de uso:** [CU27 — Apelar una sanción](/casos-de-uso/cu27-apelar-una-sancion.md)

| Escritorio | Móvil |
|---|---|
| ![Cuenta suspendida en escritorio](img/desktop/cuenta-suspendida.webp ':size=520') | ![Cuenta suspendida en móvil](img/mobile/cuenta-suspendida.webp ':size=180') |

---

## Diálogo: apelar suspensión

**Ruta:** `/appeals` | **Actor:** Usuario baneado

Formulario de apelación del propio baneo, abierto desde el aviso: motivo y detalle opcional.

**Casos de uso:** [CU27 — Apelar una sanción](/casos-de-uso/cu27-apelar-una-sancion.md)

| Escritorio | Móvil |
|---|---|
| ![Diálogo: apelar suspensión en escritorio](img/desktop/dialogo-apelar-suspension.webp ':size=520') | ![Diálogo: apelar suspensión en móvil](img/mobile/dialogo-apelar-suspension.webp ':size=180') |

---

## Cola de apelaciones

**Ruta:** `/moderation/appeals` | **Actor:** Moderador

Listado de apelaciones para revisar, con filtro por estado.

**Casos de uso:** [CU28 — Revisar una apelación](/casos-de-uso/cu28-revisar-una-apelacion.md)

| Escritorio | Móvil |
|---|---|
| ![Cola de apelaciones en escritorio](img/desktop/moderacion-apelaciones.webp ':size=520') | ![Cola de apelaciones en móvil](img/mobile/moderacion-apelaciones.webp ':size=180') |

---

## Revisar apelación

**Ruta:** `/moderation/appeals/:id` | **Actor:** Moderador

Muestra la sanción original junto al motivo y detalle de la apelación, con las acciones de aprobar (revierte la sanción) o rechazar.

**Casos de uso:** [CU28 — Revisar una apelación](/casos-de-uso/cu28-revisar-una-apelacion.md)

| Escritorio | Móvil |
|---|---|
| ![Revisar apelación en escritorio](img/desktop/moderacion-apelacion-detalle.webp ':size=520') | ![Revisar apelación en móvil](img/mobile/moderacion-apelacion-detalle.webp ':size=180') |
