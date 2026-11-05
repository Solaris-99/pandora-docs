# Pandora — Prototipo

Recorrido por las pantallas de la aplicación web, en escritorio y en móvil (diseño responsive). Cada pantalla tiene una descripción breve y los [casos de uso](/casos-de-uso/README.md) con los que se relaciona.

Hacé clic en una captura para ampliarla.

---

## Secciones

| Sección | Contenido | Pantallas |
|---|---|---|
| [Pantallas generales](general.md) | Puntos de entrada al sitio y páginas informativas. | 4 |
| [Cuenta](cuenta.md) | Alta, acceso y recuperación de la cuenta, y configuración del perfil propio. | 12 |
| [Obras](obras.md) | Exploración, publicación y participación sobre las obras. | 12 |
| [Cartas y juego](cartas-y-juego.md) | Catálogo de cartas, colección personal, apertura de paquetes y batalla. | 9 |
| [Social y notificaciones](social.md) | Perfiles, seguimiento, destacados y notificaciones. | 5 |
| [Moderación y apelaciones](moderacion.md) | Gestión de denuncias por el equipo de moderación, y apelación de sanciones por parte de los usuarios. | 10 |
| [Administración](administracion.md) | Herramientas exclusivas del rol admin. | 3 |

También podés ver el [documento completo](completo.md), con todas las pantallas en una sola página.

---

## Sobre las capturas

- Son capturas de la aplicación real, tomadas el 2026-11-05 sobre la rama `dev` del frontend.
- **Los datos son ilustrativos.** Las respuestas de la API se simularon para poder mostrar cada rol y estado, así que los usuarios, obras, cartas y denuncias no existen. Las imágenes son de relleno ([Lorem Picsum](/interfaces-entre-sistemas/lorem-picsum.md)) y no guardan relación con los títulos.
- Escritorio: 1440 × 900. Móvil: 390 × 844. Tema claro. La barra de navegación y el pie se repiten en todas las pantallas.
- Las capturas de diálogos y de apertura de paquetes muestran sólo la parte visible de la pantalla; el resto, la página completa.

---

## Casos de uso y pantallas

Relación inversa: para cada caso de uso, las pantallas donde se ejecuta o se ve su resultado.

