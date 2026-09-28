# Pandora — Documento de diseño general

> Documento técnico-funcional consolidado a partir de la propuesta de proyecto, requerimientos, casos de uso y definición inicial de datos, incorporando las decisiones posteriores indicadas para el diseño del sistema.

**Versión:** 1.0  
**Proyecto:** Pandora  
**Plataformas:** Portal Web + Aplicación Android  
**Backend:** NestJS + TypeScript  
**Base de datos:** PostgreSQL + TypeORM  
**Almacenamiento de imágenes:** Cloudinary

---

## 1. Propósito del documento

Este documento establece una guía de desarrollo para Pandora. Su objetivo es unificar la definición funcional y técnica del sistema y servir como referencia durante la implementación del backend, portal web y aplicación móvil.

El documento consolida:

- descripción y alcance del producto;
- actores y permisos;
- requerimientos funcionales y no funcionales;
- reglas de negocio;
- casos de uso detallados, incluyendo flujos alternativos y excepciones;
- entidades, atributos y relaciones principales;
- constraints de base de datos;
- endpoints REST propuestos;
- guards, autorización y validaciones;
- arquitectura sugerida;
- procesamiento periódico de conversión de obras a cartas;
- notificaciones;
- moderación y reportes;
- observaciones y decisiones de diseño.

La batalla se considera una funcionalidad local del cliente: el backend sólo interviene para seleccionar una carta rival. No existe PvP, sincronización de partida ni persistencia de victorias o derrotas.

---

# 2. Descripción del proyecto

Pandora es una plataforma que integra publicación de arte digital, curaduría comunitaria, coleccionismo de cartas y un juego de cartas automatizado. Los usuarios pueden publicar obras, explorar contenido, comentar, valorar obras durante un período limitado mediante estrellas y emociones, y participar en un sistema donde la percepción de la comunidad contribuye a determinar los atributos de una carta asociada a la obra.

Una obra publicada puede convertirse opcionalmente en una carta. La conversión sólo se realiza cuando el período de calificación termina y la obra está marcada para conversión. El procesamiento será realizado por un proceso CRON que detecta las obras pendientes y genera las cartas correspondientes.

Las cartas obtenidas pasan a formar parte de la colección del usuario mediante aperturas de paquetes. Las cartas pueden utilizarse en un autobattler completamente local al cliente. La plataforma también incluye perfiles públicos, seguimiento de usuarios, favoritos, comentarios, reportes, moderación y notificaciones.

La idea original define el ecosistema como una combinación de exposición artística, evaluación comunitaria y juego/coleccionismo. Esta guía mantiene ese enfoque, pero ajusta las partes que fueron posteriormente descartadas o modificadas.

Fuente base: la propuesta define web, backend centralizado y aplicación móvil, junto con publicación de obras, Q2Q, conversión de cartas, colección y módulo de juego. 

---

# 3. Alcance

## 3.1 Incluido

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

## 3.2 Fuera del alcance actual

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

# 4. Actores y permisos

## 4.1 Invitado

Puede:

- explorar obras;
- buscar y filtrar obras;
- consultar detalles públicos de obras;
- explorar cartas y perfiles públicos, según la información disponible;
- registrarse;
- iniciar sesión.

No puede:

- publicar obras;
- comentar;
- calificar;
- denunciar;
- seguir usuarios;
- gestionar colección propia;
- abrir paquetes;
- jugar como usuario autenticado.

## 4.2 Usuario autenticado

Un mismo usuario puede comportarse funcionalmente como artista y coleccionista; no se requieren cuentas separadas. Esta decisión coincide con la observación incluida en los casos de uso originales.

Puede:

- administrar su perfil;
- publicar obras;
- comentar;
- calificar obras en período activo;
- denunciar obras o comentarios;
- seguir usuarios;
- marcar favoritos;
- administrar su colección;
- abrir paquetes disponibles;
- iniciar una batalla local;
- consultar notificaciones.

## 4.3 Moderador

Puede:

- acceder al panel de moderación;
- consultar reportes pendientes;
- revisar el contenido reportado;
- resolver reportes;
- emitir advertencias;
- eliminar obras o comentarios;
- banear usuarios;
- descartar reportes.

## 4.4 Admin

Puede hacer todo lo que puede hacer un moderador, más:

- otorgar y revocar el rol `moderator` a otros usuarios;
- revisar reportes ya resueltos (archivados) y reabrirlos, revirtiendo la sanción aplicada (restaurar obra/comentario o desbanear al usuario) sin pasar por el flujo de apelación;
- cargar directamente una obra ya convertida en carta (imagen + rareza + estadísticas), sin pasar por el período de calificación real;
- disparar manualmente el ciclo de conversión que normalmente corre por CRON, para no depender del reloj (pensado para demos).

El rol `admin` no se otorga ni se revoca a través de la aplicación — es un rol de confianza asignado fuera de este flujo (p. ej. directamente en base de datos).

---

# 5. Requerimientos funcionales

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

---

# 6. Requerimientos no funcionales

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

---

# 7. Reglas de negocio

## 7.1 Usuarios

1. Un username debe ser único.
2. Un email debe ser único.
3. Un usuario autenticado es simultáneamente potencial artista y coleccionista.
4. Una cuenta local debe verificar el email.
5. Google puede utilizarse como proveedor alternativo de autenticación.
6. Discord OAuth no forma parte del sistema.
7. Un usuario con contraseña local puede solicitar recuperarla mediante un enlace de un solo uso enviado por email, válido por 1 hora. Una cuenta creada únicamente vía Google (sin contraseña local) no tiene nada que recuperar por esta vía: la solicitud responde igual que para cualquier otra dirección (mensaje neutro), pero no se envía correo.
8. Restablecer la contraseña revoca todas las sesiones (`refresh_tokens`) activas del usuario; no inicia sesión automáticamente.

## 7.2 Obras

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

## 7.3 Calificaciones Q2Q

1. Sólo usuarios autenticados pueden valorar.
2. Una valoración tiene entre 1 y 5 estrellas.
3. Debe elegirse una emoción del conjunto permitido.
4. Un usuario sólo puede tener una valoración activa por obra; si califica otra vez, se actualiza la existente.
5. Una vez finalizado el período de una semana, no pueden crearse nuevas valoraciones.
6. Los comentarios no se bloquean al terminar el período.

## 7.4 Conversión

1. El CRON/scheduler busca obras cuyo período de calificación terminó.
2. Sólo procesa obras elegibles para conversión.
3. Sólo procesa obras que aún no fueron convertidas.
4. Se calcula las estadísticas base:  `( b * e ) * (r + s) ` donde b es una base dependiendo del stat r es la rareza, s es el promedio de estrellas y e es el bonus de la emoción. Ver las tablas más abajo.
5. Guarda la carta generada.
6. Actualiza el estado de conversión de la obra.
7. El proceso debe ser idempotente.
8. Un admin puede disparar el mismo ciclo manualmente, fuera del horario del CRON (pensado para demos) — no es un proceso distinto, reutiliza exactamente la misma rutina.

## 7.5 Cartas y colección

1. Una obra puede generar como máximo una carta.
2. Un usuario puede poseer múltiples copias de una misma carta.
3. Las copias se almacenan como contador.
4. Al obtener una carta repetida se incrementa el contador.
5. El detalle de una carta no poseída puede ocultar estadísticas.
6. La obra original sigue siendo visible en sus colores normales aun cuando las estadísticas de la carta sean desconocidas para el usuario.
7. Un admin puede crear una obra + carta directamente (sin período de calificación), indicando rareza y, opcionalmente, estadísticas explícitas; las que no se indiquen se completan con el mismo cálculo que usa el catálogo mínimo sembrado, a partir únicamente de la rareza. El admin no pasa a poseer automáticamente esa carta por haberla creado.

## 7.6 Paquetes

1. El máximo de paquetes almacenados por usuario es 10.
2. Abrir un paquete reduce el total en 1.
3. La regeneración ocurre cada 1 minuto.
4. Nunca se supera el máximo de 10.
5. Si el usuario ya posee 10, no se genera deuda de regeneración.
6. La cantidad debe poder calcularse de forma consistente incluso si el usuario estuvo desconectado.
7. El usuario obtiene 5 cartas por paquete abierto.

## 7.7 Rarezas
1. Las rareza de una carta es determinada en el periodo de calificación, y es definida únicamente por la cantidad de votos (emociones/estrellas) sin importar su valor.
2. La rareza es definida por `v/(u * C)` donde v es la cantidad de votos, u la cantidad de usuarios registrados totales y C una variable de entorno; C es una constante que pretende representar los usuarios activos (en promedio).
3. Los paquetes dan cartas: 64% común, 20% poco común (uncommon), 10% raras, 5% épicas, 1% legendarias.

## 7.8 Batallas

1. El usuario elige una carta que posee.
2. El backend devuelve una carta rival aleatoria válida.
3. La carta rival no representa a otro jugador.
4. La simulación es completamente local.
5. No se guardan victorias ni derrotas.
6. El resultado no cambia inventario ni estadísticas persistentes.
7. Manipular el cliente no afecta datos relevantes del servidor dentro de este alcance.

## 7.9 Generales
1. Todas las eliminaciones son lógicas.
## 7.10 Estadísticas base, emociones y rarezas

| Estadística | Base |
| ----------- | ---- |
| HP/Vida     | 100  |
| Velocidad   | 1    |
| Ataque      | 10   |
| Defensa     | 10   |

Efecto de las emociones en las diferentes estadísticas

| Stat      | Alegría | Miedo | Enojo | Pesar | Serenidad | Admiración |
| --------- | ------- | ----- | ----- | ----- | --------- | ---------- |
| HP        | 2       | 1     | 1     | 1.5   | 1         | 1.25       |
| Velocidad | 1       | 1.5   | 1     | 1     | 1         | 1.25       |
| Ataque    | 1       | 1.5   | 2     | 1     | 1         | 1.25       |
| Defensa   | 1       | 1     | 1     | 1.5   | 2         | 1.25       |

