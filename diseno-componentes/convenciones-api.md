# Convenciones de API

## Respuestas de éxito

Cada endpoint devuelve directamente el recurso (o una página de recursos) serializado según su DTO de salida — ver el contrato completo en [convenciones de API](/api-conventions.md).

## Errores

```json
{
  "statusCode": 400,
  "code": "VALIDATION_ERROR",
  "message": "La obra no cumple los requisitos de validación.",
  "errors": [{ "field": "title", "message": "..." }],
  "timestamp": "2026-...",
  "path": "/api/v1/artworks"
}
```

No devolver stack traces ni detalles internos al cliente. Normalizado centralmente por `AllExceptionsFilter` (ver [clases de diseño, módulo `common`](/clases-diseno/common.md)).

## Paginación

`page`/`limit` para listados de la API (galería, comentarios, notificaciones, reportes — RNF15), con respuesta `{items, total, page, limit, totalPages}`.
