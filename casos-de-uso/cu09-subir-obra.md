# CU09 — Subir obra

**Actor principal:** Usuario autenticado
**Precondición:** Usuario autenticado y autorizado.
**Postcondición:** Obra almacenada y disponible en el ciclo de calificación si corresponde.

## Flujo principal

1. El usuario pulsa "Subir obra".
2. Completa título.
3. Selecciona imagen.
4. Opcionalmente completa Lore/descripción.
5. Opcionalmente agrega tags.
6. Indica si permite conversión a carta.
7. El cliente envía los datos al backend.
8. El backend valida campos y archivo.
9. Se carga la imagen a Cloudinary.
10. Cloudinary genera/permite servir versiones optimizadas.
11. Se registra la obra y la fecha/hora de inicio de calificación.
12. La obra queda en estado de calificación.

## Alternativo

- **A1:** Archivo > 25 MB -> rechazar.
- **A2:** Formato no permitido -> rechazar.
- **A3:** Título vacío o excede longitud -> rechazar.
- **A4:** Fallo de Cloudinary -> no crear una obra incompleta.
- **A5:** Tag inexistente -> crear si se admite administración dinámica o rechazar/normalizar según política definida.
- **A6:** Usuario baneado o sin permisos -> 403.
