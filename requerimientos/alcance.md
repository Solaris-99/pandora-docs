# 2. Alcance

## 2.1 Incluido

### Portal web

- Registro.
- Login.
- OAuth con Google.
- Perfil de usuario/artista.
- Publicación de obras y Lore.
- Tags.
- Explorador de obras.
- Búsqueda y filtros.
- Visualización de obras y cartas.
- Comentarios.
- Sistema Q2Q de calificación.
- Favoritos.
- Seguimiento de usuarios.
- Notificaciones.
- Colección de cartas.
- Apertura de paquetes.
- Autobattler local.
- Denuncias.
- Panel de moderación para usuarios autorizados.
- Apelaciones de sanciones (eliminación de obra/comentario, baneo).
- Panel de administración (gestión de moderadores, reapertura de reportes resueltos).

### Aplicación Android

Debe cubrir las funcionalidades de usuario previstas para móvil, compartiendo el mismo backend y reglas de negocio.

Quedan **fuera** de la app Android el panel de moderación, la gestión de apelaciones por parte del personal y las herramientas de administración ([CU23](../casos-de-uso/cu23-administrar-denuncias.md), [CU25](../casos-de-uso/cu25-gestionar-rol-de-moderador.md), [CU26](../casos-de-uso/cu26-reabrir-un-reporte-resuelto.md), [CU28](../casos-de-uso/cu28-revisar-una-apelacion.md)–[CU30](../casos-de-uso/cu30-disparar-el-ciclo-de-conversion-manualmente.md)); esas funciones se ofrecen únicamente en el portal web. La app sí incluye reportar contenido ([CU21](../casos-de-uso/cu21-denunciar-obra.md)/[CU22](../casos-de-uso/cu22-denunciar-comentario.md)) y apelar sanciones propias ([CU27](../casos-de-uso/cu27-apelar-una-sancion.md)).

### Backend

- API REST.
- Autenticación.
- Autorización.
- Usuarios.
- Obras.
- Comentarios.
- Calificaciones.
- Cartas.
- Colecciones.
- Paquetes.
- Seguimientos.
- Favoritos.
- Reportes.
- Apelaciones.
- Administración (gestión de moderadores, reapertura de reportes).
- Notificaciones.
- Conversión periódica mediante CRON.

## 2.2 Fuera del alcance actual

- Aplicación desktop independiente.
- Discord OAuth.
- Multijugador.
- PvP en tiempo real.
- Persistencia de victorias/derrotas de batalla.
- Efectos sobre inventario, ranking o progresión derivados del resultado de una batalla.
- Registro de resultados de batalla en backend.
- Likes sobre artworks.

La propuesta original mencionaba web/desktop; esta versión corrige el alcance para mantener exclusivamente portal web y aplicación móvil. La implementación OAuth queda limitada a autenticación propia y Google.
