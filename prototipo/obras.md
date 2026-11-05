# Obras

Exploración, publicación y participación sobre las obras.

## Nuevas obras

**Ruta:** `/explore` | **Actor:** Invitado

Explorador de las obras todavía en período de calificación, con búsqueda, filtro por tag y orden. Cada tarjeta indica cuánto falta para que termine la calificación.

**Casos de uso:** [CU12 — Explorar obras](/casos-de-uso/cu12-explorar-obras.md) · [CU14 — Calificar obra mediante Q2Q](/casos-de-uso/cu14-calificar-obra-mediante-q2q.md)

| Escritorio | Móvil |
|---|---|
| ![Nuevas obras en escritorio](img/desktop/explorar-nuevas.webp ':size=520') | ![Nuevas obras en móvil](img/mobile/explorar-nuevas.webp ':size=180') |

---

## Todas las obras

**Ruta:** `/explore/all` | **Actor:** Invitado

Explorador general: suma a las obras en calificación las que ya terminaron, e indica cuáles tienen carta disponible.

**Casos de uso:** [CU12 — Explorar obras](/casos-de-uso/cu12-explorar-obras.md) · [CU15 — Conversión de obra a carta](/casos-de-uso/cu15-conversion-de-obra-a-carta.md)

| Escritorio | Móvil |
|---|---|
| ![Todas las obras en escritorio](img/desktop/explorar-todas.webp ':size=520') | ![Todas las obras en móvil](img/mobile/explorar-todas.webp ':size=180') |

---

## Detalle de obra: invitado

**Ruta:** `/artworks/:id` | **Actor:** Invitado

Muestra la imagen, el autor, los tags, la descripción, el resumen de votos y los comentarios. Sin sesión, se invita a iniciarla para calificar y comentar.

**Casos de uso:** [CU12 — Explorar obras](/casos-de-uso/cu12-explorar-obras.md)

| Escritorio | Móvil |
|---|---|
| ![Detalle de obra: invitado en escritorio](img/desktop/obra-detalle-invitado.webp ':size=520') | ![Detalle de obra: invitado en móvil](img/mobile/obra-detalle-invitado.webp ':size=180') |

---

## Detalle de obra: usuario

**Ruta:** `/artworks/:id` | **Actor:** Usuario autenticado

Durante el período de calificación permite votar de 1 a 5 estrellas y elegir una emoción (la captura muestra un voto a medio completar). Suma el formulario de comentarios y la denuncia de la obra y de los comentarios.

**Casos de uso:** [CU12 — Explorar obras](/casos-de-uso/cu12-explorar-obras.md) · [CU13 — Comentar obra](/casos-de-uso/cu13-comentar-obra.md) · [CU14 — Calificar obra mediante Q2Q](/casos-de-uso/cu14-calificar-obra-mediante-q2q.md) · [CU21 — Denunciar obra](/casos-de-uso/cu21-denunciar-obra.md) · [CU22 — Denunciar comentario](/casos-de-uso/cu22-denunciar-comentario.md)

| Escritorio | Móvil |
|---|---|
| ![Detalle de obra: usuario en escritorio](img/desktop/obra-detalle-usuario.webp ':size=520') | ![Detalle de obra: usuario en móvil](img/mobile/obra-detalle-usuario.webp ':size=180') |

---

## Detalle de obra: autor

**Ruta:** `/artworks/:id` | **Actor:** Usuario autenticado (autor)

En una obra propia, en lugar de denunciar aparecen las acciones de destacar, editar y eliminar.

**Casos de uso:** [CU10 — Editar obra](/casos-de-uso/cu10-editar-obra.md) · [CU11 — Eliminar obra](/casos-de-uso/cu11-eliminar-obra.md) · [CU20 — Destacar obra o carta](/casos-de-uso/cu20-destacar-obra-o-carta.md)

| Escritorio | Móvil |
|---|---|
| ![Detalle de obra: autor en escritorio](img/desktop/obra-detalle-propia.webp ':size=520') | ![Detalle de obra: autor en móvil](img/mobile/obra-detalle-propia.webp ':size=180') |

---

## Detalle de obra: ya convertida

**Ruta:** `/artworks/:id` | **Actor:** Usuario autenticado (autor)

Resultado visible de la conversión: la obra ya es una carta, termina la calificación y el autor pierde las acciones de editar y eliminar, como explica un aviso. Conserva la de destacar.