Efecto de las rarezas (valor r)

| Rareza    | Valor |
| --------- | ----- |
| common    | 1     |
| uncommon  | 2     |
| rare      | 4     |
| epic      | 6     |
| legendary | 9     |

## 7.11 Apelaciones y administración

1. Toda apelación remite a un único reporte ya resuelto con una sanción punitiva (`content_removed`, `comment_removed` o `user_banned`); un reporte resuelto como `dismissed` o `warning` no tiene nada que apelar ni que reabrir.
2. Un reporte sólo puede apelarse una vez: si la apelación es rechazada, no puede volver a apelarse la misma sanción.
3. Sólo el autor de la obra/comentario sancionado, o el propio usuario baneado, puede apelar esa sanción.
4. El moderador o admin que revisa una apelación no puede ser quien resolvió el reporte original, salvo que en ese momento exista exactamente un miembro de personal de moderación (`moderator` + `admin` contados juntos).
5. Aprobar una apelación, o reabrir un reporte directamente como admin, es una reversión de un solo sentido: deshace exactamente el efecto de la sanción original (restaura el contenido o desbanea al usuario). No permite reemplazar la sanción por otra distinta en el mismo paso.
6. Un usuario baneado puede autenticarse igual que cualquier otro usuario, pero sus operaciones de escritura quedan bloqueadas salvo apelar su propio baneo, consultar el estado de sus apelaciones y consultar/gestionar sus notificaciones.
7. La suspensión (`UserStatus.suspended`) no forma parte del sistema de apelaciones: una cuenta suspendida sigue sin poder autenticarse, sin excepción.

---

# 8. Casos de uso detallados

Los casos de uso cubren subir obra, calificar, explorar, colección, moderación, seguimiento, batalla, paquetes, destacados, reportes, notificaciones, registro y login. 

---

## CU01 — Registrarse

**Actor principal:** Invitado  
**Precondición:** No existe una sesión autenticada.  
**Postcondición:** Cuenta creada. Si es registro local, queda pendiente de verificación o en estado restringido hasta confirmar email.

### Flujo principal

1. El usuario accede a la pantalla de registro.
2. Introduce email, username y contraseña.
3. El backend valida formato, longitud y unicidad.
4. Se hashea la contraseña.
5. Se crea la cuenta.
6. Se genera un token/challenge de verificación.
7. Se envía el correo de verificación.
8. El usuario abre el enlace o introduce el código recibido.
9. El backend valida el challenge.
10. La cuenta queda verificada.
11. Se informa que el registro fue exitoso.

### Flujos alternativos / excepciones

- **A1:** El email ya existe. Se rechaza el registro.
- **A2:** El username ya existe. Se rechaza el registro.
- **A3:** Email inválido. Se devuelve error de validación.
- **A4:** Contraseña no válida. Se devuelve error de validación.
- **A5:** El envío del correo falla. La cuenta queda pendiente y se permite reintentar el envío.
- **A6:** Challenge expirado. Se solicita uno nuevo.
- **A7:** Challenge inválido. Se rechaza la confirmación.

### Notas de implementación

Los tokens de verificación deben expirar y ser de un solo uso.

---

## CU02 — Login

**Actor principal:** Invitado  
**Precondición:** Cuenta registrada.  
**Postcondición:** Sesión autenticada.

### Flujo principal

1. El usuario abre login.
2. Ingresa email y contraseña.
3. El backend aplica rate limit.
4. Busca el usuario.
5. Verifica credenciales.
6. Verifica el estado de la cuenta.
7. Genera sesión/JWT.
8. Devuelve el resultado de autenticación.

### Alternativo

- **A1:** Credenciales inválidas -> respuesta genérica de autenticación fallida.
- **A2:** Demasiados intentos fallidos -> bloqueo temporal.
- **A3:** Cuenta baneada -> acceso denegado.
- **A4:** Cuenta sin verificar -> se informa que debe completar la verificación.
- **A5:** Error del proveedor Google -> mensaje genérico de autenticación fallida.

---

## CU03 — Subir obra

**Actor principal:** Usuario autenticado  
**Precondición:** Usuario autenticado y autorizado.  
**Postcondición:** Obra almacenada y disponible en el ciclo de calificación si corresponde.

### Flujo principal

1. El usuario pulsa “Subir obra”.
2. Completa título.
3. Selecciona imagen.
4. Opcionalmente completa Lore/descripcion.
5. Opcionalmente agrega tags.
6. Indica si permite conversión a carta.
7. El cliente envía los datos al backend.
8. El backend valida campos y archivo.
9. Se carga la imagen a Cloudinary.
10. Cloudinary genera/permite servir versiones optimizadas.
11. Se registra la obra y la fecha/hora de inicio de calificación.
12. La obra queda en estado de calificación.

### Alternativo

- **A1:** Archivo > 25 MB -> rechazar.
- **A2:** Formato no permitido -> rechazar.
- **A3:** Título vacío o excede longitud -> rechazar.
- **A4:** Fallo de Cloudinary -> no crear una obra incompleta.
- **A5:** Tag inexistente -> crear si se admite administración dinámica o rechazar/normalizar según política definida.
- **A6:** Usuario baneado o sin permisos -> 403.

---

## CU04 — Explorar obras

**Actor principal:** Invitado o usuario  
**Precondición:** Ninguna.

### Flujo principal

1. El usuario abre el explorador.
2. El sistema devuelve obras visibles y no eliminadas.
3. El usuario aplica filtros.
4. El cliente solicita la página correspondiente.
5. El usuario selecciona una obra.
6. El sistema muestra detalle.

### Alternativo

- **A1:** No existen resultados -> mostrar estado vacío.
- **A2:** Filtro inválido -> devolver error de validación.
- **A3:** Obra fue eliminada entre la búsqueda y el detalle -> devolver recurso no disponible.

---

## CU05 — Calificar obra mediante Q2Q

**Actor principal:** Usuario autenticado  
**Precondición:** La obra está dentro de su semana de calificación.

### Flujo principal

1. El usuario abre el detalle de una obra.
2. El backend informa si la calificación sigue habilitada.
3. El usuario selecciona entre 1 y 5 estrellas.
4. Selecciona una emoción permitida.
5. Opcionalmente publica un comentario.
6. Se registra o actualiza la valoración.
7. Se confirma la acción.

### Alternativo

- **A1:** Período expirado -> no se acepta valoración; comentarios siguen disponibles.
- **A2:** Usuario no autenticado -> se requiere login.
- **A3:** Emoción inválida -> 400.
- **A4:** Estrellas fuera de rango -> 400.
- **A5:** Usuario intenta duplicar valoración -> actualizar existente o rechazar según la política elegida. Se recomienda actualización.
- **A6:** Obra eliminada/bloqueada -> no permitir valoración.

---

## CU06 — Conversión de obra a carta

**Actor técnico:** CRON / sistema  
**Precondición:** Obra elegible, periodo de una semana terminado y `conversion_request` habilitado.

### Flujo principal

1. El CRON se ejecuta periódicamente.
2. Consulta obras cuyo período de calificación expiró.
3. Filtra obras que solicitaron conversión.
4. Filtra obras cuyo `conversion_status` indica que todavía están pendientes.
5. Obtiene calificaciones, emociones y comentarios necesarios.
6. Ejecuta el algoritmo de conversión.
7. Determina ataque, defensa, HP, velocidad y rareza.
8. Crea la carta asociada.
9. Actualiza el estado de conversión.
10. Genera una notificación de fin de calificación/conversión cuando corresponda.

### Alternativo

- **A1:** Obra no elegible -> se ignora.
- **A2:** Ya existe carta -> marcar como convertida sin duplicar.
- **A3:** Error al calcular -> registrar error y mantener estado reintentable.
- **A4:** Error de DB durante transacción -> rollback.

### Observación

NestJS cuenta con un Scheduler, utilizar esto antes que el propio cron o scheduler.

---

## CU07 — Explorar colección/cartas

**Actor principal:** Invitado o usuario autenticado.

### Flujo principal

1. El usuario abre la sección de cartas.
2. Aplica filtros.
3. El sistema pagina los resultados.
4. El usuario selecciona una carta.
5. Se muestra información pública de la carta.
6. Si el usuario la posee, se muestran sus estadísticas completas.
7. Si no la posee, se ocultan las estadísticas según la regla del álbum.

### Alternativo

- **A1:** Carta no poseída -> estadística oculta.
- **A2:** Carta inexistente -> 404.
- **A3:** El usuario solicita más de una página -> devolver `nextCursor` o metadata equivalente.

---

## CU08 — Abrir paquete

**Actor principal:** Usuario autenticado  
**Precondición:** El usuario tiene al menos un paquete disponible.

### Flujo principal

1. El usuario entra a la sección de paquetes.
2. El cliente solicita abrir un paquete.
3. El backend verifica disponibilidad.
4. Se consume exactamente un paquete.
5. Se seleccionan cartas según las reglas del sistema.
6. Se genera un número al azar y con eso se decide la rareza.
7. Se da una carta de esa rareza al azar.
8. Se incrementan las copias en la colección.
9. Se devuelve la lista de cartas obtenidas.
10. El cliente presenta la animación y el resultado.

### Alternativo

- **A1:** No hay paquetes -> 409 o respuesta de estado equivalente.
- **A2:** Peticiones simultáneas -> la operación debe protegerse contra doble consumo.
- **A3:** Error transaccional -> ni el paquete ni las recompensas deben quedar parcialmente aplicados.

### Observaciones

La regeneración de paquetes debe calcularse a partir de `last_used` y el tiempo transcurrido.

---

## CU09 — Jugar autobattler

**Actor principal:** Usuario autenticado.

### Flujo principal

