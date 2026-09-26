# Pandora — Requerimientos

Este documento cubre exclusivamente: descripción del proyecto, alcance, requerimientos funcionales, requerimientos no funcionales y reglas de negocio.

**Versión:** 1.0
**Proyecto:** Pandora
**Plataformas:** Portal Web + Aplicación Android
**Backend:** NestJS + TypeScript
**Base de datos:** PostgreSQL + TypeORM
**Almacenamiento de imágenes:** Cloudinary

---

# 1. Descripción del proyecto

Pandora es una plataforma que integra publicación de arte digital, curaduría comunitaria, coleccionismo de cartas y un juego de cartas automatizado. Los usuarios pueden publicar obras, explorar contenido, comentar, valorar obras durante un período limitado mediante estrellas y emociones, y participar en un sistema donde la percepción de la comunidad contribuye a determinar los atributos de una carta asociada a la obra.

Una obra publicada puede convertirse opcionalmente en una carta. La conversión sólo se realiza cuando el período de calificación termina y la obra está marcada para conversión. El procesamiento será realizado por un proceso CRON que detecta las obras pendientes y genera las cartas correspondientes.

Las cartas obtenidas pasan a formar parte de la colección del usuario mediante aperturas de paquetes. Las cartas pueden utilizarse en un autobattler completamente local al cliente. La plataforma también incluye perfiles públicos, seguimiento de usuarios, favoritos, comentarios, reportes, moderación y notificaciones.

La idea original define el ecosistema como una combinación de exposición artística, evaluación comunitaria y juego/coleccionismo. Esta guía mantiene ese enfoque, pero ajusta las partes que fueron posteriormente descartadas o modificadas.

Fuente base: la propuesta define web, backend centralizado y aplicación móvil, junto con publicación de obras, Q2Q, conversión de cartas, colección y módulo de juego.

La batalla se considera una funcionalidad local del cliente: el backend sólo interviene para seleccionar una carta rival. No existe PvP, sincronización de partida ni persistencia de victorias o derrotas.

---

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

---

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
| RF16 | Regeneración de paquetes | Regenerar un paquete por minuto hasta un máximo de 10. Si se alcanza el máximo, la regeneración se detiene. |
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

> Trazabilidad RF ↔ caso de uso ↔ componente: ver `pandora-trazabilidad.md`.

---

# 4. Requerimientos no funcionales

| ID    | Requerimiento              | Regla / criterio                                                                                                                                                                                                                         |
| ----- | -------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| RNF01 | Peso de obras              | Máximo 25 MB por archivo. Formatos PNG, JPG, JPEG y WEBP.                                                                                                                                                                                |
| RNF02 | Hash de contraseñas        | Nunca almacenar contraseñas en texto plano. Utilizar un algoritmo de hash seguro y robusto.                                                                                                                                              |
| RNF03 | Rate limit de login        | Limitar intentos fallidos de autenticación y bloquear temporalmente nuevos intentos.                                                                                                                                                     |
| RNF04 | Roles                      | Sólo los roles autorizados pueden utilizar funciones administrativas/moderación.                                                                                                                                                         |
| RNF05 | Usabilidad                 | Interfaz intuitiva para invitados, artistas y coleccionistas.                                                                                                                                                                            |
| RNF06 | Storage de imágenes        | Mantener original y generar versiones optimizadas/thumbnails. En esta versión se utilizará Cloudinary.                                                                                                                                   |
| RNF07 | Verificación de email      | Registrar usuarios con credenciales requiere completar un challenge de verificación.                                                                                                                                                     |
| RNF08 | Seguridad de endpoints     | Los endpoints deben estar protegidos contra acceso no autorizado.<br>Esto implica al menos:<br>- autenticación;<br>- autorización;<br>- ownership checks;<br>- validación;<br>- rate limiting;<br>- control de abuso;<br>- sanitización; |
| RNF09 | Sanidad de campos          | Sanitizar, normalizar y limitar campos de texto. Valores guía: credenciales hasta 30 caracteres, títulos hasta 50 y descripciones hasta 300.                                                                                             |
| RNF10 | Manejo de errores          | Informar errores de manera comprensible sin exponer información sensible ni interrumpir innecesariamente el uso.                                                                                                                         |
| RNF11 | Integridad                 | Las operaciones que modifican inventario, paquetes o conversión deben mantener consistencia mediante transacciones cuando corresponda.                                                                                                   |
| RNF12 | Idempotencia de conversión | El CRON/scheduler no debe generar dos cartas para una misma obra.                                                                                                                                                                        |
| RNF13 | Auditoría de moderación    | Registrar quién resolvió un reporte, cuándo y qué acción tomó.                                                                                                                                                                           |
| RNF14 | Observabilidad             | Registrar errores de aplicación, ejecuciones del CRON y fallas de integraciones externas.                                                                                                                                                |
| RNF15 | Paginación                 | Las consultas de galería, comentarios, notificaciones y reportes deben utilizar paginación.                                                                                                                                              |

