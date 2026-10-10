# Actores y permisos

Actores del sistema y lo que cada uno puede (y no puede) hacer. Los casos de uso del [catálogo](README.md#catálogo-de-casos-de-uso) indican qué actor los ejecuta.

## Invitado

Puede:

- explorar obras;
- buscar y filtrar obras;
- consultar detalles públicos de obras;
- explorar cartas y perfiles públicos, según la información disponible;
- registrarse;
- iniciar sesión;
- recuperar una contraseña olvidada.

No puede:

- publicar obras;
- comentar;
- calificar;
- denunciar;
- seguir usuarios;
- gestionar colección propia;
- abrir paquetes;
- jugar como usuario autenticado.

## Usuario autenticado

Un mismo usuario puede comportarse funcionalmente como artista y coleccionista; no se requieren cuentas separadas. Esta decisión coincide con la observación incluida en los casos de uso originales.

Puede:

- administrar su perfil, incluyendo cambiar su email o su contraseña;
- publicar, editar o eliminar sus propias obras mientras estén en período de calificación;
- comentar;
- calificar obras en período activo;
- denunciar obras o comentarios;
- seguir usuarios;
- marcar favoritos;
- administrar su colección;
- abrir paquetes disponibles;
- iniciar una batalla local;
- consultar notificaciones.

## Moderador

Puede:

- acceder al panel de moderación;
- consultar reportes pendientes;
- revisar el contenido reportado;
- resolver reportes;
- emitir advertencias;
- eliminar obras o comentarios;
- banear usuarios;
- descartar reportes.

## Admin

Puede hacer todo lo que puede hacer un moderador, más:

- otorgar y revocar el rol `moderator` a otros usuarios;
- revisar reportes ya resueltos (archivados) y reabrirlos, revirtiendo la sanción aplicada (restaurar obra/comentario o desbanear al usuario) sin pasar por el flujo de apelación;
- cargar directamente una obra ya convertida en carta (imagen + rareza + estadísticas), sin pasar por el período de calificación real;
- disparar manualmente el ciclo de conversión que normalmente corre por CRON, para no depender del reloj (pensado para demos).

El rol `admin` no se otorga ni se revoca a través de la aplicación — es un rol de confianza asignado fuera de este flujo (p. ej. directamente en base de datos).