1. El usuario abre el módulo de juego.
2. Selecciona una carta que posee.
3. El cliente solicita una carta rival.
4. El backend selecciona una carta rival válida de forma aleatoria.
5. Devuelve los datos necesarios para la simulación.
6. El cliente ejecuta la batalla.
7. El cliente muestra turnos, daño, vida restante y resultado.
8. La sesión termina.

### Alternativo

- **A1:** Usuario no posee la carta seleccionada -> 403/400.
- **A2:** Carta no existe -> 404.
- **A3:** No existen cartas válidas para rival -> error controlado.

### Observaciones

No almacenar resultado, replay, victoria, derrota ni historial de batallas en backend en esta versión.

---

## CU10 — Seguir usuario

**Actor principal:** Usuario autenticado.

### Flujo principal

1. El usuario abre un perfil.
2. Pulsa “Seguir”.
3. Se crea la relación.
4. Se genera una notificación al usuario seguido si corresponde.

### Alternativo

- **A1:** Ya existe la relación -> operación idempotente.
- **A2:** Usuario intenta seguirse a sí mismo -> rechazar.
- **A3:** Usuario objetivo inexistente/baneado -> rechazar según política.

---

## CU11 — Destacar obra o carta

**Actor principal:** Usuario autenticado.

### Flujo principal

1. El usuario abre su perfil.
2. Selecciona una obra propia o una carta que posee.
3. Pulsa “Destacar”.
4. Se registra la relación de favorito/destacado.
5. El perfil muestra el elemento en su sección correspondiente.

### Alternativo

- **A1:** El usuario no es propietario de la obra -> rechazar.
- **A2:** El usuario no posee la carta -> rechazar.
- **A3:** Ya está destacada -> operación idempotente.

---

## CU12 — Denunciar obra

**Actor principal:** Usuario autenticado.

### Flujo principal

1. El usuario abre una obra.
2. Pulsa “Denunciar”.
3. Selecciona un motivo predefinido.
4. Opcionalmente agrega comentario.
5. Se registra el reporte.
6. Se confirma al usuario.

### Alternativo

- **A1:** Motivo inválido -> rechazar.
- **A2:** Obra inexistente -> 404.
- **A3:** Obra ya eliminada -> informar que no requiere una nueva acción o permitir reporte según política.
- **A4:** Usuario intenta realizar múltiples reportes idénticos -> aplicar regla anti-spam/duplicados.

---

## CU13 — Denunciar comentario

**Actor principal:** Usuario autenticado.

### Flujo principal

1. El usuario visualiza un comentario.
2. Pulsa “Denunciar”.
3. Selecciona el motivo.
4. Opcionalmente añade detalle.
5. Se crea el reporte asociado al comentario.
6. Se notifica el registro exitoso.

### Alternativo

- **A1:** Comentario eliminado -> impedir o resolver automáticamente según estado.
- **A2:** Motivo inválido -> 400.
- **A3:** Reporte duplicado -> rechazar o consolidar.

---

## CU14 — Administrar denuncias

**Actor principal:** Moderador.

### Flujo principal

1. El moderador abre el panel.
2. El sistema devuelve reportes pendientes, paginados y ordenados por prioridad/fecha.
3. El moderador abre un reporte.
4. Se muestra el contenido denunciado y contexto suficiente.
5. Selecciona una acción.
6. El sistema aplica la acción.
7. Marca el reporte como resuelto.
8. Registra auditoría.
9. Genera notificación cuando corresponda.

### Acciones

- descartar;
- advertir al autor;
- eliminar obra;
- eliminar comentario;
- banear usuario.

### Alternativo

- **A1:** El contenido ya fue eliminado -> permitir cerrar el reporte sin duplicar la acción.
- **A2:** Usuario ya baneado -> registrar el reporte como resuelto/afectado.
- **A3:** Moderador sin permisos -> 403.
- **A4:** Dos moderadores resuelven simultáneamente -> utilizar control de concurrencia/estado para evitar doble aplicación.

---

## CU15 — Notificaciones

**Actor principal:** Usuario autenticado.

### Flujo principal

1. Ocurre un evento notificable.
2. El backend crea una notificación.
3. El usuario consulta el panel.
4. Se devuelven notificaciones paginadas.
5. El usuario marca una o varias como leídas.

### Eventos iniciales

- usuario seguido;
- nuevo upload de usuario seguido;
- fin de período de calificación;
- comentario realizado sobre una obra;
- comentario eliminado/baneado por moderación;
- resultado de una denuncia;
- actividad de votos/calificaciones relevante.

---

## CU16 — Ver/editar perfil

### Flujo principal

1. El usuario abre un perfil.
2. El sistema devuelve información pública.
3. Si el perfil es propio, se habilita edición.
4. Se guardan los cambios válidos.

### Alternativo

- **A1:** Username elegido por otro usuario -> conflicto de unicidad.
- **A2:** Datos fuera de rango -> 400.
- **A3:** Usuario no autenticado intentando editar -> 401.

---

## CU17 — Gestionar rol de moderador

**Actor principal:** Admin  
**Precondición:** El actor posee el rol `admin`.  
**Postcondición:** El usuario objetivo gana o pierde el rol `moderator`.

### Flujo principal

1. El admin abre el panel de administración.
2. Selecciona un usuario.
3. Otorga o revoca el rol `moderator`.
4. El sistema actualiza los roles del usuario objetivo.
5. Se confirma la acción.

### Alternativo

- **A1:** El usuario ya tiene (u otorgar) o ya no tiene (revocar) el rol -> operación idempotente, no es un error.
- **A2:** Usuario objetivo inexistente -> 404.
- **A3:** Actor sin rol `admin` -> 403.
- **A4:** El rol `admin` en sí mismo no se otorga ni revoca por este medio -> fuera de alcance de este caso de uso.

---

## CU18 — Reabrir un reporte resuelto

**Actor principal:** Admin  
**Precondición:** Existe un reporte con `status = resolved` y una `resolution` punitiva (`content_removed`, `comment_removed` o `user_banned`).  
**Postcondición:** La sanción original queda revertida (contenido restaurado o usuario desbaneado) y el reporte pasa a `status = overturned`.

### Flujo principal

1. El admin abre el listado de reportes resueltos.
2. Selecciona un reporte con sanción aplicada.
3. Confirma la reapertura.
4. El sistema revierte el efecto de la sanción original.
5. El sistema marca el reporte como `overturned`.
6. Se notifica al usuario afectado (obra/comentario restaurado, o cuenta desbaneada).

### Alternativo

- **A1:** El reporte ya fue reabierto o ya tiene una apelación aprobada en paralelo -> 409 (nada que revertir dos veces).
- **A2:** La `resolution` del reporte no es punitiva (`dismissed`/`warning`) -> 400, no hay nada que revertir.
- **A3:** El reporte no existe o no está resuelto -> 404.
- **A4:** Actor sin rol `admin` (un moderador sin `admin` no puede reabrir) -> 403.

---

## CU19 — Apelar una sanción

**Actor principal:** Usuario autenticado afectado (autor de la obra/comentario eliminado, o usuario baneado apelando su propia cuenta)  
**Precondición:** Existe un reporte resuelto con sanción punitiva sobre su obra, comentario o cuenta, y esa sanción todavía no fue apelada.  
**Postcondición:** Se registra una apelación en estado `pending`, visible para el personal de moderación.

### Flujo principal

1. El usuario ve que su obra, comentario o cuenta fue sancionada.
2. Abre la opción de apelar.
3. Selecciona un motivo predefinido (sanción excesiva o incorrecta, identidad equivocada, falta de contexto, reporte falso, otro).
4. Opcionalmente agrega un texto explicando su caso.
5. Envía la apelación.
6. El sistema la registra en estado `pending`.

### Alternativo

- **A1:** Ya existe una apelación previa para esa misma sanción -> 409, no se admite una segunda.
- **A2:** No existe una sanción punitiva apelable sobre ese contenido/cuenta -> 404.
- **A3:** El usuario no es el autor del contenido ni el usuario baneado -> 403.
- **A4:** El usuario apelando su propio baneo está autenticado pero baneado -> se le permite igual: apelar el propio baneo, consultar sus apelaciones y sus notificaciones son las únicas operaciones de escritura/lectura que una cuenta baneada conserva.

---

## CU20 — Revisar una apelación

**Actor principal:** Moderador o Admin  
**Precondición:** Existe una apelación en estado `pending`; el revisor no resolvió el reporte original que la apelación referencia, salvo que sea el único miembro de moderación (`moderator` + `admin`) en el sistema.  
**Postcondición:** La apelación queda `approved` o `rejected`; si fue aprobada, la sanción original queda revertida igual que en CU18.

### Flujo principal

1. El moderador abre el listado de apelaciones pendientes (el sistema excluye las que él mismo originó al resolver el reporte, salvo la excepción de moderador único).
2. Abre el detalle de una apelación: motivo, texto opcional, y el reporte/sanción que referencia.
3. Decide aprobar o rechazar.
4. Si aprueba: el sistema revierte la sanción original (mismo mecanismo que CU18) y notifica al afectado.
5. Si rechaza: no se modifica el contenido ni el reporte.
6. En ambos casos se notifica el resultado al apelante.

### Alternativo

- **A1:** El revisor resolvió el reporte original y hay más de un miembro de personal de moderación -> 403, debe revisarla otro moderador/admin.
- **A2:** La apelación ya fue resuelta por otro moderador -> 409.
- **A3:** La aprobación compite con una reapertura directa (CU18) del mismo reporte -> no es un error para la apelación: se considera igualmente aprobada, ya que el resultado deseado ya ocurrió.
- **A4:** Actor sin rol `moderator`/`admin` -> 403.

---

## CU21 — Cargar carta directamente (Admin)

**Actor principal:** Admin  
**Precondición:** El actor posee el rol `admin`.  
**Postcondición:** Existe una nueva obra con `conversion_status = converted` y su carta asociada, sin haber pasado por el período de calificación.

### Flujo principal

