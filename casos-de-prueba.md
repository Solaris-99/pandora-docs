# Pandora — Casos de Prueba

Este documento cubre la estrategia de pruebas y un caso de prueba por [caso de uso](/casos-de-uso.md), derivado de su flujo principal, su postcondición y sus flujos alternativos. La cobertura automatizada real (Jest) vive en el código fuente, bajo `src/**/*.spec.ts`, a la fecha de este documento, 50 suites con más de 330 pruebas unitarias.

# 1. Estrategia de pruebas

## 1.1 Pruebas unitarias (backend)

- validación de usuario;
- hash de contraseña;
- expiración de calificación;
- una valoración por usuario/obra;
- conversión de artwork;
- idempotencia del CRON;
- cálculo de paquetes;
- apertura atómica de paquete;
- permisos de ownership;
- permisos de moderación;
- `JwtStrategy`/`JwtAuthGuard` (resolución y bloqueo de usuarios suspendidos/baneados);
- utilidades puras de mapeo de respuesta (`toArtworkView`, `toCardView`, `toOpponentCardView`);
- sanitización de texto libre (remoción de HTML).

## 1.2 Pruebas de integración

- registro + verificación;
- login;
- creación de obra;
- subida a Cloudinary mediante mock;
- calificación;
- conversión;
- colección;
- reportes;
- notificaciones.

## 1.3 Pruebas end-to-end (flujos completos prioritarios)

```text
Registro -> Verificación -> Login -> Subir obra
Registro -> Login -> Calificar obra
Fin de semana -> Scheduler -> Carta
Login -> Abrir paquete -> Colección
Login -> Seleccionar carta -> Rival -> Batalla
Usuario -> Reporte -> Moderador -> Resolución
Usuario -> Reporte -> Moderador resuelve -> Afectado apela -> Otro moderador revisa
Usuario olvida contraseña -> Recupera -> Inicia sesión con la nueva
```

---

# 2. Casos de prueba por caso de uso

Cada caso de prueba usa como **pasos** el flujo principal del caso de uso correspondiente, como **resultado esperado** su postcondición, y como **casos negativos/alternativos** sus flujos `A*`.

## CP01 — Registro exitoso (CU01)

- **Precondición:** no existe una cuenta con ese email ni ese username.
- **Pasos:** completar email/username/contraseña válidos → backend valida formato/longitud/unicidad → hashea la contraseña → crea la cuenta → emite token de verificación → envía el correo.
- **Resultado esperado:** cuenta creada en estado `pending_verification`; al confirmar el challenge, pasa a `active` y `email_verified = true`.
- **Casos negativos:** email duplicado (A1) · username duplicado (A2) · email inválido (A3) · contraseña inválida (A4) · fallo de envío de correo no aborta la creación de la cuenta (A5) · challenge expirado (A6) · challenge inválido (A7).

## CP02 — Login (CU02)

- **Precondición:** cuenta registrada.
- **Pasos:** enviar email/contraseña → backend aplica rate limit → busca el usuario → verifica credenciales → verifica estado de la cuenta → genera JWT.
- **Resultado esperado:** par de tokens (`accessToken`/`refreshToken`) y datos del usuario.
- **Casos negativos:** credenciales inválidas → 401 genérico (A1) · demasiados intentos → 429 (A2, verificado en vivo: 6ª petición en un minuto bloqueada) · cuenta baneada → sigue permitiendo login, pero bloquea escritura vía `BannedUserGuard` (A3, ver regla 5.1.6 de `pandora-requerimientos.md`) · cuenta sin verificar → mensaje específico (A4) · error de Google → mensaje genérico (A5).

## CP03 — Subir obra (CU03)

- **Precondición:** usuario autenticado, con email verificado.
- **Pasos:** completar título, imagen, y opcionalmente lore/tags/solicitud de conversión → backend valida campos y archivo → sube a Cloudinary → registra la obra con `qualification_started_at = now()`.
- **Resultado esperado:** obra creada en `conversion_status = pending`, visible en el explorador, notificando a seguidores del autor.
- **Casos negativos:** archivo > 25 MB (A1) · formato no permitido, incluyendo mimetype falsificado (A2) · título vacío o demasiado largo (A3) · fallo de Cloudinary no debe dejar una obra incompleta (A4) · usuario baneado o sin email verificado → 403 (A6).