> El estado de implementación de cada RNF (qué componente concreto lo satisface) se documenta en `pandora-trazabilidad.md` y en `pandora-diseno-componentes.md`/`pandora-diseno-arquitectura.md`.

---

# 5. Reglas de negocio

## 5.1 Usuarios

1. Un username debe ser único.
2. Un email debe ser único.
3. Un usuario autenticado es simultáneamente potencial artista y coleccionista.
4. Una cuenta local debe verificar el email.
5. Google puede utilizarse como proveedor alternativo de autenticación.
6. Discord OAuth no forma parte del sistema.
7. Un usuario con contraseña local puede solicitar recuperarla mediante un enlace de un solo uso enviado por email, válido por 1 hora. Una cuenta creada únicamente vía Google (sin contraseña local) no tiene nada que recuperar por esta vía: la solicitud responde igual que para cualquier otra dirección (mensaje neutro), pero no se envía correo.
8. Restablecer la contraseña revoca todas las sesiones (`refresh_tokens`) activas del usuario; no inicia sesión automáticamente.

## 5.2 Obras

1. Una obra requiere título e imagen.
2. La descripción/Lore es opcional.
3. Los tags son opcionales.
4. Una obra pertenece a un único autor.
5. Una obra puede permanecer comentable después de finalizar el período de Q2Q.
6. La finalización del período sólo deshabilita nuevas calificaciones.
7. El período de calificación dura **1 semana** desde el inicio definido para la obra.
8. Una obra puede optar por convertirse o no en carta.
9. Una obra no debe convertirse más de una vez.
10. Las obras eliminadas no deben aparecer como contenido público activo.

## 5.3 Calificaciones Q2Q

1. Sólo usuarios autenticados pueden valorar.
2. Una valoración tiene entre 1 y 5 estrellas.
3. Debe elegirse una emoción del conjunto permitido.
4. Un usuario sólo puede tener una valoración activa por obra; si califica otra vez, se actualiza la existente.
5. Una vez finalizado el período de una semana, no pueden crearse nuevas valoraciones.
6. Los comentarios no se bloquean al terminar el período.

## 5.4 Conversión

1. El CRON/scheduler busca obras cuyo período de calificación terminó.
2. Sólo procesa obras elegibles para conversión.
3. Sólo procesa obras que aún no fueron convertidas.
4. Se calcula las estadísticas base: `( b * e ) * (r + s)` donde b es una base dependiendo del stat, r es la rareza, s es el promedio de estrellas y e es el bonus de la emoción. Ver las tablas en 5.10.
5. Guarda la carta generada.
6. Actualiza el estado de conversión de la obra.
7. El proceso debe ser idempotente.
8. Un admin puede disparar el mismo ciclo manualmente, fuera del horario del CRON (pensado para demos) — no es un proceso distinto, reutiliza exactamente la misma rutina.

## 5.5 Cartas y colección

1. Una obra puede generar como máximo una carta.
2. Un usuario puede poseer múltiples copias de una misma carta.
3. Las copias se almacenan como contador.
4. Al obtener una carta repetida se incrementa el contador.
5. El detalle de una carta no poseída puede ocultar estadísticas.
6. La obra original sigue siendo visible en sus colores normales aun cuando las estadísticas de la carta sean desconocidas para el usuario.
7. Un admin puede crear una obra + carta directamente (sin período de calificación), indicando rareza y, opcionalmente, estadísticas explícitas; las que no se indiquen se completan con el mismo cálculo que usa el catálogo mínimo sembrado, a partir únicamente de la rareza. El admin no pasa a poseer automáticamente esa carta por haberla creado.