1. El admin abre la herramienta de carga directa.
2. Completa título y, opcionalmente, descripción/tags, igual que al subir una obra normal.
3. Selecciona la imagen.
4. Indica la rareza de la carta resultante.
5. Opcionalmente indica estadísticas explícitas (ataque, defensa, vida, velocidad).
6. El backend sube la imagen, crea la obra ya `converted` y crea la carta en el mismo paso.
7. Las estadísticas no indicadas se completan según la rareza, con la misma fórmula que usa el sembrado del catálogo mínimo (sección 7.5, regla 7).
8. Se devuelve la carta creada.

### Alternativo

- **A1:** Archivo inválido o ausente -> mismo tratamiento que CU03 (A1/A2).
- **A2:** Título vacío o excede longitud -> rechazar.
- **A3:** Rareza inválida -> 400.
- **A4:** Estadística negativa o no entera -> 400.
- **A5:** Actor sin rol `admin` -> 403.

### Observaciones

No genera notificación a seguidores del admin: no es un upload orgánico. El admin no posee automáticamente la carta creada (ver 7.5.7).

---

## CU22 — Disparar el ciclo de conversión manualmente (Admin)

**Actor principal:** Admin  
**Precondición:** El actor posee el rol `admin`.  
**Postcondición:** Toda obra elegible cuyo período de calificación ya terminó queda procesada, igual que si hubiera corrido el CRON.

### Flujo principal

1. El admin dispara la ejecución manual (pensado para demos, sin esperar al horario del CRON).
2. El backend ejecuta exactamente la misma rutina que usa el scheduler (CU06): busca obras elegibles, las convierte y marca las no solicitadas como `skipped`.
3. Devuelve un resumen: cantidad procesada, convertida, fallida y marcada como `skipped`.

### Alternativo

- **A1:** No hay obras elegibles -> se devuelve el resumen en cero, no es un error.
- **A2:** Actor sin rol `admin` -> 403.

### Observaciones

No es un proceso alternativo al CRON: es el mismo `runConversionCycle` (CU06), sólo que disparado bajo demanda en lugar de por el reloj.

---

## CU23 — Recuperar contraseña olvidada

**Actor principal:** Invitado (usuario que no puede iniciar sesión)  
**Precondición:** Existe una cuenta con esa dirección de email; si la cuenta tiene contraseña local configurada, la solicitud desencadena el envío del enlace.  
**Postcondición:** El usuario cuenta con una nueva contraseña y puede iniciar sesión con ella; cualquier sesión abierta con la contraseña anterior quedó cerrada.

### Flujo principal

1. El usuario abre "¿Olvidaste tu contraseña?" desde la pantalla de login.
2. Introduce su email.
3. El backend responde con un mensaje neutro, sin revelar si la cuenta existe.
4. Si la cuenta existe y tiene contraseña local, se genera un token de un solo uso y se envía por email.
5. El usuario abre el enlace recibido.
6. Introduce una nueva contraseña.
7. El backend valida el token y la actualiza.
8. Se revocan todas las sesiones (`refresh_tokens`) activas de esa cuenta.
9. El usuario inicia sesión nuevamente con la contraseña nueva.

### Alternativo

- **A1:** El email no corresponde a ninguna cuenta -> mismo mensaje neutro que en el flujo exitoso (anti-enumeración), no se envía nada.
- **A2:** La cuenta existe pero es exclusivamente de Google (sin contraseña local) -> mismo mensaje neutro, no se envía nada: no hay contraseña que recuperar.
- **A3:** Token inválido, inexistente o ya utilizado -> 400.
- **A4:** Token expirado (más de 1 hora) -> 400, el usuario debe solicitar uno nuevo.
- **A5:** Nueva contraseña fuera de la política de longitud -> 400.

### Observaciones

No inicia sesión automáticamente al restablecer la contraseña (a diferencia de CU01, que sí lo hace al verificar el email) — es una operación más sensible y se prefiere que el usuario reingrese explícitamente con la contraseña nueva.

---

# 9. Modelo de datos

La definición inicial incluye usuarios, artworks, comentarios, ratings, cartas, inventario, paquetes, reportes, follows, favoritos y tags. Esta versión agrega las estructuras necesarias para soportar notificaciones, moderación y verificación de email. 

## 9.1 users

| Campo | Tipo | Reglas |
|---|---|---|
| id | bigint/int | PK |
| username | varchar(30) | NOT NULL, UNIQUE |
| email | varchar(50) | NOT NULL, UNIQUE |
| password_hash | varchar | NULL para cuentas OAuth |
| email_verified | boolean | NOT NULL, default false |
| status | enum | active, suspended, banned, pending_verification |
| created_at | timestamp | NOT NULL |
| updated_at | timestamp | nullable |

### Constraints

- `PRIMARY KEY (id)`
- `UNIQUE (username)`
- `UNIQUE (email)`
- `CHECK (char_length(username) <= 30)`
- `CHECK (char_length(email) <= 50)` según el límite funcional adoptado
- `password_hash` puede ser NULL sólo para proveedores de autenticación externa, si se modela de esa forma.
---

## 9.2 roles

Se propone separar usuarios y roles para evitar una única columna rígida y permitir crecimiento futuro.

### roles

| Campo | Tipo | Reglas |
|---|---|---|
| id | bigint | PK |
| name | enum/varchar | UNIQUE, NOT NULL |
| description | varchar | nullable |

### Enum `role_name`

```text
user
moderator
admin
```

### user_roles

| Campo | Tipo | Reglas |
|---|---|---|
| user_id | bigint | FK users.id, NOT NULL |
| role_id | bigint | FK roles.id, NOT NULL |

**PK compuesta:** `(user_id, role_id)`

Esto evita roles duplicados y permite que un usuario posea más de un rol.

---

## 9.3 artworks

| Campo                    | Tipo         | Reglas                  |
| ------------------------ | ------------ | ----------------------- |
| id                       | bigint       | PK                      |
| title                    | varchar(50)  | NOT NULL                |
| description              | varchar(300) | nullable                |
| author_id                | bigint       | FK users.id, NOT NULL   |
| image_original_url       | varchar      | NOT NULL                |
| image_medium_url         | varchar      | NOT NULL                |
| image_thumbnail_url      | varchar      | NOT NULL                |
| conversion_request       | boolean      | NOT NULL, default true  |
| conversion_status        | enum         | NOT NULL                |
| qualification_started_at | timestamp    | NOT NULL                |
| deleted                  | boolean      | NOT NULL, default false |
| created_at               | timestamp    | NOT NULL                |
| updated_at               | timestamp    | nullable                |

### Enum `conversion_status`

```text
pending
processing
converted
skipped
failed
```

Interpretación:

- `pending`: todavía no procesada;
- `processing`: tomada por un proceso de conversión;
- `converted`: ya generó su carta;
- `skipped`: no debe convertirse;
- `failed`: ocurrió un error y puede quedar disponible para reintento controlado.

### Decisiones

- `convert` del modelo original pasa a ser `conversion_request`.
- `converted` pasa a ser `conversion_status`.
- Se agregan fechas explícitas de inicio y fin de calificación.
- Se eliminan referencias conceptuales a likes.

### Constraints

- `title` obligatorio y máximo 50 caracteres.
- `description` máximo 300 caracteres.
- `author_id` debe referenciar un usuario existente.

---

## 9.4 artwork_comments

| Campo | Tipo | Reglas |
|---|---|---|
| id | bigint | PK |
| author_id | bigint | FK users.id, NOT NULL |
| artwork_id | bigint | FK artworks.id, NOT NULL |
| comment | varchar(300) | NOT NULL |
| deleted | boolean | NOT NULL, default false |
| created_at | timestamp | NOT NULL |
| updated_at | timestamp | nullable |

Las obras continúan siendo comentables después de finalizar el período de calificación.

### Constraints

- FK a `users`.
- FK a `artworks`.
- `comment` no puede estar vacío después de trim.
- Longitud máxima recomendada: 300.

---

## 9.5 artwork_ratings

Nombre recomendado para reemplazar el ambiguo `rating`.

| Campo | Tipo | Reglas |
|---|---|---|
| id | bigint | PK |
| artwork_id | bigint | FK artworks.id, NOT NULL |
| voter_id | bigint | FK users.id, NOT NULL |
| stars | smallint | NOT NULL |
| emotion | enum | NOT NULL |
| created_at | timestamp | NOT NULL |
| updated_at | timestamp | nullable |

### Enum `emotion`


```text
joy
fear
anger
serenity
grief
awe
```

Las emociones pueden extenderse aunque esto es un evento extraordinario.

### Constraints

- `CHECK (stars BETWEEN 1 AND 5)`.
- `UNIQUE (artwork_id, voter_id)`.
- No se admite valoración sobre obra eliminada.

La última regla es de dominio y debe validarse en servicio, no sólo mediante `CHECK` SQL.

---

## 9.6 cards

| Campo | Tipo | Reglas |
|---|---|---|
| id | bigint | PK |
| artwork_id | bigint | FK artworks.id, UNIQUE |
| attack | integer | NOT NULL, >= 0 |
| defense | integer | NOT NULL, >= 0 |
| hp | integer | NOT NULL, >= 0 |
| speed | integer | NOT NULL, >= 0 |
| rarity | enum | NOT NULL |
| created_at | timestamp | NOT NULL |
| updated_at | timestamp | nullable |

### Enum `rarity`

```text
common
uncommon
rare
epic
legendary
```

### Constraints

- `UNIQUE (artwork_id)` garantiza una carta por obra.
- Stats no negativos.
- `rarity` limitada al enum.

---

## 9.7 user_cards

| Campo | Tipo | Reglas |
|---|---|---|
| id | bigint | PK |
| user_id | bigint | FK users.id |
| card_id | bigint | FK cards.id |
| copies | integer | NOT NULL, default 0 |

### Constraints

- `UNIQUE (user_id, card_id)`.
- `CHECK (copies >= 0)`.
- Foreign keys a usuarios y cartas.

El contador se incrementa al recibir repetidas.

---

