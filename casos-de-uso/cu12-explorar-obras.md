# CU12 — Explorar obras

**Actor principal:** Invitado o usuario
**Precondición:** Ninguna.

## Flujo principal

1. El usuario abre el explorador.
2. El sistema devuelve obras visibles y no eliminadas.
3. El usuario aplica filtros.
4. El cliente solicita la página correspondiente.
5. El usuario selecciona una obra.
6. El sistema muestra detalle.

## Alternativo

- **A1:** No existen resultados -> mostrar estado vacío.
- **A2:** Filtro inválido -> devolver error de validación.
- **A3:** Obra fue eliminada entre la búsqueda y el detalle -> devolver recurso no disponible.
