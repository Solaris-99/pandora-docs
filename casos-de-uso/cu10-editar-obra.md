# CU10 — Editar obra

**Actor principal:** Usuario autenticado
**Precondición:** El usuario es autor de la obra; la obra está dentro de su período de calificación (todavía no fue convertida en carta).
**Postcondición:** La obra queda actualizada con los nuevos datos.

## Flujo principal

1. El usuario entra a una obra propia.
2. Pulsa "Editar".
3. Se muestra el formulario de edición, precargado con los datos actuales.
4. Completa los campos (los mismos que al subir obra: título, imagen, lore/descripción, tags, permiso de conversión).
5. Confirma los cambios.
6. El backend valida campos y archivo, igual que en [CU09](cu09-subir-obra.md).
7. Se actualiza la obra.

## Alternativo

- **A1:** La obra ya fue convertida (su período de calificación terminó) -> rechazar, no se puede editar una obra que ya es carta.
- **A2:** El usuario no es el autor -> 403.
- **A3:** Archivo o campos inválidos -> mismo tratamiento que [CU09](cu09-subir-obra.md) (A1/A2/A3).
- **A4:** Obra inexistente o eliminada -> 404.

## Observaciones

Comparte las reglas de validación de campos y archivo con [CU09](cu09-subir-obra.md).