## 9.8 card_packs

| Campo | Tipo | Reglas |
|---|---|---|
| id | bigint | PK |
| user_id | bigint | FK users.id, UNIQUE |
| amount | integer | NOT NULL, default 0 |
| last_regeneration_at | timestamp | nullable |
| updated_at | timestamp | nullable |

### Constraints

- `UNIQUE (user_id)` para representar un inventario de paquetes por usuario.
- `CHECK (amount BETWEEN 0 AND 10)`.

### Regeneración

El sistema puede determinar el incremento de paquetes mediante:

```text
elapsed_minutes = floor(now - last_regeneration_at)
regenerated = min(elapsed_minutes, 10 - amount)
```

Se recomienda realizar el cálculo cuando el usuario consulte o modifique sus paquetes, en lugar de almacenar millones de eventos de regeneración.

---

## 9.9 tags

| Campo | Tipo | Reglas |
|---|---|---|
| id | bigint | PK |
| name | varchar(30) | NOT NULL, UNIQUE |

### Constraints

- `UNIQUE(name)`.
- Normalizar a minúsculas o aplicar política uniforme de casing.

---

## 9.10 artwork_tags

| Campo | Tipo | Reglas |
|---|---|---|
| artwork_id | bigint | FK artworks.id |
| tag_id | bigint | FK tags.id |

**PK compuesta:** `(artwork_id, tag_id)`.

Esto reemplaza el `id` artificial si no existen propiedades propias de la relación.

---

## 9.11 follows

| Campo | Tipo | Reglas |
|---|---|---|
| follower_id | bigint | FK users.id |
| followed_id | bigint | FK users.id |
| created_at | timestamp | NOT NULL |

**PK compuesta:** `(follower_id, followed_id)`.

### Constraints

- Prohibir seguimiento de uno mismo: `follower_id <> followed_id`.
- Una relación no puede duplicarse.

---

## 9.12 favorite_cards

| Campo | Tipo | Reglas |
|---|---|---|
| user_id | bigint | FK users.id |
| card_id | bigint | FK cards.id |
| created_at | timestamp | NOT NULL |

**PK compuesta:** `(user_id, card_id)`.

---

## 9.13 favorite_artworks

| Campo | Tipo | Reglas |
|---|---|---|
| user_id | bigint | FK users.id |
| artwork_id | bigint | FK artworks.id |
| created_at | timestamp | NOT NULL |

**PK compuesta:** `(user_id, artwork_id)`.

Se utilizan para el mecanismo de elementos destacados. Se elimina cualquier referencia al antiguo sistema de likes generales sobre artworks.

---

## 9.14 reports

El sistema de reportes debe soportar tanto obras como comentarios.

#### artwork_reports

| Campo | Tipo | Reglas |
|---|---|---|
| id | bigint | PK |
| artwork_id | bigint | FK artworks.id, NOT NULL |
| reporter_id | bigint | FK users.id, NOT NULL |
| reason | enum | NOT NULL |
| comment | varchar(300) | nullable |
| status | enum | NOT NULL |
| resolved_by | bigint | FK users.id, nullable |
| resolution | enum | nullable |
| created_at | timestamp | NOT NULL |
| resolved_at | timestamp | nullable |

#### comment_reports

| Campo | Tipo | Reglas |
|---|---|---|
| id | bigint | PK |
| comment_id | bigint | FK artwork_comments.id, NOT NULL |
| reporter_id | bigint | FK users.id, NOT NULL |
| reason | enum | NOT NULL |
| comment | varchar(300) | nullable |
| status | enum | NOT NULL |
| resolved_by | bigint | FK users.id, nullable |
| resolution | enum | nullable |
| created_at | timestamp | NOT NULL |
| resolved_at | timestamp | nullable |

### Enum `report_reason`

```text
content_inappropriate
content_violent
content_sexual
illicit
copyright
plagiarism
spam
harassment
other
```

### Enum `report_status`

```text
pending
in_review
resolved
rejected
overturned
```

`overturned` se alcanza únicamente desde `resolved` (reapertura por admin o apelación aprobada), y sólo cuando la `resolution` original fue punitiva. `rejected` está reservado para un futuro mecanismo de auto-rechazo (p. ej. deduplicación) que ningún flujo actual produce todavía.

### Enum `report_resolution`

```text
dismissed
warning
content_removed
comment_removed
user_banned
```

### Constraints y reglas

- `reporter_id` debe ser usuario autenticado.
- `resolved_by` debe corresponder a moderador/admin.
- Un reporte resuelto debe tener `resolved_at` y `resolved_by`.
- Evitar reportes duplicados idénticos por el mismo usuario y objetivo cuando exista uno pendiente.
- Toda acción de moderación debe quedar auditada.

---

## 9.15 notifications

### Enum `notification_type`

```text
user_follow
follow_upload
qualification_ended
artwork_converted
artwork_comment
artwork_vote_summary
comment_removed
artwork_removed
user_warned
user_banned
artwork_restored
comment_restored
user_unbanned
appeal_resolved
```

El conjunto no es exhaustivo y puede ampliarse.

### Tabla

| Campo | Tipo | Reglas |
|---|---|---|
| id | bigint | PK |
| user_id | bigint | FK users.id, NOT NULL |
| type | enum | NOT NULL |
| title | varchar(100) | NOT NULL |
| content | varchar(500) | NOT NULL |
| is_read | boolean | NOT NULL, default false |
| created_at | timestamp | NOT NULL |

### Constraints

- FK de `user_id` a `users`.
- `title` y `content` no pueden estar vacíos.
- Índice sobre `(user_id, is_read, created_at)` para consultas eficientes.

---

## 9.16 email_verification_tokens

Entidad recomendada para hacer explícito el proceso de verificación.

| Campo | Tipo | Reglas |
|---|---|---|
| id | bigint | PK |
| user_id | bigint | FK users.id |
| token_hash | varchar | NOT NULL, UNIQUE |
| expires_at | timestamp | NOT NULL |
| used_at | timestamp | nullable |
| created_at | timestamp | NOT NULL |

### Reglas

- No almacenar el token de verificación en texto plano si puede evitarse.
- Expiración obligatoria.
- Un token usado no puede reutilizarse.

---

## 9.17 Artifacts de autenticación OAuth

Para Google OAuth puede incorporarse una tabla separada si se requiere soportar múltiples proveedores.

### oauth_accounts

| Campo | Tipo | Reglas |
|---|---|---|
| id | bigint | PK |
| user_id | bigint | FK users.id |
| provider | varchar | NOT NULL |
| provider_user_id | varchar | NOT NULL |

**Constraint recomendado:** `UNIQUE(provider, provider_user_id)`.

---

## 9.18 appeals

Tabla única, no dividida por tipo de contenido: toda apelación remite a un único reporte ya resuelto (`report_target_type` + `report_id`, mismo discriminador de `report_target_type` usado en 9.14), y ese reporte ya indica mediante su `resolution` si la sanción fue sobre una obra, un comentario o un usuario baneado.

| Campo | Tipo | Reglas |
|---|---|---|
| id | bigint | PK |
| report_target_type | enum | NOT NULL — mismo discriminador que reports (`artwork`, `comment`) |
| report_id | bigint | NOT NULL — referencia lógica a `artwork_reports.id` o `comment_reports.id` según `report_target_type` |
| appellant_id | bigint | FK users.id, NOT NULL |
| reason | enum | NOT NULL |
| detail | varchar(500) | nullable |
| status | enum | NOT NULL, default `pending` |
| reviewed_by | bigint | FK users.id, nullable |
| reviewed_at | timestamp | nullable |
| created_at | timestamp | NOT NULL |

### Enum `appeal_reason`

```text
excessive_punishment
mistaken_identity
missing_context
false_report
other
```

### Enum `appeal_status`

```text
pending
approved
rejected
```

### Constraints y reglas

- `UNIQUE(report_target_type, report_id)`: un reporte sólo puede apelarse una vez, sin importar el resultado.
- Un baneo de usuario se representa igual que una apelación de obra/comentario: `report_target_type`/`report_id` apuntan al reporte cuya `resolution` fue `user_banned`; no existe un tercer valor de `report_target_type` para "usuario".
- `reviewed_by` y `reviewed_at` sólo se completan al resolver (`approved`/`rejected`).
- El usuario que revisa (`reviewed_by`) no puede ser el mismo que aparece en `resolved_by` del reporte referenciado, salvo que en ese momento exista exactamente un miembro de personal de moderación (`moderator` + `admin` combinados).

---

## 9.19 password_reset_tokens

Misma estructura que `email_verification_tokens` (9.16): un token opaco de un solo uso, hasheado antes de persistir, con expiración explícita. Tabla separada en lugar de reutilizar `email_verification_tokens` para no mezclar dos propósitos distintos (activar una cuenta vs. recuperar acceso a una) bajo el mismo modelo.

| Campo | Tipo | Reglas |
|---|---|---|
| id | bigint | PK |
| user_id | bigint | FK users.id |
| token_hash | varchar | NOT NULL, UNIQUE |
| expires_at | timestamp | NOT NULL |
| used_at | timestamp | nullable |
| created_at | timestamp | NOT NULL |

### Reglas

- No almacenar el token de recuperación en texto plano (mismo hash que el resto de los tokens opacos del sistema).
- Expiración obligatoria — **1 hora**, más corta que las 24 horas de `email_verification_tokens` por ser un token más sensible.
- Un token usado no puede reutilizarse.
- Al emitir un token nuevo para un usuario, cualquier token previo sin usar de ese usuario se invalida (mismo mecanismo que el reenvío de verificación de email).

---

# 10. Relaciones principales