## CP04 — Explorar obras (CU04)

- **Precondición:** ninguna (accesible sin sesión).
- **Pasos:** abrir el explorador → aplicar filtros (búsqueda, autor, fecha, tag, rareza) → paginar → abrir el detalle de una obra.
- **Resultado esperado:** listado paginado de obras no eliminadas; detalle completo al seleccionar una.
- **Casos negativos:** sin resultados → estado vacío, no error (A1) · filtro inválido → 400 (A2) · obra eliminada entre búsqueda y detalle → recurso no disponible (A3).

## CP05 — Calificar obra mediante Q2Q (CU05)

- **Precondición:** obra dentro de su semana de calificación; usuario autenticado.
- **Pasos:** abrir detalle → backend confirma que la calificación sigue habilitada → elegir estrellas (1–5) y emoción → registrar o actualizar la valoración.
- **Resultado esperado:** valoración persistida (una por usuario/obra); si ya existía, se actualiza en vez de duplicarse.
- **Casos negativos:** período expirado (A1) · sin sesión (A2) · emoción inválida → 400 (A3) · estrellas fuera de rango → 400 (A4) · obra eliminada → rechazar (A6).

## CP06 — Conversión de obra a carta (CU06)

- **Precondición:** obra con período de calificación terminado y `conversion_request = true`.
- **Pasos:** el scheduler busca obras elegibles → obtiene valoraciones/emociones → ejecuta el algoritmo de conversión → crea la `Card` → marca la obra `converted` → notifica.
- **Resultado esperado:** exactamente una `Card` por obra elegible; `conversion_status = converted`.
- **Casos negativos:** obra no elegible → se ignora (A1) · carta ya existente → se marca `converted` sin duplicar, protegido por `UNIQUE(artwork_id)` (A2) · error de cálculo → estado reintentable (A3) · error de transacción → rollback (A4).
- **Cobertura automatizada:** `conversion-scheduler.service.spec.ts` cubre los 5 escenarios anteriores explícitamente (ciclo sin candidatos, conversión exitosa, carta duplicada, claim perdido ante otra ejecución concurrente, y fallo con marcado `failed`).

## CP07 — Explorar colección/cartas (CU07)

- **Precondición:** ninguna (detalle público accesible sin sesión; posesión requiere sesión).
- **Pasos:** abrir la sección de cartas → aplicar filtros → paginar → abrir detalle de una carta.
- **Resultado esperado:** si el usuario posee la carta (`copies > 0`), se muestran estadísticas completas; si no, `stats: null`.
- **Casos negativos:** carta no poseída → estadísticas ocultas, no error (A1) · carta inexistente → 404 (A2).

## CP08 — Abrir paquete (CU08)

- **Precondición:** el usuario tiene al menos un paquete disponible.
- **Pasos:** solicitar apertura → backend verifica disponibilidad → consume exactamente un paquete → sortea 5 cartas por rareza ponderada → incrementa copias en la colección → devuelve las cartas obtenidas.
- **Resultado esperado:** `amount` decrementado en 1; 5 cartas nuevas o incrementadas en `user_cards`; operación atómica.
- **Casos negativos:** sin paquetes disponibles → 409 (A1) · peticiones simultáneas no deben permitir doble consumo (A2) · fallo transaccional no deja estado parcial (A3).

## CP09 — Jugar autobattler (CU09)

- **Precondición:** usuario autenticado, posee al menos una carta.
- **Pasos:** seleccionar carta propia → solicitar rival → backend selecciona una carta rival válida al azar → cliente simula la batalla localmente.
- **Resultado esperado:** ambas cartas (propia y rival) con estadísticas completas; ningún resultado persiste en el backend.
- **Casos negativos:** carta no poseída → 403/400 (A1) · carta inexistente → 404 (A2) · sin cartas rivales disponibles → error controlado (A3).

## CP10 — Seguir usuario (CU10)

