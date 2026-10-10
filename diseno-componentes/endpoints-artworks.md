# Endpoints — Artworks

| Método | Endpoint | Uso |
|---|---|---|
| POST | `/artworks` | Crear obra |
| GET | `/artworks` | Explorar y filtrar |
| GET | `/artworks/:id` | Detalle |
| PATCH | `/artworks/:id` | Editar obra propia |
| DELETE | `/artworks/:id` | Eliminación lógica |
| GET | `/users/me/artworks` | Obras propias |
| GET | `/artworks/:id/comments` | Comentarios |
| POST | `/artworks/:id/comments` | Crear comentario |

## Query params sugeridos para `GET /artworks`

```text
page / cursor
limit
search
from
until
authorId
tag
sort
qualificationOnly
converted
rarity
```

Para búsquedas que crezcan en complejidad, conviene encapsular parámetros en un DTO específico de filtros en lugar de construir SQL directamente desde query strings.
