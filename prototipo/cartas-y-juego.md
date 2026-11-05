# Cartas y juego

Catálogo de cartas, colección personal, apertura de paquetes y batalla.

## Explorar cartas

**Ruta:** `/cards` | **Actor:** Invitado

Catálogo paginado de cartas con filtro por rareza. Las que no se poseen se muestran en gris y ocultan sus estadísticas.

**Casos de uso:** [CU16 — Explorar colección/cartas](/casos-de-uso/cu16-explorar-coleccion-cartas.md)

| Escritorio | Móvil |
|---|---|
| ![Explorar cartas en escritorio](img/desktop/cartas-explorar.webp ':size=520') | ![Explorar cartas en móvil](img/mobile/cartas-explorar.webp ':size=180') |

---

## Detalle de carta: poseída

**Ruta:** `/cards/:id` | **Actor:** Usuario autenticado

Muestra rareza, estadísticas completas, copias obtenidas y un enlace a la obra original. Se puede destacar.

**Casos de uso:** [CU16 — Explorar colección/cartas](/casos-de-uso/cu16-explorar-coleccion-cartas.md) · [CU20 — Destacar obra o carta](/casos-de-uso/cu20-destacar-obra-o-carta.md)

| Escritorio | Móvil |
|---|---|
| ![Detalle de carta: poseída en escritorio](img/desktop/carta-detalle-poseida.webp ':size=520') | ![Detalle de carta: poseída en móvil](img/mobile/carta-detalle-poseida.webp ':size=180') |

---

## Detalle de carta: no poseída

**Ruta:** `/cards/:id` | **Actor:** Usuario autenticado

Regla del álbum: la carta se ve en gris y sus estadísticas quedan ocultas hasta conseguirla en un paquete.

**Casos de uso:** [CU16 — Explorar colección/cartas](/casos-de-uso/cu16-explorar-coleccion-cartas.md)

| Escritorio | Móvil |
|---|---|
| ![Detalle de carta: no poseída en escritorio](img/desktop/carta-detalle-no-poseida.webp ':size=520') | ![Detalle de carta: no poseída en móvil](img/mobile/carta-detalle-no-poseida.webp ':size=180') |

---

## Colección

**Ruta:** `/collection` | **Actor:** Usuario autenticado

Álbum personal: cartas obtenidas a color, con copias y acción de destacar, junto a las que faltan. El panel de paquetes muestra el inventario, el tiempo hasta el próximo y la acción de abrir.

**Casos de uso:** [CU16 — Explorar colección/cartas](/casos-de-uso/cu16-explorar-coleccion-cartas.md) · [CU17 — Abrir paquete](/casos-de-uso/cu17-abrir-paquete.md) · [CU20 — Destacar obra o carta](/casos-de-uso/cu20-destacar-obra-o-carta.md)

| Escritorio | Móvil |
|---|---|
| ![Colección en escritorio](img/desktop/coleccion.webp ':size=520') | ![Colección en móvil](img/mobile/coleccion.webp ':size=180') |

---

## Abrir paquete: revelado

**Ruta:** `/collection` | **Actor:** Usuario autenticado

Pantalla completa que revela las cartas una a una al tocarlas, con un indicador de las rarezas obtenidas. Permite saltar directo al resumen.

**Casos de uso:** [CU17 — Abrir paquete](/casos-de-uso/cu17-abrir-paquete.md)

| Escritorio | Móvil |
|---|---|
| ![Abrir paquete: revelado en escritorio](img/desktop/paquete-revelar.webp ':size=520') | ![Abrir paquete: revelado en móvil](img/mobile/paquete-revelar.webp ':size=180') |

---

## Abrir paquete: resumen

**Ruta:** `/collection` | **Actor:** Usuario autenticado

Cierre de la apertura con todas las cartas obtenidas y sus rarezas.

**Casos de uso:** [CU17 — Abrir paquete](/casos-de-uso/cu17-abrir-paquete.md)

| Escritorio | Móvil |
|---|---|
| ![Abrir paquete: resumen en escritorio](img/desktop/paquete-resumen.webp ':size=520') | ![Abrir paquete: resumen en móvil](img/mobile/paquete-resumen.webp ':size=180') |

---

## Batalla: invitado

**Ruta:** `/battle` | **Actor:** Invitado

Explica cómo funciona la batalla e invita a iniciar sesión o registrarse, ya que jugar requiere cartas propias.

**Casos de uso:** [CU18 — Jugar autobattler](/casos-de-uso/cu18-jugar-autobattler.md)

| Escritorio | Móvil |
|---|---|
| ![Batalla: invitado en escritorio](img/desktop/batalla-invitado.webp ':size=520') | ![Batalla: invitado en móvil](img/mobile/batalla-invitado.webp ':size=180') |

---

## Batalla: elegir carta

**Ruta:** `/battle` | **Actor:** Usuario autenticado

Selección de la carta propia con la que pelear, entre las de la colección. Incluye el panel de paquetes.

**Casos de uso:** [CU17 — Abrir paquete](/casos-de-uso/cu17-abrir-paquete.md) · [CU18 — Jugar autobattler](/casos-de-uso/cu18-jugar-autobattler.md)

| Escritorio | Móvil |
|---|---|
| ![Batalla: elegir carta en escritorio](img/desktop/batalla-elegir-carta.webp ':size=520') | ![Batalla: elegir carta en móvil](img/mobile/batalla-elegir-carta.webp ':size=180') |

---

## Batalla: resultado

**Ruta:** `/battle` | **Actor:** Usuario autenticado

Simulación automática contra un rival aleatorio: barras de vida, registro de turnos con daño, críticos y esquives, resultado final y opción de volver a batallar. El resultado no se guarda.

**Casos de uso:** [CU18 — Jugar autobattler](/casos-de-uso/cu18-jugar-autobattler.md)

| Escritorio | Móvil |
|---|---|
| ![Batalla: resultado en escritorio](img/desktop/batalla-resultado.webp ':size=520') | ![Batalla: resultado en móvil](img/mobile/batalla-resultado.webp ':size=180') |
