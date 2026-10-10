# CU29 — Cargar carta directamente (Admin)

**Actor principal:** Admin
**Precondición:** El actor posee el rol `admin`.
**Postcondición:** Existe una nueva obra con `conversion_status = converted` y su carta asociada, sin haber pasado por el período de calificación.

## Flujo principal

1. El admin abre la herramienta de carga directa.
2. Completa título y, opcionalmente, descripción/tags, igual que al subir una obra normal.
3. Selecciona la imagen.
4. Indica la rareza de la carta resultante.
5. Opcionalmente indica estadísticas explícitas (ataque, defensa, vida, velocidad).
6. El backend sube la imagen, crea la obra ya `converted` y crea la carta en el mismo paso.
7. Las estadísticas no indicadas se completan según la rareza, con la misma fórmula que usa el sembrado del catálogo mínimo ([requerimientos](/requerimientos.md) §5.5, regla 7).
8. Se devuelve la carta creada.

## Alternativo

- **A1:** Archivo inválido o ausente -> mismo tratamiento que [CU09](cu09-subir-obra.md) (A1/A2).
- **A2:** Título vacío o excede longitud -> rechazar.
- **A3:** Rareza inválida -> 400.
- **A4:** Estadística negativa o no entera -> 400.
- **A5:** Actor sin rol `admin` -> 403.

## Observaciones

No genera notificación a seguidores del admin: no es un upload orgánico. El admin no posee automáticamente la carta creada.
