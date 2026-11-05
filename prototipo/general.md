# Pantallas generales

Puntos de entrada al sitio y páginas informativas.

## Inicio

**Ruta:** `/` | **Actor:** Invitado

Portada del sitio: destaca la última obra subida con sus primeros comentarios, y lista las obras en calificación y las cartas más recientes. Cierra con una explicación de qué es Pandora. Como el invitado no posee cartas, se ven en gris y sin estadísticas.

**Casos de uso:** [CU12 — Explorar obras](/casos-de-uso/cu12-explorar-obras.md) · [CU16 — Explorar colección/cartas](/casos-de-uso/cu16-explorar-coleccion-cartas.md)

| Escritorio | Móvil |
|---|---|
| ![Inicio en escritorio](img/desktop/inicio.webp ':size=520') | ![Inicio en móvil](img/mobile/inicio.webp ':size=180') |

---

## Inicio con sesión iniciada

**Ruta:** `/` | **Actor:** Usuario autenticado

La misma portada con la barra de navegación completa (mis obras, colección, subir obra, notificaciones y menú de usuario). Las cartas que el usuario posee aparecen a color y con sus estadísticas, y las invitaciones a registrarse se reemplazan por el acceso a subir obra.

**Casos de uso:** [CU09 — Subir obra](/casos-de-uso/cu09-subir-obra.md) · [CU12 — Explorar obras](/casos-de-uso/cu12-explorar-obras.md) · [CU16 — Explorar colección/cartas](/casos-de-uso/cu16-explorar-coleccion-cartas.md) · [CU24 — Notificaciones](/casos-de-uso/cu24-notificaciones.md)

| Escritorio | Móvil |
|---|---|
| ![Inicio con sesión iniciada en escritorio](img/desktop/inicio-usuario.webp ':size=520') | ![Inicio con sesión iniciada en móvil](img/mobile/inicio-usuario.webp ':size=180') |

---

## Acerca de

**Ruta:** `/about` | **Actor:** Invitado

Página informativa que resume cada funcionalidad de la plataforma en una tarjeta. No ejecuta ningún caso de uso.

**Casos de uso:** ninguno.

| Escritorio | Móvil |
|---|---|
| ![Acerca de en escritorio](img/desktop/acerca-de.webp ':size=520') | ![Acerca de en móvil](img/mobile/acerca-de.webp ':size=180') |

---

## Página no encontrada

**Ruta:** `/ruta-inexistente` | **Actor:** Invitado

Error 404 para cualquier ruta inexistente, con un acceso para volver al inicio. No ejecuta ningún caso de uso.

**Casos de uso:** ninguno.

| Escritorio | Móvil |
|---|---|
| ![Página no encontrada en escritorio](img/desktop/no-encontrada.webp ':size=520') | ![Página no encontrada en móvil](img/mobile/no-encontrada.webp ':size=180') |
