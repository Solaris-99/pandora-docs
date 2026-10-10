# Endpoints — Ratings

| Método | Endpoint | Uso |
|---|---|---|
| PUT | `/artworks/:id/rating` | Crear o actualizar rating del usuario |
| DELETE | `/artworks/:id/rating` | Retirar rating, si la regla lo permite |
| GET | `/artworks/:id/ratings` | Resumen o datos de calificación |

No se debe exponer información que permita manipular el proceso de generación de estadísticas si ésta no corresponde a la interfaz pública.
