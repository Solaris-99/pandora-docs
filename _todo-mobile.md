# Pendientes de la app móvil

> Decisiones abiertas y valores que la app Android necesita. Todo lo que implique un cambio en frontend o backend está en [`_todo-frontend.md`](/_todo-frontend.md) y en el documento correspondiente ([requerimientos](/requerimientos/README.md), `api-conventions.md`); acá sólo queda lo propio del cliente móvil. Fuera de alcance en mobile: moderación, gestión de apelaciones por el personal y herramientas de administración (ver [requerimientos §2.1](/requerimientos/alcance.md)).

## 1. Configuración por entorno (`local.properties`)

Valores que no se versionan y que se proveerán en `local.properties` (ya ignorado por git):

| Clave | Uso |
|---|---|
| `API_BASE_URL` | URL base del backend, hasta `/api/v1/` inclusive (con `/` final). En emulador, el host local es `10.0.2.2`. |
| `GOOGLE_CLIENT_ID` | Client ID **web** de Google, usado como `serverClientId` en Credential Manager (`GetGoogleIdOption`). Debe coincidir con `GOOGLE_CLIENT_ID` del backend, que es la `audience` que verifica `POST /auth/google` ([interfaces entre sistemas, flujo Android](/interfaces-entre-sistemas/google.md#flujo-android-verificación-de-id-token)). |

Trabajo en mobile:
- Leer ambas claves en `app/build.gradle.kts` y exponerlas como `BuildConfig` (hecho en la Fase 1).
- Fallar el build con mensaje claro si falta alguna en `release`; en `debug`, usar un valor por defecto (`http://10.0.2.2:3000/api/v1/`) y deshabilitar el botón de Google si no hay client ID.
- Permitir tráfico HTTP sólo en debug (network security config), nunca en release.
- Registrar en Google Cloud el cliente Android (paquete + SHA-1 de la firma debug y release) además del cliente web.

## 2. Reglas de la batalla local

Implementado según [`battle-formula.md`](/battle-formula.md) en `BattleEngine` (Kotlin puro, sin Android): orden de turnos fijado al inicio con prioridad del jugador en empate, crítico (`crit_chance`), esquive solo si no hubo crítico (tope 75 %), daño base y crítico, reducción de daño (tope 90 %), redondeo al más cercano con mitades hacia arriba, piso de 1 de daño, fin inmediato al llegar a 0 HP y tope de 500 rondas. El generador aleatorio se inyecta y el **orden de las tiradas es parte del contrato**: por ataque una tirada de crítico y, solo si no fue crítico, una de esquive.

Pendiente de coordinar con el frontend: pasar un par de casos de referencia (stats + secuencia de tiradas + resultado esperado) obtenidos de `src/features/battle/simulate.ts` para agregarlos como tests y confirmar que ambos clientes dan el mismo resultado. Si `battle-formula.md` cambia, se actualiza el documento primero y luego ambos clientes.

## ~~3. Avatar de perfil~~ — resuelto

Implementado con `POST /users/me/avatar` ([CU31](/casos-de-uso/cu31-subir-avatar.md)): en "Editar perfil", selector del sistema (photo picker, sin permisos de almacenamiento), validación local de formato (PNG/JPG/WEBP) y 2 MB, subida en streaming con barra de progreso. El botón se deshabilita con un aviso si el email no está verificado.

## 4. Enlaces de verificación y reset de contraseña

Decisión vigente: abrir en el navegador, sin deep links.
- Tras registrarse: pantalla "Revisá tu correo" con botón *Reenviar* (`POST /auth/resend-verification`, límite 5/min) y botón *Ya verifiqué* que vuelve a consultar `GET /auth/me` y continúa si `emailVerified` es `true`.
- Olvidé mi contraseña: pide el email (`POST /auth/forgot-password`), muestra el mensaje neutro de la API y vuelve al login; el cambio se hace en el portal web.
- Si más adelante se habilitan App Links (ver `_todo-frontend.md`): declarar el `intent-filter` con `autoVerify` para `/verify-email` y `/reset-password`, y mostrar pantallas nativas que llamen a `POST /auth/verify-email` / `/auth/reset-password`.

## ~~5. Cambiar email y cambiar contraseña~~ — resuelto

Pantallas nativas en Perfil → Cuenta ([CU07](/casos-de-uso/cu07-cambiar-email.md) / [CU08](/casos-de-uso/cu08-cambiar-contrasena.md)). Cambiar email pide confirmación en un popup antes de enviar y, como acordado, el enlace del correo se abre en el navegador. Cambiar contraseña valida la política del servidor (8–50, mayúscula, minúscula, número y símbolo) y que ambas contraseñas coincidan.

Punto a aclarar con backend: `api-conventions.md` §8.6 dice que el cambio revoca **todos** los `refresh_tokens` activos, y `_todo-backend.md` dice **los demás**. La app mantiene la sesión abierta tras el cambio (igual que el portal web); si el servidor también revoca el token de este dispositivo, el usuario tendrá que volver a iniciar sesión cuando venza el `accessToken` (≤15 min).

## 6. Publicación (Fase 8)

Hecho en el repositorio mobile:
- **Release**: R8 y reducción de recursos activados (`app/proguard-rules.pro` conserva los serializadores de `@Serializable`); el build de `release` exige `API_BASE_URL` con `https://`. El tráfico HTTP en claro sólo existe en `debug`. `allowBackup=false`; además `session.xml` (tokens cifrados) queda excluido de backup y de transferencia entre dispositivos.
- **Firma**: `keystore.properties` en la raíz (ignorado por git) con `storeFile`, `storePassword`, `keyAlias` y `keyPassword`. Sin ese archivo, `release` se genera sin firmar (es lo que usa el CI).
- **Idiomas**: español (por defecto) e inglés (`values-en`); un test unitario verifica que ambos tengan las mismas claves y placeholders. Los mensajes de error que llegan en `message` del backend no se muestran: la app mapea `code` a texto propio.
- **Tablet / horizontal**: desde 600 dp de ancho las pestañas pasan a un `NavigationRail` y el contenido se centra con ancho máximo de 840 dp; las grillas son adaptativas (≥156 dp por columna).
- **Accesibilidad**: objetivos táctiles de 48 dp (estrellas, chips, contadores, filas de autor), descripciones y etiquetas de acción en imagen ampliable, carta por revelar y vista previa de subida.
- **CI**: `.github/workflows/ci.yml` ejecuta `test lintDebug assembleDebug`, `minifyReleaseWithR8` y compila los tests instrumentados.
- **Smoke test instrumentado** (`SmokeTest`): contra un servidor falso, explora como invitado → inicia sesión desde la campana → abre un paquete.

Pendiente fuera del repositorio (no se puede resolver desde el código):
- Crear en Google Cloud el cliente OAuth **Android** (paquete `com.example.pandora` + SHA-1 de debug y de la clave de release); sin él, "Continuar con Google" falla en el dispositivo.
- Generar el keystore de release y guardarlo fuera del repositorio.
- Ejecutar `./gradlew connectedDebugAndroidTest` en un emulador/dispositivo: hasta ahora la app sólo se compiló y se probó con tests de JVM.