```text
users 1 ─── N artworks
users 1 ─── N artwork_comments
users 1 ─── N artwork_ratings
artworks 1 ─── N artwork_comments
artworks 1 ─── N artwork_ratings
artworks 1 ─── 1 cards
users N ─── N cards        (user_cards)
users 1 ─── 1 card_packs
users N ─── N users        (follows)
users N ─── N artworks     (favorite_artworks)
users N ─── N cards        (favorite_cards)
artworks N ─── N tags      (artwork_tags)
users 1 ─── N notifications
artworks 1 ─── N artwork_reports
artwork_comments 1 ─── N comment_reports
users 1 ─── N reports
users 1 ─── N email_verification_tokens
users 1 ─── N password_reset_tokens
users 1 ─── N oauth_accounts
users N ─── N roles        (user_roles)
users 1 ─── N appeals      (appellant_id)
users 1 ─── N appeals      (reviewed_by, nullable)
artwork_reports  1 ─── 0..1 appeals  (report_target_type = artwork)
comment_reports  1 ─── 0..1 appeals  (report_target_type = comment)
```

---

# 11. Endpoints REST propuestos

Los endpoints siguientes son una propuesta derivada de los requerimientos y casos de uso. Los nombres pueden adaptarse, pero se recomienda mantener una convención uniforme.

## 11.1 Auth

| Método | Endpoint | Uso | Auth |
|---|---|---|---|
| POST | `/auth/register` | Registrar usuario local | Público |
| POST | `/auth/login` | Login local | Público + rate limit |
| GET | `/auth/google` | Iniciar OAuth Google | Público |
| GET | `/auth/google/callback` | Callback Google | Público |
| POST | `/auth/verify-email` | Confirmar email | Público |
| POST | `/auth/resend-verification` | Reenviar challenge | Público + rate limit |
| POST | `/auth/forgot-password` | Solicitar enlace de recuperación de contraseña | Público + rate limit |
| POST | `/auth/reset-password` | Establecer una nueva contraseña con el enlace recibido | Público |
| POST | `/auth/refresh` | Renovar sesión, si se utiliza refresh token | Auth/refresh |
| POST | `/auth/logout` | Cerrar sesión | Auth |
| GET | `/auth/me` | Consultar usuario autenticado | Auth |

## 11.2 Users / Profiles

| Método | Endpoint | Uso |
|---|---|---|
| GET | `/users/:id` | Perfil público |
| PATCH | `/users/me` | Editar perfil propio |
| GET | `/users/me/followers` | Seguidores |
| GET | `/users/me/following` | Seguidos |
| POST | `/users/:id/follow` | Seguir |
| DELETE | `/users/:id/follow` | Dejar de seguir |

## 11.3 Artworks

| Método | Endpoint | Uso |
|---|---|---|
| POST | `/artworks` | Crear obra |
| GET | `/artworks` | Explorar y filtrar |
| GET | `/artworks/:id` | Detalle |
| PATCH | `/artworks/:id` | Editar obra propia |
| DELETE | `/artworks/:id` | Eliminación lógica |
| GET | `/users/me/artworks` | Obras propias |
| GET | `/artworks/:id/comments` | Comentarios |
| POST | `/artworks/:id/comments` | Crear comentario |

### Query params sugeridos para `GET /artworks`

```text
page / cursor
limit
search
from
until
authorId
tag
sort
qualificationOnly
converted
rarity
```

Para búsquedas que crezcan en complejidad, conviene encapsular parámetros en un DTO específico de filtros en lugar de construir SQL directamente desde query strings.

## 11.4 Ratings

| Método | Endpoint | Uso |
|---|---|---|
| PUT | `/artworks/:id/rating` | Crear o actualizar rating del usuario |
| DELETE | `/artworks/:id/rating` | Retirar rating, si la regla lo permite |
| GET | `/artworks/:id/ratings` | Resumen o datos de calificación |

No se debe exponer información que permita manipular el proceso de generación de estadísticas si ésta no corresponde a la interfaz pública.

## 11.5 Cards

| Método | Endpoint | Uso |
|---|---|---|
| GET | `/cards` | Explorar cartas |
| GET | `/cards/:id` | Detalle público |
| GET | `/users/me/cards` | Colección propia |
| GET | `/users/me/cards/:id` | Estado de posesión/detalle |
| POST | `/cards/:id/favorite` | Destacar carta |
| DELETE | `/cards/:id/favorite` | Quitar destacado |

## 11.6 Packages

| Método | Endpoint | Uso |
|---|---|---|
| GET | `/users/me/packages` | Consultar cantidad/estado |
| POST | `/users/me/packages/open` | Abrir paquete |

La operación de apertura debe ser atómica.

## 11.7 Battle

| Método | Endpoint | Uso |
|---|---|---|
| GET | `/battle/opponent` | Obtener una carta rival aleatoria |

No se requieren endpoints para guardar resultados.

No se recomienda implementar `/battle/start`, `/battle/result` o `/battle/history` mientras la batalla siga siendo puramente cliente-side.

## 11.8 Favorites

| Método | Endpoint | Uso |
|---|---|---|
| POST | `/artworks/:id/favorite` | Destacar obra |
| DELETE | `/artworks/:id/favorite` | Quitar destacado |
| GET | `/users/me/favorites/artworks` | Obras destacadas |
| GET | `/users/me/favorites/cards` | Cartas destacadas |

## 11.9 Reports

| Método | Endpoint | Uso |
|---|---|---|
| POST | `/artworks/:id/reports` | Reportar obra |
| POST | `/comments/:id/reports` | Reportar comentario |
| GET | `/moderation/reports` | Listar reportes |
| GET | `/moderation/reports/:id` | Detalle |
| POST | `/moderation/reports/:id/resolve` | Resolver reporte |

## 11.10 Notifications

| Método | Endpoint | Uso |
|---|---|---|
| GET | `/notifications` | Listar notificaciones |
| GET | `/notifications/unread-count` | Contador no leídas |
| PATCH | `/notifications/:id/read` | Marcar una como leída |
| POST | `/notifications/read-all` | Marcar todas como leídas |

## 11.11 Tags

| Método | Endpoint | Uso |
|---|---|---|
| GET | `/tags` | Consultar catálogo |

La administración de tags puede restringirse a administrador si dejan de ser libres.

## 11.12 Administración y Apelaciones

| Método | Endpoint | Uso | Auth |
|---|---|---|---|
| POST | `/roles/moderators/:userId` | Otorgar rol `moderator` | Admin |
| DELETE | `/roles/moderators/:userId` | Revocar rol `moderator` | Admin |
| POST | `/moderation/reports/:id/reopen` | Reabrir un reporte resuelto (reversión directa) | Admin |
| POST | `/artworks/:id/appeals` | Apelar la eliminación de una obra propia | Auth |
| POST | `/comments/:id/appeals` | Apelar la eliminación de un comentario propio | Auth |
| POST | `/users/me/ban-appeal` | Apelar el propio baneo | Auth (permitido para cuenta baneada) |
| GET | `/users/me/appeals` | Consultar apelaciones propias | Auth (permitido para cuenta baneada) |
| GET | `/moderation/appeals` | Listar apelaciones pendientes (excluye las que el revisor no puede resolver) | Moderator/Admin |
| GET | `/moderation/appeals/:id` | Detalle de una apelación | Moderator/Admin |
| POST | `/moderation/appeals/:id/resolve` | Aprobar o rechazar una apelación | Moderator/Admin |
| POST | `/artworks/admin-upload` | Cargar directamente obra + carta ya convertida (CU21) | Admin |
| POST | `/conversion/run` | Disparar manualmente el ciclo de conversión (CU22), pensado para demos | Admin |

`POST /moderation/reports/:id/reopen` es el único endpoint de `/moderation/*` restringido a `admin` exclusivamente; el resto de `/moderation/*` acepta `moderator` o `admin` por igual. `POST /artworks/admin-upload` y `POST /conversion/run` también son exclusivos de `admin`.

---

# 12. Guards y autorización

La arquitectura NestJS debería separar autenticación de autorización.

## 12.1 `JwtAuthGuard`

Protege rutas que requieren sesión autenticada.

Ejemplo conceptual de rutas protegidas:

```text
POST /artworks
PUT /artworks/:id/rating
POST /artworks/:id/comments
POST /artworks/:id/reports
POST /users/:id/follow
POST /users/me/packages/open
GET  /notifications
```

## 12.2 `RolesGuard`

Comprueba que el usuario tenga un rol requerido.

Ejemplo:

```text
GET  /moderation/reports
POST /moderation/reports/:id/resolve
GET  /moderation/appeals
POST /moderation/appeals/:id/resolve
```

requieren `moderator` o `admin`. En cambio:

```text
POST   /roles/moderators/:userId
DELETE /roles/moderators/:userId
POST   /moderation/reports/:id/reopen
POST   /artworks/admin-upload
POST   /conversion/run
```

requieren exclusivamente `admin`.

## 12.3 `OwnershipGuard`

No todo se resuelve con roles. El backend debe verificar propiedad del recurso.

Ejemplos:

- editar una obra -> el usuario debe ser su autor;
- eliminar una obra propia -> autor o moderador;
- destacar una obra -> debe ser propia;
- iniciar batalla con carta -> debe poseerla.

## 12.4 `VerifiedEmailGuard`

Puede utilizarse en funcionalidades que requieran cuenta plenamente habilitada.

Ejemplo:

```text
POST /artworks
POST /artworks/:id/comments
PUT  /artworks/:id/rating
POST /artworks/:id/reports
```

## 12.5 Rate limiting

Implementado con `@nestjs/throttler` (Fase 9/Hardening), por IP, en memoria del proceso:

- Límite global por defecto (100/min) como red de seguridad general, vía guard global.
- Override estricto (5/min, `@Throttle` por endpoint) en: login, registro, reenvío de verificación, recuperación de contraseña (`forgot-password`/`reset-password`) y `refresh`.
- Override moderado (10/min) en endpoints de escritura sensibles a spam: reportes, comentarios, valoraciones.

Detalle completo (valores exactos, comportamiento del error 429) en `docs/shared/api-conventions.md` §4.1.

## 12.6 `BannedUserGuard`

