# Trazabilidad de requerimientos no funcionales

| RNF | Descripción corta | Mecanismo / componente | Estado |
|---|---|---|---|
| RNF01 | Peso/formato de obras | `buildImageValidationPipe` ([clases de diseño, módulo `common`](/clases-diseno/common.md)) | Implementado |
| RNF02 | Hash de contraseñas | `bcrypt` en `AuthService.register`/`resetPassword` | Implementado |
| RNF03 | Rate limit de login | `AppThrottlerGuard` + `@Throttle` ([diseño de componentes, rate limiting](/diseno-componentes/guards.md#rate-limiting)) | Implementado (Fase 9) |
| RNF04 | Roles | `RolesGuard` | Implementado |
| RNF05 | Usabilidad | Responsabilidad de los clientes (web/Android), fuera del alcance del backend | N/A backend |
| RNF06 | Storage de imágenes | `StorageService` (Cloudinary) | Implementado |
| RNF07 | Verificación de email | `EmailVerificationToken` + `AuthService` | Implementado |
| RNF08 | Seguridad de endpoints | Combinación `JwtAuthGuard` + `RolesGuard` + `BannedUserGuard` + `VerifiedEmailGuard` + `AppThrottlerGuard` + sanitización + ownership checks en servicio | Implementado |
| RNF09 | Sanidad de campos | `@SanitizeText()` + límites de longitud en DTOs ([diseño de componentes, validación general](/diseno-componentes/validaciones.md#validación-general)) | Implementado (Fase 9) |
| RNF10 | Manejo de errores | `AllExceptionsFilter` | Implementado |
| RNF11 | Integridad | Transacciones en `ConversionSchedulerService` y `PackagesService` | Implementado |
| RNF12 | Idempotencia de conversión | `UNIQUE(artwork_id)` en `cards` + claim atómico (`UPDATE ... WHERE conversion_status IN (...)`) | Implementado |
| RNF13 | Auditoría de moderación | `resolved_by`/`resolved_at` en reportes, `reviewed_by`/`reviewed_at` en apelaciones | Implementado |
| RNF14 | Observabilidad | `loggingMiddleware` + `AllExceptionsFilter` + logs puntuales en `MailService`/`StorageService`/`ConversionSchedulerService` | Implementado (Fase 9) |
| RNF15 | Paginación | `PaginationDto`/`PaginatedResponseDto` en galería, comentarios, notificaciones y reportes | Implementado |

Ítems de Fase 9/Hardening todavía **pendientes** a la fecha de este documento: métricas, revisión de queries e índices, revisión de seguridad.
