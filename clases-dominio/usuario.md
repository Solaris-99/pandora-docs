# Usuario

Representa a cualquier persona registrada en la plataforma. Un mismo Usuario es simultáneamente artista potencial y coleccionista potencial — no existen subtipos de cuenta.

- **Atributos conceptuales:** nombre de usuario, email, contraseña (sólo si se registró localmente), estado de la cuenta (activo/suspendido/baneado/pendiente de verificación), biografía, avatar, email verificado (sí/no).
- **Responsabilidades:** autenticarse; publicar Obras; comentar; calificar; seguir a otros Usuarios; destacar Obras/Cartas propias o poseídas; poseer una Colección de Cartas y un Inventario de Paquetes; recibir Notificaciones; (si tiene rol de moderación) resolver Reportes y revisar Apelaciones; (si es admin) gestionar Roles y disparar herramientas administrativas.
- **Relaciones:** autor de N Obras · autor de N Comentarios · emisor de N Valoraciones · posee N Cartas (a través de la Colección, con contador de copias) · posee 1 Inventario de Paquetes · sigue a N Usuarios / es seguido por N Usuarios · destaca N Obras y N Cartas · recibe N Notificaciones · tiene N Roles · presenta N Apelaciones (como apelante) · revisa N Apelaciones (como moderador/admin) · resuelve N Reportes (como moderador/admin).
