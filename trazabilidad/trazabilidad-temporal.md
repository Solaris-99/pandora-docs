# Trazabilidad temporal (orden histórico de implementación)

## Fase 1 — Fundaciones

Proyecto NestJS, PostgreSQL, TypeORM, configuración por entorno, migraciones, estructura de módulos, manejo global de errores, validación global, autenticación JWT, roles.

## Fase 2 — Usuarios

Registro, verificación email, login, recuperación de contraseña, Google OAuth, perfiles, follows.

## Fase 3 — Obras

Tags, Cloudinary, creación, edición/eliminación, explorador, comentarios.

## Fase 4 — Q2Q

Ratings, emociones, período de una semana, restricciones de cierre.

## Fase 5 — Cartas

Algoritmo de conversión, `conversion_status`, Cron/Scheduler (rutina de conversión), cartas, colección.

## Fase 6 — Paquetes

Inventario de paquetes, regeneración de 1 minuto, apertura, distribución de cartas.

## Fase 7 — Juego

Endpoint de rival, simulador local, UI de batalla.

## Fase 8 — Moderación y notificaciones

Reportes de obras, reportes de comentarios, resolución, auditoría, notificaciones.

## Fase 9 — Hardening

- rate limiting (**implementado** — RNF03);
- sanitización (**implementado** — RNF09);
- tests (**ampliado**: cobertura agregada sobre `JwtStrategy`, `JwtAuthGuard`, utils puras de mapeo de respuesta — `toArtworkView`/`toCardView`/`toOpponentCardView` —, `buildImageValidationPipe`, `GoogleStrategy` y el middleware de logging; antes sin ningún test propio — ver [casos de prueba](/casos-de-prueba/README.md));
- logs (**implementado** — RNF14);
- métricas (pendiente);
- revisión de queries e índices (pendiente);
- revisión de seguridad (pendiente).

## Extensión — Favoritos, borrado con cascada y rol de admin/apelaciones

Construidos después de la Fase 8, a pedido directo y fuera del orden de fases original: favoritos de obras/cartas, borrado lógico de obras con cascada a comentarios/valoraciones/carta, validación real de contenido de archivo, notificación apilable de votos (RF19–RF21 y RF25–RF27). El detalle de diseño e implementación de cada uno vive en `docs/mejoras-post-fase-8.md` y `docs/mejoras-admin-apelaciones.md`.

## Extensión — Herramientas de admin para demo (carga directa de carta y disparo manual de conversión)

También construidas fuera del orden de fases original, a pedido directo: carga directa de obra+carta por un admin sin pasar por el período de calificación (RF28, [CU29](/casos-de-uso/cu29-cargar-carta-directamente.md)) y disparo manual del ciclo de conversión que normalmente corre por CRON (RF29, [CU30](/casos-de-uso/cu30-disparar-el-ciclo-de-conversion-manualmente.md)). Ninguna de las dos requirió cambios de modelo de datos — reutilizan las mismas tablas/columnas ya definidas (`artworks.conversion_status`, `cards`) y, en el caso del disparo manual, exactamente la misma rutina de conversión de la Fase 5. El contrato HTTP completo vive en [convenciones de API](/api-conventions.md) §18.8–18.9.

## Extensión — Recuperación de contraseña

Agregada después de la implementación inicial de la Fase 2, a pedido directo (RF03b, [CU04](/casos-de-uso/cu04-recuperar-contrasena-olvidada.md)). Reutiliza exactamente el mismo mecanismo que la verificación de email: token opaco de un solo uso, hasheado con SHA-256 antes de persistir, con expiración explícita — sólo que en tabla propia (`password_reset_tokens`) para no mezclar los dos propósitos, y con una vigencia más corta (1 hora en vez de 24) por ser un token más sensible. Sigue el mismo patrón anti-enumeración que el reenvío de verificación: responde con el mismo mensaje neutro exista o no la cuenta, y también para una cuenta creada exclusivamente vía Google (sin contraseña local, nada que recuperar). Efecto adicional al actualizar la contraseña: revoca todas las sesiones (`refresh_tokens`) activas de la cuenta. El contrato HTTP completo vive en [convenciones de API](/api-conventions.md) §7.5.