## 5.6 Paquetes

1. El máximo de paquetes almacenados por usuario es 10.
2. Abrir un paquete reduce el total en 1.
3. La regeneración ocurre cada 1 minuto.
4. Nunca se supera el máximo de 10.
5. Si el usuario ya posee 10, no se genera deuda de regeneración.
6. La cantidad debe poder calcularse de forma consistente incluso si el usuario estuvo desconectado.
7. El usuario obtiene 5 cartas por paquete abierto.

## 5.7 Rarezas

1. La rareza de una carta es determinada en el período de calificación, y es definida únicamente por la cantidad de votos (emociones/estrellas) sin importar su valor.
2. La rareza es definida por `v/(u * C)` donde v es la cantidad de votos, u la cantidad de usuarios registrados totales y C una variable de entorno; C es una constante que pretende representar los usuarios activos (en promedio).
3. Los paquetes dan cartas: 64% común, 20% poco común (uncommon), 10% raras, 5% épicas, 1% legendarias.

## 5.8 Batallas

1. El usuario elige una carta que posee.
2. El backend devuelve una carta rival aleatoria válida.
3. La carta rival no representa a otro jugador.
4. La simulación es completamente local.
5. No se guardan victorias ni derrotas.
6. El resultado no cambia inventario ni estadísticas persistentes.
7. Manipular el cliente no afecta datos relevantes del servidor dentro de este alcance.

## 5.9 Generales

1. Todas las eliminaciones son lógicas.

## 5.10 Estadísticas base, emociones y rarezas

| Estadística | Base |
| ----------- | ---- |
| HP/Vida     | 100  |
| Velocidad   | 1    |
| Ataque      | 10   |
| Defensa     | 10   |

Efecto de las emociones en las diferentes estadísticas:

| Stat      | Alegría | Miedo | Enojo | Pesar | Serenidad | Admiración |
| --------- | ------- | ----- | ----- | ----- | --------- | ---------- |
| HP        | 2       | 1     | 1     | 1.5   | 1         | 1.25       |
| Velocidad | 1       | 1.5   | 1     | 1     | 1         | 1.25       |
| Ataque    | 1       | 1.5   | 2     | 1     | 1         | 1.25       |
| Defensa   | 1       | 1     | 1     | 1.5   | 2         | 1.25       |

Efecto de las rarezas (valor r):

| Rareza    | Valor |
| --------- | ----- |
| common    | 1     |
| uncommon  | 2     |
| rare      | 4     |
| epic      | 6     |
| legendary | 9     |

## 5.11 Apelaciones y administración

1. Toda apelación remite a un único reporte ya resuelto con una sanción punitiva (`content_removed`, `comment_removed` o `user_banned`); un reporte resuelto como `dismissed` o `warning` no tiene nada que apelar ni que reabrir.
2. Un reporte sólo puede apelarse una vez: si la apelación es rechazada, no puede volver a apelarse la misma sanción.
3. Sólo el autor de la obra/comentario sancionado, o el propio usuario baneado, puede apelar esa sanción.
4. El moderador o admin que revisa una apelación no puede ser quien resolvió el reporte original, salvo que en ese momento exista exactamente un miembro de personal de moderación (`moderator` + `admin` contados juntos).
5. Aprobar una apelación, o reabrir un reporte directamente como admin, es una reversión de un solo sentido: deshace exactamente el efecto de la sanción original (restaura el contenido o desbanea al usuario). No permite reemplazar la sanción por otra distinta en el mismo paso.
6. Un usuario baneado puede autenticarse igual que cualquier otro usuario, pero sus operaciones de escritura quedan bloqueadas salvo apelar su propio baneo, consultar el estado de sus apelaciones y consultar/gestionar sus notificaciones.
7. La suspensión (`UserStatus.suspended`) no forma parte del sistema de apelaciones: una cuenta suspendida sigue sin poder autenticarse, sin excepción.

---

