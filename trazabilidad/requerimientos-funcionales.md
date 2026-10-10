# Trazabilidad de requerimientos funcionales

| RF | Descripción corta | Caso(s) de uso | Endpoint(s) / componente principal |
|---|---|---|---|
| RF01 | Registro | [CU01](/casos-de-uso/cu01-registrarse.md) | `POST /auth/register` |
| RF02 | Verificación de email | [CU02](/casos-de-uso/cu02-verificar-email.md) | `POST /auth/verify-email`, `POST /auth/resend-verification` |
| RF03 | Login | [CU03](/casos-de-uso/cu03-login.md) | `POST /auth/login`, `GET /auth/google`, `POST /auth/google` |
| RF03b | Recuperación de contraseña | [CU04](/casos-de-uso/cu04-recuperar-contrasena-olvidada.md) | `POST /auth/forgot-password`, `POST /auth/reset-password` |
| RF04 | Perfil | [CU06](/casos-de-uso/cu06-ver-editar-perfil.md) | `GET /users/:id`, `PATCH /users/me` |
| RF05 | Publicación de obra | [CU09](/casos-de-uso/cu09-subir-obra.md) | `POST /artworks` |
| RF06 | Gestión de obras | [CU09](/casos-de-uso/cu09-subir-obra.md), [CU10](/casos-de-uso/cu10-editar-obra.md), [CU11](/casos-de-uso/cu11-eliminar-obra.md) | `GET /users/me/artworks`, `PATCH /artworks/:id`, `DELETE /artworks/:id` |
| RF07 | Explorador de obras | [CU12](/casos-de-uso/cu12-explorar-obras.md) | `GET /artworks` |
| RF08 | Detalle de obra | [CU12](/casos-de-uso/cu12-explorar-obras.md) | `GET /artworks/:id` |
| RF09 | Comentarios | [CU13](/casos-de-uso/cu13-comentar-obra.md) | `GET/POST /artworks/:id/comments` |
| RF10 | Q2Q | [CU14](/casos-de-uso/cu14-calificar-obra-mediante-q2q.md) | `PUT /artworks/:id/rating`, `GET /artworks/:id/ratings` |
| RF11 | Conversión a carta | [CU15](/casos-de-uso/cu15-conversion-de-obra-a-carta.md), [CU30](/casos-de-uso/cu30-disparar-el-ciclo-de-conversion-manualmente.md) | `ConversionSchedulerService.runConversionCycle`, `POST /conversion/run` |
| RF12 | Estadísticas de carta | [CU15](/casos-de-uso/cu15-conversion-de-obra-a-carta.md) | entidad `Card` |
| RF13 | Colección | [CU16](/casos-de-uso/cu16-explorar-coleccion-cartas.md) | `GET /cards`, `GET /users/me/cards` |
| RF14 | Detalle de carta | [CU16](/casos-de-uso/cu16-explorar-coleccion-cartas.md) | `GET /cards/:id` |
| RF15 | Paquetes | [CU17](/casos-de-uso/cu17-abrir-paquete.md) | `POST /users/me/packages/open` |
| RF16 | Regeneración de paquetes | [CU17](/casos-de-uso/cu17-abrir-paquete.md) | `GET /users/me/packages` |
| RF17 | Autobattler | [CU18](/casos-de-uso/cu18-jugar-autobattler.md) | `GET /battle/opponent` |
| RF18 | Seguimiento | [CU19](/casos-de-uso/cu19-seguir-usuario.md) | `POST/DELETE /users/:id/follow` |
| RF19 | Favoritos | [CU20](/casos-de-uso/cu20-destacar-obra-o-carta.md) | `POST/DELETE /artworks/:id/favorite`, `POST/DELETE /cards/:id/favorite` |
| RF20 | Notificaciones | [CU24](/casos-de-uso/cu24-notificaciones.md) | `GET /notifications`, `PATCH /notifications/:id/read` |
| RF21 | Reportes de obras | [CU21](/casos-de-uso/cu21-denunciar-obra.md) | `POST /artworks/:id/reports` |
| RF22 | Reportes de comentarios | [CU22](/casos-de-uso/cu22-denunciar-comentario.md) | `POST /comments/:id/reports` |
| RF23 | Moderación | [CU23](/casos-de-uso/cu23-administrar-denuncias.md) | `GET /moderation/reports`, `POST /moderation/reports/:id/resolve` |
| RF24 | Estados de contenido | *(transversal — aplica a todas las entidades con `deleted`/`status`)* | soft delete + enums de estado en cada módulo |
| RF25 | Gestión de moderadores | [CU25](/casos-de-uso/cu25-gestionar-rol-de-moderador.md) | `POST/DELETE /roles/moderators/:userId` |
| RF26 | Reapertura de reportes | [CU26](/casos-de-uso/cu26-reabrir-un-reporte-resuelto.md) | `POST /moderation/reports/:id/reopen` |
| RF27 | Apelaciones | [CU27](/casos-de-uso/cu27-apelar-una-sancion.md), [CU28](/casos-de-uso/cu28-revisar-una-apelacion.md) | `POST /artworks/:id/appeals`, `POST /comments/:id/appeals`, `POST /users/me/ban-appeal`, `GET/POST /moderation/appeals*` |
| RF28 | Carga directa de carta (Admin) | [CU29](/casos-de-uso/cu29-cargar-carta-directamente.md) | `POST /artworks/admin-upload` |
| RF29 | Disparo manual de conversión (Admin) | [CU30](/casos-de-uso/cu30-disparar-el-ciclo-de-conversion-manualmente.md) | `POST /conversion/run` |