| Caso de uso | Pantallas |
|---|---|
| [CU01 — Registrarse](/casos-de-uso/cu01-registrarse.md) | [Registro](cuenta.md#registro) |
| [CU02 — Verificar email](/casos-de-uso/cu02-verificar-email.md) | [Registro](cuenta.md#registro), [Verificar email: pendiente](cuenta.md#verificar-email-pendiente), [Verificar email: confirmado](cuenta.md#verificar-email-confirmado), [Verificar email: enlace inválido](cuenta.md#verificar-email-enlace-inválido) |
| [CU03 — Login](/casos-de-uso/cu03-login.md) | [Inicio de sesión](cuenta.md#inicio-de-sesión), [Error de inicio de sesión con Google](cuenta.md#error-de-inicio-de-sesión-con-google) |
| [CU04 — Recuperar contraseña olvidada](/casos-de-uso/cu04-recuperar-contrasena-olvidada.md) | [Inicio de sesión](cuenta.md#inicio-de-sesión), [Recuperar contraseña](cuenta.md#recuperar-contraseña), [Recuperar contraseña: solicitud enviada](cuenta.md#recuperar-contraseña-solicitud-enviada), [Restablecer contraseña](cuenta.md#restablecer-contraseña) |
| [CU05 — Limpieza de cuentas no verificadas](/casos-de-uso/cu05-limpieza-de-cuentas-no-verificadas.md) | Sin pantalla: es una tarea programada (CRON) sin interfaz. |
| [CU06 — Ver/editar perfil](/casos-de-uso/cu06-ver-editar-perfil.md) | [Configuración de perfil](cuenta.md#configuración-de-perfil), [Perfil de otro usuario](social.md#perfil-de-otro-usuario), [Perfil propio](social.md#perfil-propio) |
| [CU07 — Cambiar email](/casos-de-uso/cu07-cambiar-email.md) | [Confirmar cambio de email](cuenta.md#confirmar-cambio-de-email), [Configuración de perfil](cuenta.md#configuración-de-perfil), [Diálogo: confirmar cambio de email](cuenta.md#diálogo-confirmar-cambio-de-email) |
| [CU08 — Cambiar contraseña](/casos-de-uso/cu08-cambiar-contrasena.md) | [Configuración de perfil](cuenta.md#configuración-de-perfil) |
| [CU09 — Subir obra](/casos-de-uso/cu09-subir-obra.md) | [Inicio con sesión iniciada](general.md#inicio-con-sesión-iniciada), [Subir obra](obras.md#subir-obra), [Mis obras](obras.md#mis-obras) |
| [CU10 — Editar obra](/casos-de-uso/cu10-editar-obra.md) | [Detalle de obra: autor](obras.md#detalle-de-obra-autor), [Detalle de obra: ya convertida](obras.md#detalle-de-obra-ya-convertida), [Editar obra](obras.md#editar-obra), [Mis obras](obras.md#mis-obras) |
| [CU11 — Eliminar obra](/casos-de-uso/cu11-eliminar-obra.md) | [Detalle de obra: autor](obras.md#detalle-de-obra-autor), [Detalle de obra: ya convertida](obras.md#detalle-de-obra-ya-convertida), [Mis obras](obras.md#mis-obras), [Diálogo: eliminar obra](obras.md#diálogo-eliminar-obra) |
| [CU12 — Explorar obras](/casos-de-uso/cu12-explorar-obras.md) | [Inicio](general.md#inicio), [Inicio con sesión iniciada](general.md#inicio-con-sesión-iniciada), [Nuevas obras](obras.md#nuevas-obras), [Todas las obras](obras.md#todas-las-obras), [Detalle de obra: invitado](obras.md#detalle-de-obra-invitado), [Detalle de obra: usuario](obras.md#detalle-de-obra-usuario) |
| [CU13 — Comentar obra](/casos-de-uso/cu13-comentar-obra.md) | [Detalle de obra: usuario](obras.md#detalle-de-obra-usuario) |
| [CU14 — Calificar obra mediante Q2Q](/casos-de-uso/cu14-calificar-obra-mediante-q2q.md) | [Nuevas obras](obras.md#nuevas-obras), [Detalle de obra: usuario](obras.md#detalle-de-obra-usuario) |
| [CU15 — Conversión de obra a carta](/casos-de-uso/cu15-conversion-de-obra-a-carta.md) | [Todas las obras](obras.md#todas-las-obras), [Detalle de obra: ya convertida](obras.md#detalle-de-obra-ya-convertida), [Notificaciones](social.md#notificaciones), [Herramientas de administración](administracion.md#herramientas-de-administración), [Ciclo de conversión: resultado](administracion.md#ciclo-de-conversión-resultado) |
| [CU16 — Explorar colección/cartas](/casos-de-uso/cu16-explorar-coleccion-cartas.md) | [Inicio](general.md#inicio), [Inicio con sesión iniciada](general.md#inicio-con-sesión-iniciada), [Explorar cartas](cartas-y-juego.md#explorar-cartas), [Detalle de carta: poseída](cartas-y-juego.md#detalle-de-carta-poseída), [Detalle de carta: no poseída](cartas-y-juego.md#detalle-de-carta-no-poseída), [Colección](cartas-y-juego.md#colección) |
| [CU17 — Abrir paquete](/casos-de-uso/cu17-abrir-paquete.md) | [Colección](cartas-y-juego.md#colección), [Abrir paquete: revelado](cartas-y-juego.md#abrir-paquete-revelado), [Abrir paquete: resumen](cartas-y-juego.md#abrir-paquete-resumen), [Batalla: elegir carta](cartas-y-juego.md#batalla-elegir-carta) |
| [CU18 — Jugar autobattler](/casos-de-uso/cu18-jugar-autobattler.md) | [Batalla: invitado](cartas-y-juego.md#batalla-invitado), [Batalla: elegir carta](cartas-y-juego.md#batalla-elegir-carta), [Batalla: resultado](cartas-y-juego.md#batalla-resultado) |
| [CU19 — Seguir usuario](/casos-de-uso/cu19-seguir-usuario.md) | [Perfil de otro usuario](social.md#perfil-de-otro-usuario), [Seguidores y seguidos](social.md#seguidores-y-seguidos) |
| [CU20 — Destacar obra o carta](/casos-de-uso/cu20-destacar-obra-o-carta.md) | [Detalle de obra: autor](obras.md#detalle-de-obra-autor), [Detalle de obra: ya convertida](obras.md#detalle-de-obra-ya-convertida), [Mis obras](obras.md#mis-obras), [Detalle de carta: poseída](cartas-y-juego.md#detalle-de-carta-poseída), [Colección](cartas-y-juego.md#colección), [Perfil propio](social.md#perfil-propio), [Destacados](social.md#destacados) |
| [CU21 — Denunciar obra](/casos-de-uso/cu21-denunciar-obra.md) | [Detalle de obra: usuario](obras.md#detalle-de-obra-usuario), [Diálogo: denunciar obra](obras.md#diálogo-denunciar-obra), [Denuncia de obra](moderacion.md#denuncia-de-obra) |
| [CU22 — Denunciar comentario](/casos-de-uso/cu22-denunciar-comentario.md) | [Detalle de obra: usuario](obras.md#detalle-de-obra-usuario), [Diálogo: denunciar comentario](obras.md#diálogo-denunciar-comentario), [Denuncia de comentario](moderacion.md#denuncia-de-comentario) |
| [CU23 — Administrar denuncias](/casos-de-uso/cu23-administrar-denuncias.md) | [Cola de denuncias](moderacion.md#cola-de-denuncias), [Denuncia de obra](moderacion.md#denuncia-de-obra), [Denuncia de comentario](moderacion.md#denuncia-de-comentario) |
| [CU24 — Notificaciones](/casos-de-uso/cu24-notificaciones.md) | [Inicio con sesión iniciada](general.md#inicio-con-sesión-iniciada), [Notificaciones](social.md#notificaciones) |
| [CU25 — Gestionar rol de moderador](/casos-de-uso/cu25-gestionar-rol-de-moderador.md) | [Perfil visto por un admin](administracion.md#perfil-visto-por-un-admin) |
| [CU26 — Reabrir un reporte resuelto](/casos-de-uso/cu26-reabrir-un-reporte-resuelto.md) | [Cola de denuncias](moderacion.md#cola-de-denuncias), [Denuncia resuelta: reabrir](moderacion.md#denuncia-resuelta-reabrir) |
| [CU27 — Apelar una sanción](/casos-de-uso/cu27-apelar-una-sancion.md) | [Notificaciones](social.md#notificaciones), [Mis apelaciones](moderacion.md#mis-apelaciones), [Diálogo: apelar contenido eliminado](moderacion.md#diálogo-apelar-contenido-eliminado), [Cuenta suspendida](moderacion.md#cuenta-suspendida), [Diálogo: apelar suspensión](moderacion.md#diálogo-apelar-suspensión) |
| [CU28 — Revisar una apelación](/casos-de-uso/cu28-revisar-una-apelacion.md) | [Cola de apelaciones](moderacion.md#cola-de-apelaciones), [Revisar apelación](moderacion.md#revisar-apelación) |
| [CU29 — Cargar carta directamente (Admin)](/casos-de-uso/cu29-cargar-carta-directamente.md) | [Herramientas de administración](administracion.md#herramientas-de-administración) |
| [CU30 — Disparar el ciclo de conversión manualmente (Admin)](/casos-de-uso/cu30-disparar-el-ciclo-de-conversion-manualmente.md) | [Herramientas de administración](administracion.md#herramientas-de-administración), [Ciclo de conversión: resultado](administracion.md#ciclo-de-conversión-resultado) |
| [CU31 — Subir avatar](/casos-de-uso/cu31-subir-avatar.md) | [Configuración de perfil](cuenta.md#configuración-de-perfil) |
