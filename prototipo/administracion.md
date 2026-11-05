# Administración

Herramientas exclusivas del rol admin.

## Perfil visto por un admin

**Ruta:** `/users/:id` | **Actor:** Admin

La gestión del rol vive en el perfil público de cada usuario: un admin ve allí la acción de otorgar o quitar el rol de moderador.

**Casos de uso:** [CU25 — Gestionar rol de moderador](/casos-de-uso/cu25-gestionar-rol-de-moderador.md)

| Escritorio | Móvil |
|---|---|
| ![Perfil visto por un admin en escritorio](img/desktop/perfil-vista-admin.webp ':size=520') | ![Perfil visto por un admin en móvil](img/mobile/perfil-vista-admin.webp ':size=180') |

---

## Herramientas de administración

**Ruta:** `/admin` | **Actor:** Admin

Carga directa de una carta, con rareza y estadísticas opcionales y sin pasar por la calificación, y disparo manual del ciclo de conversión.

**Casos de uso:** [CU15 — Conversión de obra a carta](/casos-de-uso/cu15-conversion-de-obra-a-carta.md) · [CU29 — Cargar carta directamente (Admin)](/casos-de-uso/cu29-cargar-carta-directamente.md) · [CU30 — Disparar el ciclo de conversión manualmente (Admin)](/casos-de-uso/cu30-disparar-el-ciclo-de-conversion-manualmente.md)

| Escritorio | Móvil |
|---|---|
| ![Herramientas de administración en escritorio](img/desktop/admin-herramientas.webp ':size=520') | ![Herramientas de administración en móvil](img/mobile/admin-herramientas.webp ':size=180') |

---

## Ciclo de conversión: resultado

**Ruta:** `/admin` | **Actor:** Admin

Tras disparar el ciclo manualmente se muestra el resumen de obras procesadas, convertidas, fallidas y omitidas.

**Casos de uso:** [CU15 — Conversión de obra a carta](/casos-de-uso/cu15-conversion-de-obra-a-carta.md) · [CU30 — Disparar el ciclo de conversión manualmente (Admin)](/casos-de-uso/cu30-disparar-el-ciclo-de-conversion-manualmente.md)

| Escritorio | Móvil |
|---|---|
| ![Ciclo de conversión: resultado en escritorio](img/desktop/admin-conversion-resultado.webp ':size=520') | ![Ciclo de conversión: resultado en móvil](img/mobile/admin-conversion-resultado.webp ':size=180') |
