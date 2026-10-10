# CP04 — Explorar obras (CU12)

**Caso de uso:** [CU12 — Explorar obras](/casos-de-uso/cu12-explorar-obras.md)

- **Precondición:** ninguna (accesible sin sesión).
- **Pasos:** abrir el explorador → aplicar filtros (búsqueda, autor, fecha, tag, rareza) → paginar → abrir el detalle de una obra.
- **Resultado esperado:** listado paginado de obras no eliminadas; detalle completo al seleccionar una.
- **Casos negativos:** sin resultados → estado vacío, no error (A1) · filtro inválido → 400 (A2) · obra eliminada entre búsqueda y detalle → recurso no disponible (A3).