**Casos de uso:** [CU10 — Editar obra](/casos-de-uso/cu10-editar-obra.md) · [CU11 — Eliminar obra](/casos-de-uso/cu11-eliminar-obra.md) · [CU15 — Conversión de obra a carta](/casos-de-uso/cu15-conversion-de-obra-a-carta.md) · [CU20 — Destacar obra o carta](/casos-de-uso/cu20-destacar-obra-o-carta.md)

| Escritorio | Móvil |
|---|---|
| ![Detalle de obra: ya convertida en escritorio](img/desktop/obra-detalle-convertida.webp ':size=520') | ![Detalle de obra: ya convertida en móvil](img/mobile/obra-detalle-convertida.webp ':size=180') |

---

## Subir obra

**Ruta:** `/artworks/new` | **Actor:** Usuario con email verificado

Formulario de publicación: título, lore o descripción, imagen, tags y permiso de conversión a carta.

**Casos de uso:** [CU09 — Subir obra](/casos-de-uso/cu09-subir-obra.md)

| Escritorio | Móvil |
|---|---|
| ![Subir obra en escritorio](img/desktop/obra-subir.webp ':size=520') | ![Subir obra en móvil](img/mobile/obra-subir.webp ':size=180') |

---

## Editar obra

**Ruta:** `/artworks/:id/edit` | **Actor:** Usuario autenticado (autor)

El mismo formulario precargado con los datos actuales. La imagen sólo se reemplaza si se elige una nueva.

**Casos de uso:** [CU10 — Editar obra](/casos-de-uso/cu10-editar-obra.md)

| Escritorio | Móvil |
|---|---|
| ![Editar obra en escritorio](img/desktop/obra-editar.webp ':size=520') | ![Editar obra en móvil](img/mobile/obra-editar.webp ':size=180') |

---

## Mis obras

**Ruta:** `/artworks/mine` | **Actor:** Usuario autenticado

Listado de las obras propias con su estado, un acceso para subir una nueva y la acción de destacar en cada tarjeta.

**Casos de uso:** [CU09 — Subir obra](/casos-de-uso/cu09-subir-obra.md) · [CU10 — Editar obra](/casos-de-uso/cu10-editar-obra.md) · [CU11 — Eliminar obra](/casos-de-uso/cu11-eliminar-obra.md) · [CU20 — Destacar obra o carta](/casos-de-uso/cu20-destacar-obra-o-carta.md)

| Escritorio | Móvil |
|---|---|
| ![Mis obras en escritorio](img/desktop/mis-obras.webp ':size=520') | ![Mis obras en móvil](img/mobile/mis-obras.webp ':size=180') |

---

## Diálogo: eliminar obra

**Ruta:** `/artworks/:id` | **Actor:** Usuario autenticado (autor)

Confirmación previa a eliminar una obra propia.

**Casos de uso:** [CU11 — Eliminar obra](/casos-de-uso/cu11-eliminar-obra.md)

| Escritorio | Móvil |
|---|---|
| ![Diálogo: eliminar obra en escritorio](img/desktop/dialogo-eliminar-obra.webp ':size=520') | ![Diálogo: eliminar obra en móvil](img/mobile/dialogo-eliminar-obra.webp ':size=180') |

---

## Diálogo: denunciar obra

**Ruta:** `/artworks/:id` | **Actor:** Usuario autenticado

Formulario con un motivo predefinido y un comentario opcional.

**Casos de uso:** [CU21 — Denunciar obra](/casos-de-uso/cu21-denunciar-obra.md)

| Escritorio | Móvil |
|---|---|
| ![Diálogo: denunciar obra en escritorio](img/desktop/dialogo-denunciar-obra.webp ':size=520') | ![Diálogo: denunciar obra en móvil](img/mobile/dialogo-denunciar-obra.webp ':size=180') |

---

## Diálogo: denunciar comentario

**Ruta:** `/artworks/:id` | **Actor:** Usuario autenticado

El mismo formulario de denuncia, abierto desde la bandera de un comentario ajeno.

**Casos de uso:** [CU22 — Denunciar comentario](/casos-de-uso/cu22-denunciar-comentario.md)

| Escritorio | Móvil |
|---|---|
| ![Diálogo: denunciar comentario en escritorio](img/desktop/dialogo-denunciar-comentario.webp ':size=520') | ![Diálogo: denunciar comentario en móvil](img/mobile/dialogo-denunciar-comentario.webp ':size=180') |
