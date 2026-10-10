# Organización del backend por módulos

Estructura de `src/` en NestJS, un módulo por dominio funcional:

```text
src/
├── auth/
├── users/
├── roles/
├── artworks/
├── ratings/
├── comments/
├── cards/
├── collections/
├── packages/
├── battle/
├── follows/
├── favorites/
├── notifications/
├── reports/
├── moderation/
├── tags/
├── storage/
├── conversion/
├── scheduler/
├── common/
└── database/
```

## Capas recomendadas

```text
Controller
    ↓
Application / Service
    ↓
Domain logic
    ↓
Repository / ORM
    ↓
PostgreSQL
```

El detalle de qué controladores, servicios y entidades componen cada módulo vive en [clases de diseño](/clases-diseno/README.md) y [diseño de componentes](/diseno-componentes/README.md) (endpoints y guards).