Un usuario baneado no debería poder utilizar funcionalidades de escritura aunque posea un JWT válido — pero sí debe poder **autenticarse**, porque necesita un token para apelar su propio baneo. Por eso el chequeo de estado `banned` no vive en `JwtStrategy`/`AuthService` (eso impediría obtener un token en absoluto), sino en este guard, aplicado junto a `JwtAuthGuard` en prácticamente todos los endpoints protegidos, **excepto**:

- `POST /users/me/ban-appeal` y `GET /users/me/appeals` (para poder apelar y consultar el resultado);
- todo `/notifications/*` (para poder leer la notificación que confirma el desenlace de la apelación).

El estado `suspended` es distinto: sigue bloqueado directamente en `JwtStrategy`/`AuthService`, sin ninguna excepción — la suspensión no forma parte del sistema de apelaciones.

---

# 13. Validaciones

## 13.1 Validación general

Toda entrada debe:

1. validarse en el backend aunque ya se valide en frontend;
2. normalizarse cuando corresponda;
3. limitar longitud y tamaño;
4. rechazar tipos inesperados;
5. sanitizar contenido apropiadamente — **implementado**: todo campo de texto libre (título/descripción de obra, tags, comentarios, bio, `comment`/`detail` de reportes y apelaciones) remueve marcado HTML antes de persistir, defensa contra XSS almacenado independiente del cliente que lo renderice (detalle en `docs/shared/api-conventions.md` §4.2);
6. evitar concatenación directa en consultas SQL — ya garantizado en todo el proyecto vía TypeORM (queries parametrizadas / query builder, sin interpolación directa de strings).

## 13.2 Usuario

```text
username: 1–30 caracteres, formato permitido, único
email: formato válido, máximo 50 según RNF adoptado, único
password: política de complejidad definida por implementación
```

La contraseña debe almacenarse siempre como hash.

## 13.3 Obras

```text
title: requerido, <= 50
email: no aplica
description: opcional, <= 300
tags: lista acotada y normalizada
image: <= 25 MB, PNG/JPG/JPEG/WEBP
```

## 13.4 Comentarios

```text
comment: requerido después de trim, <= 300
```

Deben aplicarse medidas anti-spam y rate limiting.

## 13.5 Ratings

```text
stars: integer 1..5
emotion: valor perteneciente al enum
```

Además debe verificarse que la obra esté dentro de su período activo.

## 13.6 Reportes

```text
reason: enum
comment: <= 300
```

No aceptar resolución enviada desde un cliente normal. Las resoluciones son acciones administrativas.

---

# 14. Manejo del almacenamiento de imágenes

Se utilizará **Cloudinary** para las imágenes de obras.

## Flujo propuesto

```text
Cliente
  |
  v
Backend valida metadata
  |
  v
Subida a Cloudinary
  |
  +--> original
  +--> thumbnail
  +--> medium
  |
  v
Backend guarda URLs / public_id
  |
  v
Artwork disponible
```

## Consideraciones

- No guardar blobs de imagen directamente en PostgreSQL.
- Guardar identificadores/URLs de Cloudinary.
- Utilizar transformaciones para thumbnails.
- Mantener acceso público sólo a los recursos que correspondan.
- Definir política de borrado de Cloudinary al eliminar una obra.
- No confiar sólo en la extensión del archivo: verificar MIME/type y contenido.

---

# 15. Arquitectura propuesta

## 15.1 Vista general

```text
+----------------------+       +----------------------+
|     Portal Web       |       |    Android App       |
| React + TypeScript   |       | Java                 |
+----------+-----------+       +-----------+----------+
           |                               |
           +---------------+---------------+
                           |
                           v
                  +-------------------+
                  |    REST API       |
                  | NestJS + TypeScript|
                  +---------+---------+
                            |
             +--------------+--------------+
             |                             |
             v                             v
      +-------------+               +-------------+
      | PostgreSQL  |               |  Cloudinary |
      +-------------+               +-------------+

                    +-------------------+
                    | Scheduler / CRON  |
                    | Conversion Worker |
                    +-------------------+
```

## 15.2 Backend por módulos

Una estructura razonable para NestJS:

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

### Capas recomendadas

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

El frontend no debe contener reglas críticas de negocio que también deban respetarse en móvil. Las reglas relevantes deben existir en backend.

## 15.3 Observabilidad / logging (RNF14)

- **Log de acceso HTTP** (`src/common/middleware/logging.middleware.ts`): un log por request (`método ruta status duración - ip`), a nivel `log` para 2xx/3xx y `warn` para 4xx/5xx. Deliberadamente implementado como **middleware de Express** (`app.use(...)` en `main.ts`), no como interceptor de Nest: los guards (`JwtAuthGuard`, `RolesGuard`, `AppThrottlerGuard`, etc.) corren *antes* que los interceptors en el pipeline de Nest, así que un interceptor nunca vería una request rechazada por un guard (401/403/429) — justo el tráfico más interesante de registrar para seguridad. El middleware corre primero de todos modos y se engancha a `res.on('finish')`, que se dispara sin importar en qué capa terminó resolviéndose la respuesta. Verificado en vivo: una request no autenticada (401) y una bloqueada por rate limiting (429) quedan registradas igual que una exitosa.
- **Errores no manejados**: `AllExceptionsFilter` ya logueaba (antes de esta fase) cualquier excepción no capturada, con stack trace.
- **Ejecuciones del CRON**: `ConversionSchedulerService` ya logueaba (antes de esta fase) un resumen por ciclo (procesadas/convertidas/fallidas/skipped).
- **Fallas de integraciones externas**: `MailService` (Resend) y `StorageService` (Cloudinary) ya logueaban (antes de esta fase) sus fallos sin abortar el flujo que los llamó (best-effort).

---

# 16. Conversión de obras a cartas

## 16.1 Objetivo

Evitar conversiones manuales y garantizar que una obra sólo sea convertida después de finalizar su semana de calificación.

## 16.2 Flujo

```text
CRON/Scheduler
  |
  v
Buscar artworks:
  qualification_started_at >= 7 dias
  conversion_request = true
  conversion_status = pending/failed reintentable
  |
  v
Tomar trabajo
  |
  v
Marcar processing
  |
  v
Obtener ratings + emociones + comentarios
  |
  v
Ejecutar CardConversionService
  |
  +--> attack
  +--> defense
  +--> hp
  +--> speed
  +--> rarity
  |
  v
Crear Card
  |
  v
Marcar converted
  |
  v
Notificación
```

## 16.3 Estados

```text
(creación) -> pending    [conversionRequest = true, u omitido]
(creación) -> skipped    [conversionRequest = false desde la creación: no hay período que dar]
pending -> processing -> converted
                  \-> failed
pending -> skipped        [conversionRequest pasó a false por una edición, antes del corte semanal]
```

## 16.4 Idempotencia

Debe existir una protección de base de datos mediante `UNIQUE(artwork_id)` en `cards`. El servicio debe utilizar transacciones para impedir que dos instancias del scheduler creen la misma carta.

## 16.5 Frecuencia del Scheduler

El scheduler debe dispararse a las 00:00, diariamente. Como se menciono antes, se debería utilizar el scheduler de Nest.

## 16.6 Disparo manual (Admin, demo)

La rutina de conversión (16.2–16.4) se expone también como método invocable independientemente del disparador de Cron, y un endpoint de administración (`POST /conversion/run`, CU22) la ejecuta bajo demanda. No es una segunda implementación: es la misma lógica, sólo que disparada por una request en lugar de por el reloj — útil para demos donde no tiene sentido esperar a que termine una semana de calificación real.

---

# 17. Lógica de regeneración de paquetes

Se requiere un máximo de 10 paquetes y una regeneración de 1 paquete por minuto.

## Estado mínimo

```text
amount
last_regeneration_at
```

## Cálculo

Al consultar o abrir paquetes:

1. calcular minutos completos transcurridos;
2. determinar cuántos paquetes deben regenerarse;
3. limitar el resultado a 10;
4. actualizar `last_regeneration_at` de forma consistente;
5. si se llega a 10, evitar acumular tiempo pendiente de manera que al bajar a 9 aparezcan paquetes de golpe no previstos por el diseño.

La apertura debe ejecutarse con protección de concurrencia para que dos requests no consuman un único paquete dos veces.

---

# 18. Sistema de notificaciones

## Eventos iniciales

| Evento | Receptor |
|---|---|
| Usuario comenzó a seguirlo | Usuario seguido |
| Usuario seguido publicó obra | Seguidores |
| Terminó calificación de una obra | Autor |
| Obra fue convertida | Autor |
| Nuevo comentario en obra | Autor / participantes según política |
| Comentario eliminado | Autor del comentario |
| Obra eliminada | Autor |
| Advertencia | Usuario afectado |
| Baneo | Usuario afectado |

La notificación debe ser un registro persistente. La entrega en tiempo real puede añadirse después mediante WebSocket/push sin cambiar necesariamente el modelo principal.

---

# 19. Moderación

## Flujo general

```text
Usuario denuncia
      |
      v
Reporte pendiente
      |
      v
Moderador revisa
      |
      +---- descartar
      +---- advertir
      +---- eliminar contenido
      +---- banear usuario
      |
      v
Reporte resuelto
      |
      v
Auditoría + notificación
      |
      v
¿Sanción punitiva? (content_removed / comment_removed / user_banned)
      |
      +---- No (dismissed/warning) -> no hay nada que reabrir ni apelar
      |
      v
Sí:
      +---- Admin reabre directamente ─────────────┐
      |                                             v
      +---- Afectado apela ──► Apelación pending    │
                    │                                │
                    v                                │
          Otro moderador/admin revisa                │
          (no quien resolvió, salvo único)            │
                    │                                │
          +---------+---------+                       │
          |                   |                       │
        rechaza            aprueba                     │
          |                   |                       │
          v                   v                       v
   No se modifica     Revertir sanción  <──────────────┘
   nada, se notifica  (restaurar obra/comentario
   al apelante         o desbanear) + reporte -> overturned
                        + notificar al afectado
                        + notificar al apelante
```

