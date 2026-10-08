# Pendientes de la app móvil

> Decisiones abiertas y valores que la app Android necesita. Todo lo que implique un cambio en frontend o backend está en [`_todo-frontend.md`](/_todo-frontend.md) y en el documento correspondiente (`requerimientos.md`, `api-conventions.md`); acá sólo queda lo propio del cliente móvil. Fuera de alcance en mobile: moderación, gestión de apelaciones por el personal y herramientas de administración (ver `requerimientos.md` §2.1).

## 1. Configuración por entorno (`local.properties`)

Valores que no se versionan y que se proveerán en `local.properties` (ya ignorado por git):

| Clave | Uso |
|---|---|
| `API_BASE_URL` | URL base del backend, hasta `/api/v1/` inclusive (con `/` final). En emulador, el host local es `10.0.2.2`. |
| `GOOGLE_CLIENT_ID` | Client ID **web** de Google, usado como `serverClientId` en Credential Manager (`GetGoogleIdOption`). Debe coincidir con `GOOGLE_CLIENT_ID` del backend, que es la `audience` que verifica `POST /auth/google` (`interfaces-entre-sistemas.md` §4.2). |

Trabajo en mobile:
- Leer ambas claves en `app/build.gradle.kts` y exponerlas como `BuildConfig` (hoy `buildConfig = false`; hay que activarlo).
- Fallar el build con mensaje claro si falta alguna en `release`; en `debug`, usar un valor por defecto (`http://10.0.2.2:3000/api/v1/`) y deshabilitar el botón de Google si no hay client ID.
- Permitir tráfico HTTP sólo en debug (network security config), nunca en release.
- Registrar en Google Cloud el cliente Android (paquete + SHA-1 de la firma debug y release) además del cliente web.

## 2. Reglas de la batalla local

**Resuelto (2026-10-07):** la fórmula ya está acordada y documentada en [`battle-formula.md`](/battle-formula.md) — orden de turnos por velocidad, esquive (tope 75%), golpe crítico (sin tope, ignora esquive) y reducción de daño (tope 90%), con cada decisión de implementación (empate de velocidad, redondeo, piso de daño, tope de rondas) marcada explícitamente para que `BattleEngine` la replique igual. Implementación de referencia en el frontend web: `src/features/battle/simulate.ts` (incluye tests unitarios de cada fórmula por separado).

**Pendiente en mobile:** la propuesta provisoria de abajo (`daño = max(1, attack − defense/2)`, sin esquive ni crítico) queda obsoleta — hay que migrar `BattleEngine` a la fórmula de `battle-formula.md`. Se mantiene aislado en su propia clase (determinista con semilla inyectada) para que este cambio no toque la UI:

- Emite una lista de eventos (turno, atacante, daño, `isCrit`, `dodged`, HP restante) que la UI anima — la UI web ya distingue los tres casos en el log (golpe normal / crítico / esquive), mobile debería hacer lo mismo. No persiste nada ni llama al backend salvo `GET /battle/opponent`.
- Tests unitarios: cada fórmula (esquive, reducción de daño, crítico) por separado además del flujo completo — mismo criterio que `simulate.test.ts` en el frontend web.

<details>
<summary>Propuesta original (obsoleta, dejada de referencia)</summary>

- Orden de turnos por `speed` (mayor primero; empate: la carta del jugador).
- Turnos alternados; `daño = max(1, attack − defense / 2)`.
- Termina cuando `hp ≤ 0`; tope de 50 turnos, gana quien tenga mayor porcentaje de HP restante (empate = empate).

</details>

## 3. Avatar de perfil

Hasta que exista un endpoint de subida (ver `_todo-frontend.md`):
- La pantalla de edición permite cambiar `username` y `bio` (validación local: `^[a-zA-Z0-9_]{1,30}$` y ≤300).
- El avatar se muestra desde `avatarUrl` (Google o URL existente); sin selector de imagen.
- Al existir el endpoint: agregar el selector (photo picker, sin permisos de almacenamiento), validar formato y 25 MB, y subir en `multipart` en streaming.

## 4. Enlaces de verificación y reset de contraseña

Decisión vigente: abrir en el navegador, sin deep links.
- Tras registrarse: pantalla "Revisá tu correo" con botón *Reenviar* (`POST /auth/resend-verification`, límite 5/min) y botón *Ya verifiqué* que vuelve a consultar `GET /auth/me` y continúa si `emailVerified` es `true`.
- Olvidé mi contraseña: pide el email (`POST /auth/forgot-password`), muestra el mensaje neutro de la API y vuelve al login; el cambio se hace en el portal web.
- Si más adelante se habilitan App Links (ver `_todo-frontend.md`): declarar el `intent-filter` con `autoVerify` para `/verify-email` y `/reset-password`, y mostrar pantallas nativas que llamen a `POST /auth/verify-email` / `/auth/reset-password`.