- **Pasos:** abrir perfil ajeno → pulsar "Seguir" → se crea el `Follow` → se notifica al usuario seguido.
- **Resultado esperado:** relación creada; notificación `user_follow` generada.
- **Casos negativos:** relación ya existente → idempotente, no error (A1) · intento de seguirse a sí mismo → rechazar (A2) · usuario objetivo inexistente/baneado → rechazar (A3).

## CP11 — Destacar obra o carta (CU11)

- **Pasos:** seleccionar obra propia o carta poseída → pulsar "Destacar" → se registra el favorito.
- **Resultado esperado:** el elemento aparece en la sección de destacados del perfil.
- **Casos negativos:** obra ajena (A1) · carta no poseída (A2) · ya destacada → idempotente (A3).

## CP12 — Denunciar obra (CU12)

- **Pasos:** seleccionar motivo predefinido → opcionalmente agregar comentario → registrar el reporte.
- **Resultado esperado:** `ArtworkReport` en estado `pending`.
- **Casos negativos:** motivo inválido (A1) · obra inexistente → 404 (A2) · obra ya eliminada (A3) · reportes duplicados idénticos → regla anti-spam (A4).

## CP13 — Denunciar comentario (CU13)

- **Pasos:** seleccionar motivo → opcionalmente agregar detalle → registrar `CommentReport`.
- **Resultado esperado:** reporte creado en `pending`.
- **Casos negativos:** comentario ya eliminado (A1) · motivo inválido → 400 (A2) · reporte duplicado (A3).

## CP14 — Administrar denuncias (CU14)

- **Precondición:** actor con rol `moderator` o `admin`.
- **Pasos:** abrir panel de reportes pendientes → abrir un reporte → elegir acción (descartar/advertir/eliminar obra/eliminar comentario/banear) → aplicar → marcar `resolved` → auditar → notificar.
- **Resultado esperado:** reporte `resolved` con `resolved_by`/`resolved_at`/`resolution` completos; sanción aplicada; notificación al afectado cuando corresponde.
- **Casos negativos:** contenido ya eliminado → cerrar sin duplicar acción (A1) · usuario ya baneado (A2) · sin permisos → 403 (A3) · dos moderadores resolviendo simultáneamente → control de concurrencia (A4).

## CP15 — Notificaciones (CU15)

- **Pasos:** ocurre un evento notificable → se crea la notificación → el usuario consulta su panel paginado → marca una o varias como leídas.
- **Resultado esperado:** notificación persistente, correctamente tipada, marcable como leída.

## CP16 — Ver/editar perfil (CU16)

- **Pasos:** abrir perfil → se devuelve información pública → si es el propio, habilitar edición → guardar cambios válidos.
- **Resultado esperado:** perfil actualizado; consulta pública siempre disponible.
- **Casos negativos:** username ya usado por otro (A1) · datos fuera de rango → 400 (A2) · edición sin autenticación → 401 (A3).

## CP17 — Gestionar rol de moderador (CU17)

- **Precondición:** actor con rol `admin`.
- **Pasos:** seleccionar usuario objetivo → otorgar o revocar `moderator` → confirmar.
- **Resultado esperado:** roles del usuario objetivo actualizados.
- **Casos negativos:** operación repetida → idempotente (A1) · usuario objetivo inexistente → 404 (A2) · actor sin rol `admin` → 403 (A3) · intento de otorgar/revocar `admin` por esta vía → fuera de alcance (A4).

## CP18 — Reabrir un reporte resuelto (CU18)

- **Precondición:** reporte `resolved` con `resolution` punitiva; actor `admin`.
- **Pasos:** abrir listado de resueltos → seleccionar reporte sancionado → confirmar reapertura → revertir la sanción → marcar `overturned` → notificar al afectado.
- **Resultado esperado:** contenido restaurado o usuario desbaneado; reporte en `overturned`.
- **Casos negativos:** ya reabierto o ya apelado y aprobado → 409 (A1) · resolución no punitiva → 400 (A2) · reporte inexistente/no resuelto → 404 (A3) · actor sin `admin` → 403 (A4).

## CP19 — Apelar una sanción (CU19)

