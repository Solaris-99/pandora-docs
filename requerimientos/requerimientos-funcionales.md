# 3. Requerimientos funcionales

| ID | Requerimiento | Descripción |
|---|---|---|
| RF01 | Registro | Permitir crear una cuenta mediante email, username y contraseña; opcionalmente mediante Google. |
| RF02 | Verificación de email | Las cuentas creadas mediante credenciales deben validar su email antes de quedar plenamente habilitadas. |
| RF03 | Login | Permitir autenticación mediante credenciales o Google. |
| RF03b | Recuperación de contraseña | Un usuario que olvidó su contraseña puede solicitar un enlace de un solo uso por email para establecer una nueva, sin exponer si la cuenta existe o si es una cuenta exclusivamente de Google. |
| RF04 | Perfil | Consultar y editar información propia y consultar perfiles públicos. |
| RF05 | Publicación de obra | Crear una obra con título, imagen y datos opcionales como descripción/Lore y tags. |
| RF06 | Gestión de obras | El autor puede consultar y gestionar sus propias obras según su estado. |
| RF07 | Explorador de obras | Explorar obras con filtros por fecha, tags, título, descripción, autor y calificación disponible. |
| RF08 | Detalle de obra | Mostrar imagen, autor, fechas, tags, descripción, comentarios y datos de calificación permitidos. |
| RF09 | Comentarios | Los usuarios autenticados pueden comentar obras. Las obras permanecen comentables después de finalizar la calificación. |
| RF10 | Q2Q | Durante una semana, los usuarios autenticados pueden asignar hasta 5 estrellas y una emoción definida a las obras en período de calificación. |
| RF11 | Conversión a carta | Al finalizar el período de calificación, las obras elegibles son procesadas por un CRON y pueden generar una carta. |
| RF12 | Estadísticas de carta | La carta generada posee ataque, defensa, vida, velocidad y rareza derivados del proceso de conversión. |
| RF13 | Colección | Consultar cartas propias, cantidad de copias y cartas aún no obtenidas. |
| RF14 | Detalle de carta | Mostrar estadísticas completas si el usuario posee la carta y aplicar ocultamiento cuando no la posee, según las reglas de colección. |
| RF15 | Paquetes | Permitir abrir paquetes disponibles y recibir cartas, admitiendo repetidas. |
| RF16 | Regeneración de paquetes | Regenerar un paquete por minuto hasta un máximo de 10. Si se alcanza el máximo, la regeneración se detiene. Un usuario sin email verificado tiene siempre 0 paquetes; al verificarlo recibe 10. |
| RF17 | Autobattler | Seleccionar una carta propia y solicitar al backend una carta rival aleatoria. La batalla se resuelve localmente en el cliente. |
| RF18 | Seguimiento | Seguir y dejar de seguir usuarios. |
| RF19 | Favoritos | Destacar obras propias y cartas de la colección. |
| RF20 | Notificaciones | Generar y consultar notificaciones producidas por acontecimientos relevantes. |
| RF21 | Reportes de obras | Registrar denuncias sobre obras mediante motivos predefinidos y comentario opcional. |
| RF22 | Reportes de comentarios | Registrar denuncias sobre comentarios. |
| RF23 | Moderación | Revisar y resolver reportes mediante descarte, advertencia, eliminación o baneo según corresponda. |
| RF24 | Estados de contenido | Respetar estados de publicación, eliminación, conversión, moderación y calificación. |
| RF25 | Gestión de moderadores | Un admin puede otorgar o revocar el rol `moderator` a cualquier usuario. |
| RF26 | Reapertura de reportes | Un admin puede reabrir un reporte ya resuelto con sanción punitiva, revirtiendo la sanción aplicada. |
| RF27 | Apelaciones | Un usuario cuya obra, comentario o cuenta fue sancionada por un reporte resuelto puede apelar esa sanción una única vez, indicando un motivo predefinido y un texto opcional. Un moderador o admin distinto de quien resolvió el reporte original (salvo que sea el único miembro de moderación) revisa la apelación y la aprueba o rechaza. |
| RF28 | Carga directa de carta (Admin) | Un admin puede subir una imagen junto con rareza y estadísticas de carta; el sistema crea la obra ya convertida (sin período de calificación) y la carta en un mismo paso. |
| RF29 | Disparo manual de conversión (Admin) | Un admin puede ejecutar bajo demanda el mismo ciclo que normalmente corre el CRON, para no depender del reloj. Pensado para demos. |

> Trazabilidad RF ↔ caso de uso ↔ componente: ver [documento de trazabilidad](/trazabilidad/requerimientos-funcionales.md).
