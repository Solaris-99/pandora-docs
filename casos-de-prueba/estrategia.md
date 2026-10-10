# Estrategia de pruebas

## Pruebas unitarias (backend)

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

## Pruebas de integración

- registro + verificación;
- login;
- creación de obra;
- subida a Cloudinary mediante mock;
- calificación;
- conversión;
- colección;
- reportes;
- notificaciones.

## Pruebas end-to-end (flujos completos prioritarios)

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
