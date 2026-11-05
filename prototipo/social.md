# Social y notificaciones

Perfiles, seguimiento, destacados y notificaciones.

## Perfil de otro usuario

**Ruta:** `/users/:id` | **Actor:** Usuario autenticado

Perfil público con avatar, bio, seguidores y seguidos, y la acción de seguir o dejar de seguir. Sin sesión se ve igual, pero sin esa acción.

**Casos de uso:** [CU06 — Ver/editar perfil](/casos-de-uso/cu06-ver-editar-perfil.md) · [CU19 — Seguir usuario](/casos-de-uso/cu19-seguir-usuario.md)

| Escritorio | Móvil |
|---|---|
| ![Perfil de otro usuario en escritorio](img/desktop/perfil-otro-usuario.webp ':size=520') | ![Perfil de otro usuario en móvil](img/mobile/perfil-otro-usuario.webp ':size=180') |

---

## Perfil propio

**Ruta:** `/users/:id` | **Actor:** Usuario autenticado

Reemplaza seguir por el acceso a editar el perfil y agrega una vista previa de las obras y cartas destacadas.

**Casos de uso:** [CU06 — Ver/editar perfil](/casos-de-uso/cu06-ver-editar-perfil.md) · [CU20 — Destacar obra o carta](/casos-de-uso/cu20-destacar-obra-o-carta.md)

| Escritorio | Móvil |
|---|---|
| ![Perfil propio en escritorio](img/desktop/perfil-propio.webp ':size=520') | ![Perfil propio en móvil](img/mobile/perfil-propio.webp ':size=180') |

---

## Seguidores y seguidos

**Ruta:** `/users/:id/connections` | **Actor:** Usuario autenticado

Listas de seguidores y seguidos de un usuario, en dos pestañas.

**Casos de uso:** [CU19 — Seguir usuario](/casos-de-uso/cu19-seguir-usuario.md)

| Escritorio | Móvil |
|---|---|
| ![Seguidores y seguidos en escritorio](img/desktop/conexiones.webp ':size=520') | ![Seguidores y seguidos en móvil](img/mobile/conexiones.webp ':size=180') |

---

## Destacados

**Ruta:** `/favorites` | **Actor:** Usuario autenticado

Listado completo de las obras y cartas destacadas, en dos pestañas.

**Casos de uso:** [CU20 — Destacar obra o carta](/casos-de-uso/cu20-destacar-obra-o-carta.md)

| Escritorio | Móvil |
|---|---|
| ![Destacados en escritorio](img/desktop/destacados.webp ':size=520') | ![Destacados en móvil](img/mobile/destacados.webp ':size=180') |

---

## Notificaciones

**Ruta:** `/notifications` | **Actor:** Usuario autenticado

Panel paginado de notificaciones. Las no leídas se resaltan y se pueden marcar como leídas una a una o todas juntas. Es también donde el autor se entera de la conversión de sus obras y donde puede apelar la eliminación de una obra o comentario.

**Casos de uso:** [CU15 — Conversión de obra a carta](/casos-de-uso/cu15-conversion-de-obra-a-carta.md) · [CU24 — Notificaciones](/casos-de-uso/cu24-notificaciones.md) · [CU27 — Apelar una sanción](/casos-de-uso/cu27-apelar-una-sancion.md)

| Escritorio | Móvil |
|---|---|
| ![Notificaciones en escritorio](img/desktop/notificaciones.webp ':size=520') | ![Notificaciones en móvil](img/mobile/notificaciones.webp ':size=180') |