---

# 20. Índices recomendados

Para PostgreSQL se recomienda considerar, como mínimo:

```text
users(email)
users(username)
artworks(author_id, created_at)
artworks(deleted, created_at)
artwork_comments(artwork_id, created_at)
artwork_ratings(artwork_id)
user_cards(user_id, card_id)
card_packs(user_id)
follows(follower_id)
follows(followed_id)
notifications(user_id, is_read, created_at)
artwork_reports(status, created_at)
comment_reports(status, created_at)
appeals(status, created_at)
appeals(report_target_type, report_id)  -- ya cubierto por el UNIQUE
```

Los índices exactos deben revisarse mediante consultas reales y `EXPLAIN ANALYZE` cuando el volumen de datos sea representativo.

---

# 21. Convenciones de API

## Respuestas de éxito

Usar códigos HTTP coherentes y estructuras uniformes.

Ejemplo:

```json
{
  "data": {...},
  "meta": {...}
}
```

## Errores

```json
{
  "statusCode": 400,
  "code": "INVALID_ARTWORK",
  "message": "La obra no cumple los requisitos de validación."
}
```

No devolver stack traces ni detalles internos al cliente.

## Paginación

Preferir cursor pagination para feeds potencialmente grandes y page/limit para paneles administrativos simples cuando resulte suficiente.

---

# 22. DTOs y validación en NestJS

Los DTOs deberían representar contratos explícitos de entrada.

Ejemplos conceptuales:

```text
RegisterUserDto
LoginDto
CreateArtworkDto
UpdateArtworkDto
ArtworkFiltersDto
CreateCommentDto
RateArtworkDto
CreateReportDto
ResolveReportDto
UpdateProfileDto
CreateAppealDto
ResolveAppealDto
ReopenReportDto
QueryAppealsDto
```

No reutilizar automáticamente entidades TypeORM como DTOs de entrada.

Esto ayuda a:

- controlar qué campos puede modificar el cliente;
- aplicar validaciones;
- evitar mass assignment;
- separar modelo de persistencia y contrato de API.

---

# 23. Estado de una carta en el cliente

La API de colección debe permitir distinguir como mínimo:

```text
owned = true/false
copies = integer
statsVisible = true/false
```

Esto permite que el frontend renderice:

- carta completa si es poseída;
- imagen/obra visible con estadísticas desconocidas si no es poseída.

La ocultación es una regla de presentación, pero la API no debería enviar información privada de forma innecesaria cuando el usuario no posee la carta.

---

# 24. Lógica de selección del rival

El endpoint de rival debe cumplir las restricciones de negocio sin asumir que se trata de una partida multijugador.

Conceptualmente:

```text
GET /battle/opponent
        |
        v
Seleccionar carta válida aleatoria
        |
        v
Devolver datos necesarios
```

El endpoint no debe crear una “partida” persistente.

---

# 25. Pruebas recomendadas

## Backend unit tests

- validación de usuario;
- hash de contraseña;
- expiración de calificación;
- una valoración por usuario/obra;
- conversión de artwork;
- idempotencia del CRON;
- cálculo de paquetes;
- apertura atómica de paquete;
- permisos de ownership;
- permisos de moderación.

## Integration tests

- registro + verificación;
- login;
- creación de obra;
- subida a Cloudinary mediante mock;
- calificación;
- conversión;
- colección;
- reportes;
- notificaciones.

## End-to-end

Flujos completos prioritarios:

```text
Registro -> Verificación -> Login -> Subir obra
Registro -> Login -> Calificar obra
Fin de semana -> Scheduler -> Carta
Login -> Abrir paquete -> Colección
Login -> Seleccionar carta -> Rival -> Batalla
Usuario -> Reporte -> Moderador -> Resolución
```

---

# 26. Orden recomendado de implementación

## Fase 1 — Fundaciones

- proyecto NestJS;
- PostgreSQL;
- TypeORM;
- configuración por entorno;
- migraciones;
- estructura de módulos;
- manejo global de errores;
- validación global;
- autenticación JWT;
- roles.

## Fase 2 — Usuarios

- registro;
- verificación email;
- login;
- recuperación de contraseña;
- Google OAuth;
- perfiles;
- follows.

## Fase 3 — Obras

- tags;
- Cloudinary;
- creación;
- edición/eliminación;
- explorador;
- comentarios.

## Fase 4 — Q2Q

- ratings;
- emociones;
- período de una semana;
- restricciones de cierre.

## Fase 5 — Cartas

- algoritmo de conversión;
- `conversion_status`;
- Cron/Scheduler - rutina para el algoritmo de conversión;
- cartas;
- colección.

## Fase 6 — Paquetes

- inventario de paquetes;
- regeneración de 1 minuto;
- apertura;
- distribución de cartas.

## Fase 7 — Juego

- endpoint de rival;
- simulador local;
- UI de batalla.

## Fase 8 — Moderación y notificaciones

- reportes de obras;
- reportes de comentarios;
- resolución;
- auditoría;
- notificaciones.

## Fase 9 — Hardening

- rate limiting (**implementado**, ver 12.5 y `docs/shared/api-conventions.md` §4.1);
- sanitización (**implementado**, ver 13.1 y `docs/shared/api-conventions.md` §4.2);
- tests (**ampliado**: cobertura agregada sobre `JwtStrategy`, `JwtAuthGuard`, utils puras de mapeo de respuesta — `toArtworkView`/`toCardView`/`toOpponentCardView` —, `buildImageValidationPipe`, `GoogleStrategy` y el middleware de logging; antes sin ningún test propio);
- logs (**implementado**, ver 15.3 y `docs/shared/api-conventions.md` §4.3);
- métricas;
- revisión de queries e índices;
- revisión de seguridad.

## Extensión — Favoritos, borrado con cascada y rol de admin/apelaciones

Construidos después de la Fase 8, a pedido directo y fuera del orden de fases original: favoritos de obras/cartas, borrado lógico de obras con cascada a comentarios/valoraciones/carta, validación real de contenido de archivo, notificación apilable de votos (RF19–RF21 y RF25–RF27 de la sección 5). El detalle de diseño e implementación de cada uno vive en `docs/mejoras-post-fase-8.md` y `docs/mejoras-admin-apelaciones.md`; este documento incorpora sus decisiones estructurales (actores, reglas de negocio, modelo de datos, endpoints y guards) para que el diseño general se mantenga como referencia única y actualizada.

## Extensión — Herramientas de admin para demo (carga directa de carta y disparo manual de conversión)

También construidas fuera del orden de fases original, a pedido directo: carga directa de obra+carta por un admin sin pasar por el período de calificación (RF28, CU21) y disparo manual del ciclo de conversión que normalmente corre por CRON (RF29, CU22). Ninguna de las dos requirió cambios de modelo de datos — reutilizan las mismas tablas/columnas ya definidas en la sección 9 (`artworks.conversion_status`, `cards`) y, en el caso del disparo manual, exactamente la misma rutina de conversión de la sección 16. El contrato HTTP completo (`POST /artworks/admin-upload`, `POST /conversion/run`) vive en `docs/shared/api-conventions.md` §18.8–18.9.

## Extensión — Recuperación de contraseña

Agregada después de la implementación inicial de la Fase 2, a pedido directo (RF03b, CU23). Reutiliza exactamente el mismo mecanismo que la verificación de email (9.16): token opaco de un solo uso, hasheado con SHA-256 antes de persistir, con expiración explícita — sólo que en tabla propia (`password_reset_tokens`, 9.19) para no mezclar los dos propósitos, y con una vigencia más corta (1 hora en vez de 24) por ser un token más sensible. Sigue el mismo patrón anti-enumeración que el reenvío de verificación (7.2 de `api-conventions.md`): responde con el mismo mensaje neutro exista o no la cuenta, y también para una cuenta creada exclusivamente vía Google (sin contraseña local, nada que recuperar). Efecto adicional al actualizar la contraseña: revoca todas las sesiones (`refresh_tokens`) activas de la cuenta. El contrato HTTP completo (`POST /auth/forgot-password`, `POST /auth/reset-password`) vive en `docs/shared/api-conventions.md` §7.5.

---

# 27. Recomendaciones de diseño para evitar deuda técnica

1. Mantener la generación de cartas como servicio de dominio independiente del Scheduler.
2. No mezclar entidades TypeORM con DTOs de entrada.
3. No confiar en restricciones del frontend para autorización.
4. Aplicar ownership checks además de roles.
5. Usar transacciones para apertura de paquetes y conversión de cartas.
6. Registrar estados de procesos que puedan reintentarse.
7. Mantener soft delete en contenido moderado cuando la auditoría lo requiera.
8. Mantener la lógica de batalla fuera del backend mientras no haya persistencia ni interacción remota.
9. Evitar endpoints que existan sólo porque “podrían hacer falta” pero no tengan una operación de dominio real.
10. Versionar las migraciones de PostgreSQL desde el comienzo.


---

# 28. Resumen arquitectónico final

Pandora queda definido como un sistema cliente-servidor compuesto por un portal web y una aplicación Android que consumen una API NestJS. PostgreSQL centraliza usuarios, contenido, calificaciones, cartas, inventarios, relaciones, moderación y notificaciones. Cloudinary gestiona los assets gráficos.

La principal secuencia de negocio es:

```text
Usuario publica obra
        |
        v
Período Q2Q de 7 días
        |
        +--> comentarios continúan habilitados
        |
        v
Fin de período
        |
        v
Scheduler detecta obras elegibles
        |
        v
Conversión a carta
        |
        v
Carta disponible para colección
        |
        v
Paquetes
        |
        v
Colección del usuario
        |
        v
Autobattler local
```

La separación entre backend y cliente es especialmente importante en el juego: el backend mantiene únicamente la autoridad necesaria sobre inventario y selección del rival, mientras la simulación de batalla no se considera información de negocio persistente.