- **Precondición:** reporte resuelto con sanción punitiva sobre el contenido/cuenta del actor, aún no apelada.
- **Pasos:** elegir motivo predefinido → opcionalmente agregar texto → enviar → apelación queda `pending`.
- **Resultado esperado:** una única `Appeal` por sanción, visible para moderación.
- **Casos negativos:** apelación previa para la misma sanción → 409 (A1) · sin sanción punitiva apelable → 404 (A2) · actor no es el afectado → 403 (A3) · cuenta baneada apelando su propio baneo → permitido explícitamente (A4).

## CP20 — Revisar una apelación (CU20)

- **Precondición:** apelación `pending`; revisor distinto de quien resolvió el reporte original (salvo moderador único).
- **Pasos:** abrir listado (excluye conflictos de revisor) → abrir detalle → aprobar o rechazar → si aprueba, revertir la sanción (mismo mecanismo que CU18) → notificar al apelante en ambos casos.
- **Resultado esperado:** apelación `approved`/`rejected`; si `approved`, mismo efecto que CU18.
- **Casos negativos:** revisor en conflicto con más de un moderador disponible → 403 (A1) · ya resuelta por otro → 409 (A2) · aprobación compite con reapertura directa del mismo reporte → se considera igualmente aprobada, no es error (A3) · sin rol de moderación → 403 (A4).

## CP21 — Cargar carta directamente (Admin) (CU21)

- **Precondición:** actor `admin`.
- **Pasos:** completar título/descripción/tags → seleccionar imagen → indicar rareza → opcionalmente estadísticas explícitas → backend sube la imagen, crea la obra `converted` y la carta en el mismo paso; estadísticas faltantes se completan según la rareza.
- **Resultado esperado:** obra + carta creadas atómicamente; sin período de calificación; admin no posee automáticamente la carta.
- **Casos negativos:** archivo inválido/ausente (A1) · título inválido (A2) · rareza inválida → 400 (A3) · estadística negativa o no entera → 400 (A4) · actor sin `admin` → 403 (A5).
- **Verificación en vivo realizada:** confirmado por HTTP real (no sólo test unitario) — override explícito de estadísticas respetado campo a campo, y default por rareza correctamente calculado cuando se omiten.

## CP22 — Disparar el ciclo de conversión manualmente (Admin) (CU22)

- **Precondición:** actor `admin`.
- **Pasos:** disparar la ejecución manual → el backend ejecuta `runConversionCycle()` (la misma rutina que CU06) → devuelve el resumen.
- **Resultado esperado:** `{processed, converted, failed, skipped}`; mismo efecto que una corrida real del CRON.
- **Casos negativos:** sin obras elegibles → resumen en cero, no error (A1) · actor sin `admin` → 403 (A2).
- **Verificación en vivo realizada:** confirmado por HTTP real; rechazo 403 para un usuario sin rol `admin` también verificado en vivo.

## CP23 — Recuperar contraseña olvidada (CU23)

- **Precondición:** ninguna del lado del actor (invitado); la cuenta puede existir o no.
- **Pasos:** solicitar recuperación con un email → respuesta neutra siempre → si la cuenta existe y tiene contraseña local, se emite un token de un solo uso y se envía por correo → el usuario abre el enlace → envía la nueva contraseña → backend valida el token, actualiza la contraseña y revoca todos los `refresh_tokens` activos → el usuario inicia sesión con la contraseña nueva.
- **Resultado esperado:** contraseña actualizada; sesiones previas invalidadas; no se inicia sesión automáticamente.
- **Casos negativos:** email inexistente → mismo mensaje neutro, nada se envía (A1) · cuenta exclusivamente de Google → mismo mensaje neutro, nada se envía (A2) · token inválido/inexistente/ya usado → 400 (A3) · token expirado (> 1 hora) → 400 (A4) · contraseña nueva fuera de política → 400 (A5).
- **Verificación en vivo realizada:** confirmado por HTTP real end-to-end — token bogus rechazado (400), token válido aceptado (200), reintento del mismo token rechazado (400, ya usado), login con contraseña vieja rechazado (401), login con contraseña nueva exitoso (200), y confirmado en base de datos que el `refresh_token` emitido antes del reset quedó `revoked_at` mientras el emitido después del login post-reset permanece activo.

