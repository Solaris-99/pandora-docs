# Módulo `tags`

## Entidad: `Tag`

`id`, `name: string` (único, normalizado a minúsculas).

## Servicio: `TagsService`

`findAllView(query?)` (filtro `search` server-side vía `ILIKE`), `findOrCreateByNames(names)` (normaliza, deduplica y crea dinámicamente los tags nuevos).
