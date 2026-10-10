# Pandora — Diseño de Componentes

Este documento cubre los componentes de la capa API: endpoints REST propuestos, guards de autorización, validaciones, convenciones de respuesta y el contrato de estado de una carta. El contrato HTTP exacto (payloads, respuestas de ejemplo, códigos de error) vive en [convenciones de API](/api-conventions.md); este documento es su resumen a nivel de diseño.

---

## Índice

- [Catálogo de endpoints](#catálogo-de-endpoints) — cada fila enlaza al grupo
- [Guards y autorización](guards.md)
  - [`JwtAuthGuard`](guards.md#jwtauthguard)
  - [`RolesGuard`](guards.md#rolesguard)
  - [`OwnershipGuard` (verificación en servicio)](guards.md#ownershipguard-verificación-en-servicio)
  - [`VerifiedEmailGuard`](guards.md#verifiedemailguard)
  - [Rate limiting](guards.md#rate-limiting)
  - [`BannedUserGuard`](guards.md#banneduserguard)
- [Validaciones](validaciones.md)
  - [Validación general](validaciones.md#validación-general)
  - [Usuario](validaciones.md#usuario)
  - [Obras](validaciones.md#obras)
  - [Comentarios](validaciones.md#comentarios)
  - [Ratings](validaciones.md#ratings)
  - [Reportes](validaciones.md#reportes)
  - [DTOs](validaciones.md#dtos)
- [Convenciones de API](convenciones-api.md)
  - [Respuestas de éxito](convenciones-api.md#respuestas-de-éxito)
  - [Errores](convenciones-api.md#errores)
  - [Paginación](convenciones-api.md#paginación)
- [Estado de una carta en el cliente](estado-carta.md)
- [Documento completo](completo.md) — todo el documento en una sola página

---

## Catálogo de endpoints

Los endpoints siguientes son una propuesta derivada de los requerimientos y casos de uso (ver [requerimientos](/requerimientos/README.md), [casos de uso](/casos-de-uso/README.md)). Los nombres pueden adaptarse, pero se recomienda mantener una convención uniforme.

| Grupo | Prefijos | Endpoints |
|---|---|---|
| [Auth](endpoints-auth.md) | `/auth` | 11 |
| [Users / Profiles](endpoints-users.md) | `/users` | 6 |
| [Artworks](endpoints-artworks.md) | `/artworks`, `/users` | 8 |
| [Ratings](endpoints-ratings.md) | `/artworks` | 3 |
| [Cards](endpoints-cards.md) | `/cards`, `/users` | 6 |
| [Packages](endpoints-packages.md) | `/users` | 2 |
| [Battle](endpoints-battle.md) | `/battle` | 1 |
| [Favorites](endpoints-favorites.md) | `/artworks`, `/users` | 4 |
| [Reports](endpoints-reports.md) | `/artworks`, `/comments`, `/moderation` | 5 |
| [Notifications](endpoints-notifications.md) | `/notifications` | 4 |
| [Tags](endpoints-tags.md) | `/tags` | 1 |
| [Administración y Apelaciones](endpoints-administracion.md) | `/roles`, `/moderation`, `/artworks`, `/comments`, `/users`, `/conversion` | 12 |
