# Convenciones de API y Contratos Compartidos — Pandora

Este documento establece las especificaciones, formatos y contratos de comunicación entre el **Backend de Pandora** y sus clientes (**Portal Web** y **Aplicación Móvil Android**).

---

## 1. Prefijo Global y Versionado

Todos los endpoints REST del backend se encuentran bajo el prefijo:
```text
/api/v1
```

Ejemplos:
- `/api/v1/auth/register`
- `/api/v1/auth/login`
- `/api/v1/auth/me`
- `/api/v1/users/:id`

---

## 2. Formato Estándar de Respuestas de Error

Todo error devuelto por la API (errores 4xx y 5xx) sigue una estructura JSON homogénea y predecible:

```json
{
  "statusCode": 400,
  "code": "VALIDATION_ERROR",
  "message": "Mensaje legible para el usuario final o desarrollador.",
  "errors": [
    {
      "field": "email",
      "message": "El correo electrónico debe ser válido."
    }
  ],
  "timestamp": "2026-09-15T23:20:00.000Z",
  "path": "/api/v1/auth/register"
}
```

### Campos:
| Campo | Tipo | Descripción |
|---|---|---|
| `statusCode` | number | Código de estado HTTP (400, 401, 403, 404, 409, 500, etc.) |
| `code` | string | Identificador programático del error en formato `SNAKE_CASE_UPPER` |
| `message` | string | Descripción comprensible del error |
| `errors` | array (opcional) | Lista detallada de fallos de validación por campo |
| `timestamp` | string (ISO 8601) | Momento exacto en el que ocurrió el error |
| `path` | string | Ruta y endpoint en el que se produjo la petición |

> [!IMPORTANT]
> Por políticas de seguridad (RNF10), los errores 500 devuelven un mensaje genérico (`"Ha ocurrido un error interno en el servidor."`) y **nunca** exponen detalles internos ni volcados de base de datos o stack traces.

---

## 3. Catálogo de Códigos de Error

| Código (`code`) | HTTP Status | Causa típica |
|---|---|---|
| `VALIDATION_ERROR` | 400 | Los datos enviados en el body o query no cumplen las restricciones del DTO. |
| `BAD_REQUEST` | 400 | Petición mal formada o parámetros no válidos. |
| `UNAUTHORIZED` | 401 | Token JWT ausente, expirado o con firma inválida. |
| `INVALID_CREDENTIALS` | 401 | Correo o contraseña incorrectos en `/auth/login`. |
| `FORBIDDEN` | 403 | El usuario autenticado carece de permisos suficientes para la acción. |
| `USER_BANNED` | 403 | La cuenta del usuario ha sido baneada permanentemente por moderación. |
| `USER_SUSPENDED` | 403 | La cuenta del usuario está suspendida temporalmente. |
| `USER_NOT_VERIFIED` | 403 | La acción requiere una cuenta con email verificado (p. ej. publicar una obra o comentar). |
| `NOT_FOUND` | 404 | El recurso solicitado no existe. |
| `USER_NOT_FOUND` | 404 | El usuario especificado no fue encontrado. |
| `CONFLICT` | 409 | Conflicto de estado o duplicidad de clave única en la base de datos. |
| `EMAIL_ALREADY_EXISTS` | 409 | El email ya está registrado por otra cuenta. |
| `USERNAME_ALREADY_EXISTS` | 409 | El username ya está registrado por otra cuenta. |
| `INVALID_TOKEN` | 400 | Token de verificación de email inválido o ya utilizado. |
| `TOKEN_EXPIRED` | 400 | Token de verificación de email expirado (vigencia: 24 horas). |
| `USER_ALREADY_VERIFIED` | 400 | Se intentó reenviar verificación a una cuenta ya verificada. |
| `CANNOT_FOLLOW_SELF` | 400 | Se intentó seguir/dejar de seguir a la propia cuenta. |
| `QUALIFICATION_ENDED` | 403 | Se intentó crear, actualizar o retirar una valoración Q2Q fuera de la semana de calificación de la obra. |
| `OAUTH_ERROR` | 400/401 | Falla al validar el `idToken` o el perfil devuelto por Google. |
| `NO_PACKAGES_AVAILABLE` | 409 | Se intentó abrir un paquete sin tener ninguno disponible. |
| `NO_CARDS_AVAILABLE` | 409 | Aún no existe ninguna carta en el sistema para otorgar al abrir un paquete. |
| `CARD_NOT_OWNED` | 403 | Se intentó usar como propia una carta que el usuario no posee (p. ej. al pedir un rival de batalla). |
| `NO_OPPONENT_AVAILABLE` | 409 | No existe ninguna otra carta en el sistema para actuar como rival de batalla. |
| `REPORT_ALREADY_PENDING` | 409 | El usuario ya tiene un reporte pendiente sobre esa misma obra/comentario. |
| `REPORT_ALREADY_RESOLVED` | 409 | El reporte ya fue resuelto (por este u otro moderador) antes de esta petición. |
| `CONTENT_ALREADY_REMOVED` | 409 | Se intentó reportar una obra o comentario que ya fue eliminado. |
| `INVALID_REPORT_ACTION` | 400 | La acción de resolución no corresponde al tipo de contenido reportado (p. ej. `comment_removed` sobre un reporte de obra). |
| `NOTIFICATION_NOT_FOUND` | 404 | La notificación no existe o no pertenece al usuario autenticado. |
| `APPEAL_ALREADY_EXISTS` | 409 | Ya existe una apelación (de cualquier estado) sobre esta sanción — sólo se permite una por reporte. |
| `APPEAL_ALREADY_RESOLVED` | 409 | La apelación ya fue resuelta (por este u otro moderador) antes de esta petición. |
| `APPEAL_REVIEWER_CONFLICT` | 403 | El moderador que intenta resolver la apelación es el mismo que resolvió el reporte original, y hay más de un moderador/admin disponible. |
| `REPORT_NOT_OVERTURNABLE` | 400/409 | Se intentó reabrir un reporte que no está en estado `resolved`, o cuya resolución no tiene ninguna sanción que revertir (`dismissed`/`warning`), o que ya fue reabierto por otro camino. |
| `TOO_MANY_REQUESTS` | 429 | Límite de peticiones excedido (rate limiting). |
| `INTERNAL_SERVER_ERROR` | 500 | Error no previsto en el servidor. |
| `DATABASE_ERROR` | 500 | Falla en la persistencia de datos. |

---

## 4. Validaciones Globales

El backend implementa rechazo estricto de campos no permitidos (**whitelisting**):
- Si el cliente envía propiedades adicionales o no reconocidas en el cuerpo JSON, el backend responderá con `400 Bad Request` (`VALIDATION_ERROR`).
- Las cadenas de texto son sanitizadas y validadas según las siguientes reglas base:
  - **username**: Obligatorio, entre 1 y 30 caracteres, sólo letras/números/guión bajo (`^[a-zA-Z0-9_]+$`) — mismo charset exigido tanto al registrarse como al editar el perfil.
  - **email**: Formato de email válido (`user@domain.com`), máximo 30 caracteres.
  - **password**: Mínimo 6 caracteres, máximo 50 caracteres.
  - **Todo campo de texto libre que un cliente pueda renderizar** (título/descripción de obra, tags, comentario, bio, campo `comment`/`detail` de reportes y apelaciones): se le remueve cualquier marcado HTML antes de persistir — ver 4.2.

### 4.1 Rate limiting (RNF03, RNF08)

Implementado con `@nestjs/throttler`, por IP:

- **Límite global por defecto:** 100 peticiones / minuto, aplicado automáticamente a toda ruta que no tenga un override propio (red de seguridad general, incluye rutas públicas de sólo lectura).
- **Endpoints de autenticación** (`POST /auth/register`, `/auth/login`, `/auth/resend-verification`, `/auth/forgot-password`, `/auth/reset-password`, `/auth/refresh`): **5 peticiones / minuto**, cada endpoint con su propio contador independiente.
- **Escritura sensible a spam** (`POST /artworks/:id/comments`, `PUT /artworks/:id/rating`, `POST /artworks/:id/reports`, `POST /comments/:id/reports`): **10 peticiones / minuto**.
- Al exceder el límite: `429 Too Many Requests`, `code: "TOO_MANY_REQUESTS"` (mismo formato de error que el resto de la API, ver sección 2).
- El contador es por IP y en memoria del proceso (sin Redis/almacenamiento compartido) — suficiente para una única instancia; una implementación multi-instancia necesitaría un `ThrottlerStorage` compartido.

### 4.2 Sanitización de texto libre (XSS almacenado)

Todo campo de texto libre que algún cliente (web o Android) pudiera renderizar se sanitiza en el backend al validarlo, sin depender de que cada cliente escape correctamente — defensa en profundidad contra XSS almacenado, independiente de cómo cada cliente termine mostrando el contenido:

- **Campos afectados:** `title`/`description` de obras (incluida la carga directa de admin, 18.8), cada elemento de `tags`, `comment` de comentarios, `bio` de perfil, `comment` de reportes (16.1/16.2) y `detail` de apelaciones (18.3/18.4).
- **Comportamiento:** se remueve cualquier etiqueta HTML pero se conserva el texto interno (`"<b>hola</b>"` → `"hola"`); el contenido de etiquetas peligrosas por diseño (`<script>`, `<style>`) se descarta por completo en vez de dejarlo como texto plano. Se aplica **antes** de los límites de longitud (`MaxLength`), así que no es posible burlar el límite rellenando con marcado que luego se remueve.
- Los límites de longitud (`MaxLength`) documentados en cada endpoint ya reflejan el conteo sobre el texto sanitizado.
- No afecta a `username` (que ya tiene su propio charset restringido, ver 4.) ni a campos que no son texto libre (emails, enums, IDs).

### 4.3 Logging (RNF14)

Un log por request, a nivel `log` (2xx/3xx) o `warn` (4xx/5xx), con la forma `MÉTODO ruta status duración - ip`. Implementado como middleware de Express (corre antes que los guards) en vez de interceptor de Nest, justamente para no perder el registro de requests rechazadas por autenticación/roles/rate limiting (401/403/429) — el tráfico más relevante desde el punto de vista de seguridad. No es visible en la respuesta HTTP; vive sólo en los logs del proceso (stdout).

---

## 5. Autenticación y Autorización (JWT)

### 5.1 Cabecera de Autorización
Para todas las rutas protegidas, el cliente debe enviar el token en la cabecera HTTP:

```http
Authorization: Bearer <accessToken>
```

### 5.2 Estructura del Payload del JWT
El `accessToken` emitido por `/auth/login`, `/auth/register` y demás endpoints de autenticación contiene las siguientes claims:

```json
{
  "sub": 1,
  "username": "pandora_artist",
  "email": "artist@pandora.art",
  "roles": ["user"],
  "iat": 1726442400,
  "exp": 1726443300
}
```

### 5.3 Roles del Sistema
| Rol | Identificador | Descripción |
|---|---|---|
| Usuario estándar | `user` | Rol por defecto al registrarse. Habilita publicación de obras, calificación Q2Q, apertura de paquetes, colección y autobattler. |
| Moderador | `moderator` | Acceso a panel de reportes, resolución de denuncias, advertencias y moderación de contenido. |
| Administrador | `admin` | Permisos totales sobre configuración, roles y auditoría. |

### 5.4 Access Token + Refresh Token (Fase 4)
A partir de la Fase 4, **todo endpoint que autentica** (registro, login, verificación de email, login con Google y refresh) devuelve **dos** tokens en vez de uno:

```json
{
  "accessToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "refreshToken": "6177fb5b0a35c484132b9c47884e2372ea824c82c2dfc44702d31a29d8a85ef",
  "user": { "...": "..." }
}
```

| Token | Formato | Vigencia por defecto | Uso |
|---|---|---|---|
| `accessToken` | JWT | **15 minutos** | Se envía en `Authorization: Bearer <accessToken>` en cada request a una ruta protegida. |
| `refreshToken` | Cadena opaca (no es un JWT, no se puede decodificar) | **30 días** | Se guarda de forma segura en el cliente (ver nota) y se usa **sólo** contra `POST /auth/refresh` para obtener un `accessToken` nuevo sin pedir credenciales de nuevo. |

**Importante — antes de esta fase el `accessToken` duraba 7 días y no existía `refreshToken`.** Cualquier cliente que todavía asuma sesiones de larga duración con un solo token debe actualizarse: ahora hay que refrescar la sesión periódicamente (por ejemplo, al recibir un `401 UNAUTHORIZED` de una ruta protegida, o proactivamente unos minutos antes de que expire el `accessToken` decodificando su claim `exp`).

**Rotación:** cada `refreshToken` es de un solo uso. Al llamar a `/auth/refresh`, el token enviado queda invalidado y la respuesta trae un `refreshToken` nuevo — el cliente debe reemplazar el que tenía guardado por el nuevo en cada refresh. Reutilizar un `refreshToken` ya usado (o uno después de `/auth/logout`) responde `401 INVALID_TOKEN`.

**Dónde guardar el `refreshToken` en el cliente:** al ser una credencial de larga duración, no debe guardarse en `localStorage` de forma expuesta a XSS si se puede evitar; se recomienda un almacenamiento seguro (Keystore/Keychain en Android, `httpOnly` cookie o almacenamiento cifrado en web) — la implementación concreta queda a criterio de cada cliente.

#### Refrescar la sesión
- **Ruta:** `POST /api/v1/auth/refresh`
- **Acceso:** Público (no requiere `Authorization`, la validez la da el propio `refreshToken`).
- **Cuerpo:** `{ "refreshToken": "..." }`
- **Respuesta de éxito (200):** mismo formato que 5.4 (accessToken + refreshToken nuevos + user).
- **Errores:** `INVALID_TOKEN` (401, token inexistente, ya usado o revocado por logout), `TOKEN_EXPIRED` (401, token vencido — hay que iniciar sesión de nuevo), `USER_BANNED` / `USER_SUSPENDED` (403).

#### Cerrar sesión
- **Ruta:** `POST /api/v1/auth/logout`
- **Acceso:** Protegido (`Authorization: Bearer <accessToken>`)
- **Cuerpo:** `{ "refreshToken": "..." }`
- **Respuesta de éxito (200):** `{ "message": "Sesión cerrada correctamente." }` — idempotente, siempre responde igual aunque el token ya estuviera invalidado.
- Invalida únicamente el `refreshToken` enviado (la sesión de otros dispositivos, si el usuario tiene varios `refreshToken` activos, no se ve afectada). El `accessToken` vigente sigue siendo técnicamente válido hasta que expira por sí solo (máx. 15 minutos), ya que los access tokens no se revocan individualmente.

---

## 6. Contrato de Endpoints de Autenticación (Fase 1)

### 6.1 Registro Local
- **Ruta:** `POST /api/v1/auth/register`
- **Acceso:** Público
- **Cuerpo de la petición:**
```json
{
  "username": "solaris",
  "email": "solaris@pandora.art",
  "password": "miPasswordSegura123"
}
```
- **Respuesta de éxito (201 Created):**
```json
{
  "accessToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "refreshToken": "6177fb5b0a35c484132b9c47884e2372ea824c82c2dfc44702d31a29d8a85ef",
  "user": {
    "id": 1,
    "username": "solaris",
    "email": "solaris@pandora.art",
    "roles": ["user"],
    "emailVerified": false,
    "status": "pending_verification"
  }
}
```
> Ver 5.4 para el significado y manejo de `accessToken`/`refreshToken`.

### 6.2 Inicio de Sesión (Login)
- **Ruta:** `POST /api/v1/auth/login`
- **Acceso:** Público
- **Cuerpo de la petición:**
```json
{
  "email": "solaris@pandora.art",
  "password": "miPasswordSegura123"
}
```
- **Respuesta de éxito (200 OK):**
```json
{
  "accessToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "refreshToken": "6177fb5b0a35c484132b9c47884e2372ea824c82c2dfc44702d31a29d8a85ef",
  "user": {
    "id": 1,
    "username": "solaris",
    "email": "solaris@pandora.art",
    "roles": ["user"],
    "emailVerified": false,
    "status": "pending_verification"
  }
}
```

### 6.3 Perfil del Usuario Autenticado
- **Ruta:** `GET /api/v1/auth/me`
- **Acceso:** Protegido (`Authorization: Bearer <accessToken>`)
- **Respuesta de éxito (200 OK):**
```json
{
  "id": 1,
  "username": "solaris",
  "email": "solaris@pandora.art",
  "bio": null,
  "avatarUrl": null,
  "roles": ["user"],
  "emailVerified": false,
  "status": "pending_verification",
  "createdAt": "2026-09-15T20:00:00.000Z"
}
```

> Nota: al registrarse, la cuenta queda en `status: "pending_verification"` hasta completar la verificación de email (ver 7.1) o autenticarse con Google, que verifica el email automáticamente.

---

## 7. Contrato de Endpoints de Autenticación (Fase 2)

### 7.1 Verificación de Email
- **Ruta:** `POST /api/v1/auth/verify-email`
- **Acceso:** Público
- **Cuerpo de la petición:**
```json
{ "token": "b7e2...f1a9" }
```
- El `token` es el recibido por correo (parámetro `?token=` del enlace `{FRONTEND_URL}/verify-email?token=...`). **Vigencia: 24 horas, uso único.**
- **Respuesta de éxito (200 OK):**
```json
{
  "message": "Correo electrónico verificado exitosamente.",
  "accessToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "refreshToken": "6177fb5b0a35c484132b9c47884e2372ea824c82c2dfc44702d31a29d8a85ef",
  "user": {
    "id": 1,
    "username": "solaris",
    "email": "solaris@pandora.art",
    "roles": ["user"],
    "emailVerified": true,
    "status": "active"
  }
}
```
- **Errores:** `INVALID_TOKEN` (400), `TOKEN_EXPIRED` (400).

### 7.2 Reenvío de Verificación
- **Ruta:** `POST /api/v1/auth/resend-verification`
- **Acceso:** Público
- **Cuerpo de la petición:**
```json
{ "email": "solaris@pandora.art" }
```
- **Respuesta de éxito (200 OK):** mensaje neutro tanto si el email existe como si no (evita enumeración de cuentas):
```json
{ "message": "Si la dirección de correo electrónico está registrada, se ha enviado un nuevo enlace de verificación." }
```
- **Errores:** `USER_ALREADY_VERIFIED` (400) si la cuenta ya estaba verificada.

### 7.3 Login con Google — Web (redirección)
- **Ruta:** `GET /api/v1/auth/google`
- **Acceso:** Público
- Redirige al navegador al consentimiento de Google. Uso exclusivo del **portal web**.

- **Ruta:** `GET /api/v1/auth/google/callback`
- **Acceso:** Público (callback de Google)
- Tras autenticar, redirige a:
  ```text
  {FRONTEND_URL}/auth/callback?token=<accessToken>
  ```
  El frontend debe leer `token` de la query string en esa ruta y guardarlo como el `accessToken` habitual.
  Si la petición envía `Accept: application/json`, responde en su lugar el mismo JSON que 7.4 (útil sólo para pruebas manuales, no para el flujo real de navegador).
  **El `refreshToken` no viaja en la URL de redirección** (para no exponer una credencial de larga duración en el historial del navegador ni en logs) — el flujo de redirección sólo entrega un `accessToken` de corta duración. Si el portal web necesita mantener sesión más allá de esos 15 minutos sin repetir el consentimiento de Google, debe usar el flujo 7.4 (por ejemplo, vía un popup/fetch en lugar de una redirección de página completa) para recibir el `refreshToken` en el cuerpo JSON.

### 7.4 Login con Google — App / SPA (ID Token)
- **Ruta:** `POST /api/v1/auth/google`
- **Acceso:** Público
- **Uso recomendado para la app Android**: el cliente obtiene un `idToken` mediante el SDK/Credential Manager de Google y lo envía al backend, sin redirecciones.
- **Cuerpo de la petición:**
```json
{ "idToken": "eyJhbGciOiJSUzI1NiIsImtpZCI6..." }
```
- **Respuesta de éxito (200 OK):** igual forma que login/registro local:
```json
{
  "accessToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "refreshToken": "6177fb5b0a35c484132b9c47884e2372ea824c82c2dfc44702d31a29d8a85ef",
  "user": {
    "id": 2,
    "username": "solaris_google",
    "email": "solaris@gmail.com",
    "roles": ["user"],
    "emailVerified": true,
    "status": "active",
    "avatarUrl": "https://lh3.googleusercontent.com/..."
  }
}
```
- Si es la primera vez que ese email/cuenta de Google se ve, se crea un usuario nuevo automáticamente (`email_verified: true`, sin contraseña local). Si ya existía una cuenta local con ese email, se vincula y se marca el email como verificado.
- **Errores:** `OAUTH_ERROR` (401) si el `idToken` es inválido o expiró.

### 7.5 Recuperación de contraseña

- **Ruta:** `POST /api/v1/auth/forgot-password` — **Acceso:** Público.
- **Cuerpo de la petición:**
```json
{ "email": "solaris@pandora.art" }
```
- **Respuesta de éxito (200 OK):** mensaje neutro sin importar si la cuenta existe, si ya tiene contraseña verificada o no, o si es una cuenta exclusivamente de Google (evita enumeración de cuentas, mismo patrón que 7.2):
```json
{ "message": "Si la dirección de correo electrónico está registrada y tiene una contraseña configurada, se ha enviado un enlace para restablecerla." }
```
- Una cuenta creada únicamente por Google (sin contraseña local) recibe la misma respuesta neutra, pero **no** se le envía correo — no hay contraseña que recuperar, y confirmar esa distinción filtraría cómo se registró la cuenta.
- Invalida cualquier token de recuperación previo sin usar antes de emitir uno nuevo (mismo mecanismo que 7.2 con `email_verification_tokens`, tabla propia `password_reset_tokens`). **Vigencia: 1 hora** (más corta que la de verificación de email por tratarse de un token más sensible), uso único.
- **Rate limit:** 5 peticiones/minuto por IP (ver 4.1) — es el endpoint más sensible a spamear de todo `/auth/*` por costo de envío de correo y riesgo de enumeración.

- **Ruta:** `POST /api/v1/auth/reset-password` — **Acceso:** Público.
- **Cuerpo de la petición:**
```json
{ "token": "b7e2...f1a9", "newPassword": "miPasswordNueva123" }
```
- El `token` es el recibido por correo (parámetro `?token=` del enlace `{FRONTEND_URL}/reset-password?token=...`).
- **Respuesta de éxito (200 OK):**
```json
{ "message": "Tu contraseña fue actualizada correctamente." }
```
- A diferencia de 7.1 (verificar email), **no** inicia sesión automáticamente ni devuelve un par de tokens: el usuario vuelve a iniciar sesión explícitamente con la contraseña nueva.
- Efecto adicional: revoca todos los `refresh_tokens` activos del usuario — cualquier sesión abierta con la contraseña anterior deja de poder renovar su `accessToken` y debe volver a autenticarse.
- **Errores:** `INVALID_TOKEN` (400, token inexistente o ya usado — mismo código que 7.1, no uno específico), `TOKEN_EXPIRED` (400).

---

## 8. Contrato de Endpoints de Usuarios (Fase 2)

### 8.1 Perfil Público
- **Ruta:** `GET /api/v1/users/:id`
- **Acceso:** Público. Si se envía `Authorization: Bearer <accessToken>` válido, la respuesta incluye `isFollowing`.
- **Respuesta de éxito (200 OK):**
```json
{
  "id": 1,
  "username": "solaris",
  "bio": "Artista digital.",
  "avatarUrl": "https://res.cloudinary.com/.../avatar.png",
  "roles": ["user"],
  "status": "active",
  "followersCount": 12,
  "followingCount": 5,
  "isFollowing": false,
  "createdAt": "2026-09-15T20:00:00.000Z"
}
```
- `isFollowing` es `undefined`/ausente cuando la petición es anónima o cuando se consulta el propio perfil.
- **Errores:** `USER_NOT_FOUND` (404).

### 8.2 Editar Perfil Propio
- **Ruta:** `PATCH /api/v1/users/me`
- **Acceso:** Protegido
- **Cuerpo de la petición (todos los campos opcionales):**
```json
{
  "username": "nuevo_username",
  "bio": "Hasta 300 caracteres.",
  "avatar_url": "https://res.cloudinary.com/.../avatar.png"
}
```
- **Respuesta de éxito (200 OK):** el perfil actualizado, mismo formato que `GET /auth/me`.
- **Errores:** `USERNAME_ALREADY_EXISTS` (409), `VALIDATION_ERROR` (400).

### 8.3 Seguir / Dejar de Seguir
- **Rutas:** `POST /api/v1/users/:id/follow`, `DELETE /api/v1/users/:id/follow`
- **Acceso:** Protegido
- Operaciones **idempotentes**: seguir dos veces o dejar de seguir sin estar siguiendo no generan error, sólo devuelven un mensaje informativo.
- **Errores:** `CANNOT_FOLLOW_SELF` (400), `USER_NOT_FOUND` (404), `USER_BANNED` (403, no se puede seguir a una cuenta baneada).

### 8.4 Listados de Seguidores / Seguidos
- **Rutas:**
  - `GET /api/v1/users/me/followers`, `GET /api/v1/users/me/following` (protegidas)
  - `GET /api/v1/users/:id/followers`, `GET /api/v1/users/:id/following` (públicas)
- **Query params (paginación estándar, ver 8.5):** `page` (default 1), `limit` (default 20, máx. 100)
- **Respuesta de éxito (200 OK):**
```json
{
  "items": [
    {
      "id": 3,
      "username": "otroArtista",
      "bio": null,
      "avatarUrl": null,
      "followedAt": "2026-09-16T10:00:00.000Z"
    }
  ],
  "total": 1,
  "page": 1,
  "limit": 20,
  "totalPages": 1
}
```

### 8.5 Paginación Estándar
A partir de la Fase 2, los endpoints con listados paginados aceptan `page` y `limit` como query params y devuelven el envoltorio `{ items, total, page, limit, totalPages }`. Este es el formato que deben esperar el portal web y la app móvil para toda nueva colección paginada (galería de obras, notificaciones, reportes, etc. en fases posteriores).

---

## 9. Contrato de Endpoints de Obras (Fase 3)

### 9.1 Subir Obra
- **Ruta:** `POST /api/v1/artworks`
- **Acceso:** Protegido + **cuenta con email verificado** (ver 9.6)
- **Content-Type:** `multipart/form-data` (no JSON). Campos:

| Campo | Tipo | Obligatorio | Notas |
|---|---|---|---|
| `title` | string | Sí | 1–50 caracteres |
| `description` | string | No | Hasta 300 caracteres (el "Lore") |
| `image` | file | Sí | PNG/JPG/JPEG/WEBP, máx. 25 MB |
| `conversionRequest` | `"true"` \| `"false"` | No | Default `true` si se omite |
| `tags` | string | No | Lista de tags. Acepta un JSON array serializado (`'["fantasy","paisaje"]'`) o una lista separada por comas (`"fantasy,paisaje"`) |

- Los tags se normalizan a minúsculas y se crean automáticamente si no existen (no hay endpoint de administración de tags en esta fase).
- **Respuesta de éxito (201):** ver forma completa en 9.3 (Detalle de Obra).
- **Errores:** `VALIDATION_ERROR` (400, incluye imagen faltante/formato inválido/demasiado grande), `USER_NOT_VERIFIED` (403, ver 9.6).

### 9.2 Editar / Eliminar Obra
- **Ruta:** `PATCH /api/v1/artworks/:id` — **Acceso:** Protegido, sólo el autor.
  - Mismo `multipart/form-data` que la creación, pero **todos los campos son opcionales**. Omitir `image` conserva la imagen actual; si se envía una nueva, la anterior se elimina de Cloudinary. Omitir `tags` conserva los tags actuales; enviar `tags` reemplaza la lista completa (enviar una lista vacía la limpia).
  - **Errores:** `FORBIDDEN` (403, no es el autor), `NOT_FOUND` (404).
- **Ruta:** `DELETE /api/v1/artworks/:id` — **Acceso:** Protegido, autor **o** moderador/admin. Eliminación lógica (la obra deja de listarse pero no se borra físicamente).
  - **Errores:** `FORBIDDEN` (403), `NOT_FOUND` (404).

### 9.3 Detalle de Obra
- **Ruta:** `GET /api/v1/artworks/:id` — **Acceso:** Público.
- **Respuesta de éxito (200 OK):**
```json
{
  "id": 1,
  "title": "Atardecer en la niebla",
  "description": "Lore opcional de la obra.",
  "author": { "id": 1, "username": "solaris", "avatarUrl": null },
  "image": {
    "original": "https://res.cloudinary.com/.../original.png",
    "medium": "https://res.cloudinary.com/.../medium.png",
    "thumbnail": "https://res.cloudinary.com/.../thumb.png"
  },
  "tags": ["fantasy", "paisaje"],
  "conversionRequest": true,
  "conversionStatus": "pending",
  "qualification": {
    "startedAt": "2026-09-16T20:00:00.000Z",
    "endsAt": "2026-09-23T20:00:00.000Z",
    "active": true
  },
  "createdAt": "2026-09-16T20:00:00.000Z",
  "updatedAt": null,
  "isFavorited": true
}
```
- `conversionStatus` es uno de: `pending`, `processing`, `converted`, `skipped`, `failed` (la conversión a carta en sí llega en la Fase 5; por ahora toda obra nueva queda en `pending`).
- `qualification.active` indica si la obra todavía admite valoraciones Q2Q (el sistema de ratings llega en la Fase 4; por ahora es informativo).
- `isFavorited`: sólo presente con un token válido — indica si el usuario autenticado ya destacó esta obra (mismo patrón que `isFollowing` en 8.1). Ausente para peticiones anónimas.
- **Errores:** `NOT_FOUND` (404, obra inexistente o eliminada).

### 9.4 Explorador de Obras
- **Ruta:** `GET /api/v1/artworks` — **Acceso:** Público.
- **Query params (todos opcionales):**

| Param | Tipo | Descripción |
|---|---|---|
| `page`, `limit` | number | Paginación estándar (ver 8.5) |
| `search` | string | Búsqueda parcial (case-insensitive) en `title` y `description` |
| `authorId` | number | Filtra por autor |
| `tag` | string | Filtra por un tag exacto (normalizado a minúsculas) |
| `from`, `until` | string (ISO 8601) | Rango de `createdAt` |
| `converted` | boolean | `true` = sólo obras con carta generada; `false` = el resto |
| `qualificationOnly` | boolean | `true` = sólo obras aún dentro de su semana de calificación |
| `sort` | `recent` \| `oldest` \| `title` | Orden del listado. Default `recent` |
| `rarity` | `common`\|`uncommon`\|`rare`\|`epic`\|`legendary` | Filtra por la rareza de la carta ya generada para la obra. Una obra sin carta convertida de esa rareza no aparece — no existe un valor "sin rareza" |

- **Respuesta:** envoltorio de paginación estándar (8.5) con `items` en el mismo formato que 9.3.

### 9.5 Obras en Período de Calificación
- **Ruta:** `GET /api/v1/artworks/qualification` — **Acceso:** Público.
- Atajo equivalente a `GET /artworks?qualificationOnly=true`, con los mismos query params de paginación/orden/filtro (excepto `qualificationOnly`, que queda fijo en `true`).

### 9.6 Obras Propias
- **Ruta:** `GET /api/v1/users/me/artworks` — **Acceso:** Protegido.
- Lista paginada (formato 8.5) de las obras del usuario autenticado, mismo formato de item que 9.3.

### 9.7 Requisito de Email Verificado
A partir de esta fase, **crear una obra** y **comentar una obra** (ver 9.9) requieren, además del JWT, que la cuenta tenga el email verificado. Si no lo está, la API responde:
```json
{
  "statusCode": 403,
  "code": "USER_NOT_VERIFIED",
  "message": "Debes verificar tu correo electrónico antes de realizar esta acción."
}
```
El resto de las acciones de escritura sobre obras (editar, eliminar) **no** exigen este requisito, sólo sesión + ownership.

---

## 10. Contrato de Endpoints de Tags (Fase 3)

- **Ruta:** `GET /api/v1/tags` — **Acceso:** Público.
- **Query params (opcional):** `search` (string) — filtra por nombre (`ILIKE`, insensible a mayúsculas), server-side.
- **Respuesta de éxito (200 OK):** sigue siendo un array plano, sin envoltorio de paginación (el catálogo de tags es chico) — `search` reduce el array devuelto en vez de cambiar su forma:
```json
[
  { "id": 1, "name": "fantasy" },
  { "id": 2, "name": "paisaje" }
]
```
- No hay endpoints de creación/edición/borrado manual de tags en esta fase: se crean dinámicamente al publicar o editar una obra (ver 9.1).

---

## 11. Contrato de Endpoints de Comentarios (Fase 3)

Los comentarios están anidados bajo la obra a la que pertenecen; no existe un `/comments` de nivel superior.

### 11.1 Listar Comentarios de una Obra
- **Ruta:** `GET /api/v1/artworks/:id/comments` — **Acceso:** Público.
- **Query params:** paginación estándar (`page`, `limit`, ver 8.5).
- **Respuesta:** envoltorio de paginación estándar con `items`:
```json
{
  "items": [
    {
      "id": 10,
      "comment": "¡Qué obra tan increíble!",
      "author": { "id": 2, "username": "otroArtista", "avatarUrl": null },
      "createdAt": "2026-09-16T21:00:00.000Z",
      "updatedAt": null
    }
  ],
  "total": 1,
  "page": 1,
  "limit": 20,
  "totalPages": 1
}
```

### 11.2 Comentar una Obra
- **Ruta:** `POST /api/v1/artworks/:id/comments` — **Acceso:** Protegido + email verificado (ver 9.7).
- **Cuerpo de la petición:**
```json
{ "comment": "¡Qué obra tan increíble!" }
```
- `comment`: obligatorio después de recortar espacios, máximo 300 caracteres.
- Las obras permanecen comentables aunque su período de calificación Q2Q ya haya terminado.
- **Respuesta de éxito (201 Created):** mismo formato que un ítem de 11.1.
- **Errores:** `VALIDATION_ERROR` (400), `NOT_FOUND` (404, obra inexistente/eliminada), `USER_NOT_VERIFIED` (403).

> No existe todavía edición ni borrado de comentarios por parte del propio usuario; la eliminación de comentarios llega junto con el panel de moderación (Fase 8).

---

## 12. Contrato de Endpoints de Calificaciones Q2Q (Fase 4)

Igual que los comentarios, las calificaciones están anidadas bajo la obra; no existe un `/ratings` de nivel superior.

### 12.1 Calificar (crear o actualizar)
- **Ruta:** `PUT /api/v1/artworks/:id/rating` — **Acceso:** Protegido + **cuenta con email verificado** (ver 9.7).
- **Cuerpo de la petición:**
```json
{ "stars": 5, "emotion": "joy" }
```

| Campo | Tipo | Reglas |
|---|---|---|
| `stars` | integer | 1 a 5 |
| `emotion` | string | Uno de: `joy`, `fear`, `anger`, `serenity`, `grief`, `awe` |

- Es una operación **upsert**: si el usuario ya había calificado esta obra, su valoración se actualiza (no se crea una segunda fila, no hay error de duplicado).
- Sólo funciona mientras la obra está dentro de su semana de calificación (ver `qualification.active` en el detalle de obra, 9.3, o en 12.3).
- **Respuesta de éxito (200 OK):**
```json
{
  "stars": 5,
  "emotion": "joy",
  "createdAt": "2026-09-16T21:00:00.000Z",
  "updatedAt": "2026-09-16T21:00:00.000Z"
}
```
- **Errores:** `VALIDATION_ERROR` (400), `NOT_FOUND` (404, obra inexistente/eliminada), `QUALIFICATION_ENDED` (403, el período ya cerró), `USER_NOT_VERIFIED` (403).

### 12.2 Retirar Calificación
- **Ruta:** `DELETE /api/v1/artworks/:id/rating` — **Acceso:** Protegido (no requiere email verificado).
- Idempotente: si el usuario no había calificado la obra, no es un error.
- **Respuesta de éxito (200 OK):** `{ "message": "Se retiró tu valoración de la obra." }`
- **Errores:** `NOT_FOUND` (404), `QUALIFICATION_ENDED` (403 — al igual que crear/actualizar, retirar un voto sólo se permite mientras el período sigue activo; ver `docs/fase-4-q2q.md` para el razonamiento de esta decisión).

### 12.3 Resumen de Calificaciones
- **Ruta:** `GET /api/v1/artworks/:id/ratings` — **Acceso:** Público. Si se envía `Authorization: Bearer <accessToken>` válido, la respuesta incluye `myRating`.
- **Respuesta de éxito (200 OK):**
```json
{
  "artworkId": 1,
  "totalVotes": 37,
  "averageStars": 4.3,
  "qualification": {
    "startedAt": "2026-09-16T20:00:00.000Z",
    "endsAt": "2026-09-23T20:00:00.000Z",
    "active": true
  },
  "myRating": {
    "stars": 5,
    "emotion": "joy",
    "createdAt": "2026-09-16T21:00:00.000Z",
    "updatedAt": "2026-09-16T21:00:00.000Z"
  }
}
```
- `myRating` es `null` cuando la petición es anónima o el usuario autenticado todavía no calificó esta obra.
- `averageStars` es `null` cuando `totalVotes` es `0`.
- **No se expone** el desglose de votos por emoción — sólo el total y el promedio de estrellas. Ver `docs/fase-4-q2q.md` §1.4 para el razonamiento (evitar exponer datos que permitan manipular el futuro cálculo de estadísticas de la carta, Fase 5).
- **Errores:** `NOT_FOUND` (404, obra inexistente/eliminada).

---

## 13. Contrato de Endpoints de Cartas (Fase 5)

Las cartas se generan automáticamente cuando el Scheduler convierte una obra elegible (ver `docs/fase-5-cartas.md`); no hay ningún endpoint para crearlas manualmente.

### 13.1 Forma Común de una Carta
Tanto el explorador público como la colección propia devuelven cartas con esta misma forma:
```json
{
  "id": 1,
  "artwork": {
    "id": 10,
    "title": "Retrato legendario",
    "thumbnailUrl": "https://res.cloudinary.com/.../thumb.png",
    "authorId": 3
  },
  "rarity": "legendary",
  "owned": true,
  "copies": 2,
  "stats": {
    "attack": 140,
    "defense": 140,
    "hp": 2800,
    "speed": 14
  },
  "createdAt": "2026-09-17T21:03:46.438Z",
  "isFavorited": false
}
```
- `rarity` es uno de: `common`, `uncommon`, `rare`, `epic`, `legendary`.
- **`stats` es `null` cuando `owned` es `false`** (RG 7.5.5: el detalle de una carta no poseída oculta sus estadísticas). `owned`/`copies` reflejan siempre al usuario que hace la petición: `copies: 0` cuando no se posee, y ambos campos son "neutros" (`owned: false, copies: 0`) en una petición anónima.
- `isFavorited`: sólo presente con un token válido (mismo patrón que en 9.3) — ausente en una petición anónima. Sólo aparece en el detalle (13.3/13.5), no en los listados (13.2/13.4).

### 13.2 Explorador de Cartas
- **Ruta:** `GET /api/v1/cards` — **Acceso:** Público. Si se envía `Authorization: Bearer <accessToken>` válido, `owned`/`copies`/`stats` reflejan la posesión real del usuario.
- **Query params:** paginación estándar (`page`, `limit`, ver 8.5) + `rarity` (opcional, uno de los valores de 13.1) + `owned` (opcional, boolean — ver 13.4).
- **Respuesta:** envoltorio de paginación estándar con `items` en el formato de 13.1.

### 13.3 Detalle de Carta
- **Ruta:** `GET /api/v1/cards/:id` — **Acceso:** Público (igual que 13.2, enriquecido si autenticado).
- **Respuesta de éxito (200 OK):** un ítem con el formato de 13.1.
- **Errores:** `NOT_FOUND` (404).

### 13.4 Colección Propia
- **Ruta:** `GET /api/v1/users/me/cards` — **Acceso:** Protegido.
- Devuelve el **catálogo completo** de cartas del juego (no sólo las obtenidas), cada una anotada con las copias del usuario autenticado — así se pueden mostrar también las "cartas aún no obtenidas" (RF13) en una vista tipo álbum.
- **Query params:** paginación estándar + `owned` (opcional, boolean). `owned=true` filtra a sólo las cartas que el usuario posee (`copies > 0`) — sin necesidad de traer el catálogo completo para filtrar en el cliente. Sin token, `owned=true` devuelve una lista vacía (no tiene sentido "poseer" nada anónimamente).
- **Respuesta:** envoltorio de paginación estándar con `items` en el formato de 13.1.

### 13.5 Detalle de Carta Propia
- **Ruta:** `GET /api/v1/users/me/cards/:id` — **Acceso:** Protegido.
- Mismo formato que 13.3, pero siempre evaluado desde la perspectiva del usuario autenticado (no hace falta enviar nada adicional). Funciona igual para una carta no poseída (no da 404; simplemente `owned: false, stats: null`).
- **Errores:** `NOT_FOUND` (404, la carta en sí no existe).

> **Nota:** hasta que exista el sistema de paquetes (Fase 6), ningún usuario posee cartas — `GET /users/me/cards` devolverá todo el catálogo con `owned: false` para todos los ítems. No es un error.

---

## 14. Contrato de Endpoints de Paquetes (Fase 6)

Cada usuario tiene un inventario de paquetes (0 a 10) que se regenera solo, a razón de 1 paquete por minuto (ver `docs/fase-6-paquetes.md` para el detalle de la fórmula). No hace falta ningún endpoint para "crear" el inventario: se crea automáticamente (vacío) en la primera consulta.

### 14.1 Consultar Inventario de Paquetes
- **Ruta:** `GET /api/v1/users/me/packages` — **Acceso:** Protegido.
- Aplica y persiste la regeneración pendiente antes de responder (el valor devuelto es siempre el real "a este instante", no uno desactualizado).
- **Respuesta de éxito (200 OK):**
```json
{
  "amount": 4,
  "capacity": 10,
  "lastRegenerationAt": "2026-09-17T21:00:00.000Z",
  "secondsUntilNextPack": 37
}
```
- `secondsUntilNextPack` es `null` cuando el inventario está al máximo (no hay "próximo paquete" que esperar).

### 14.2 Abrir un Paquete
- **Ruta:** `POST /api/v1/users/me/packages/open` — **Acceso:** Protegido.
- Operación atómica (RG 7.6, CU08): consume exactamente un paquete y otorga 5 cartas en una única transacción — si algo falla, ni el paquete se descuenta ni se otorga ninguna carta.
- Protegida contra doble consumo concurrente (CU08-A2): ante dos peticiones simultáneas del mismo usuario, sólo prosperan tantas como paquetes disponibles haya.
- **Respuesta de éxito (200 OK):**
```json
{
  "packOpened": true,
  "remainingPackages": 3,
  "cardsObtained": [
    { "id": 12, "artwork": { "...": "..." }, "rarity": "common", "owned": true, "copies": 3, "stats": { "...": "..." }, "createdAt": "..." }
  ]
}
```
  Cada elemento de `cardsObtained` usa la misma forma que 13.1 — si el paquete otorgó cartas repetidas entre sí, `copies` ya refleja el total acumulado en la colección tras la apertura (no sólo las obtenidas en este paquete).
- **Errores:** `NO_PACKAGES_AVAILABLE` (409, CU08-A1 — no hay paquetes disponibles), `NO_CARDS_AVAILABLE` (409 — aún no existe ninguna carta en el sistema; ver nota de diseño en `docs/fase-6-paquetes.md`).

---

## 15. Contrato de Endpoints de Batalla (Fase 7)

RG 7.8 / CU09: la batalla es enteramente local en el cliente. El backend sólo resuelve la selección del rival; no existen endpoints para iniciar partidas, guardar resultados o consultar historial.

### 15.1 Obtener Rival
- **Ruta:** `GET /api/v1/battle/opponent?cardId=<id>` — **Acceso:** Protegido.
- `cardId` (query, requerido): la carta propia que el usuario eligió para la batalla.
- **Respuesta de éxito (200 OK):**
```json
{
  "opponent": {
    "id": 7,
    "artwork": { "id": 20, "title": "...", "thumbnailUrl": "...", "authorId": 9 },
    "rarity": "epic",
    "stats": { "attack": 90, "defense": 60, "hp": 1200, "speed": 40 }
  }
}
```
- A diferencia de las cartas de la colección (13.1), la carta rival **siempre** incluye `stats` completos — es necesaria para que el cliente simule la batalla, y no representa la colección privada de otro jugador (es una carta cualquiera del catálogo, no la de un rival humano real).
- **Errores:** `NOT_FOUND` (404, CU09-A2 — la carta indicada no existe), `CARD_NOT_OWNED` (403, CU09-A1 — el usuario no posee la carta elegida), `NO_OPPONENT_AVAILABLE` (409, CU09-A3 — no existe ninguna otra carta en el sistema que pueda actuar como rival).

---

## 16. Contrato de Endpoints de Moderación y Notificaciones (Fase 8)

### 16.1 Reportar una Obra (CU12)
- **Ruta:** `POST /api/v1/artworks/:id/reports` — **Acceso:** Protegido + email verificado.
- **Body:**
```json
{ "reason": "spam", "comment": "Texto opcional, máx. 300 caracteres" }
```
  `reason` es una de: `content_inappropriate`, `content_violent`, `content_sexual`, `illicit`, `copyright`, `plagiarism`, `spam`, `harassment`, `other`.
- **Respuesta de éxito (201):** el reporte creado, con `status: "pending"`.
- **Errores:** `NOT_FOUND` (404, CU12-A2), `CONTENT_ALREADY_REMOVED` (409, CU12-A3 — la obra ya fue eliminada), `REPORT_ALREADY_PENDING` (409, CU12-A4 — ya existe un reporte pendiente del mismo usuario sobre esta obra).

### 16.2 Reportar un Comentario (CU13)
- **Ruta:** `POST /api/v1/comments/:id/reports` — **Acceso:** Protegido + email verificado.
- Mismo body y misma forma de respuesta que 16.1.
- **Errores:** `NOT_FOUND` (404), `CONTENT_ALREADY_REMOVED` (409, CU13-A1), `REPORT_ALREADY_PENDING` (409, CU13-A3).

### 16.3 Listar Reportes (CU14)
- **Ruta:** `GET /api/v1/moderation/reports` — **Acceso:** Protegido, rol `moderator` o `admin`.
- **Query params:** paginación estándar (`page`, `limit`) + `status` opcional (`pending` | `in_review` | `resolved` | `rejected`).
- **Respuesta:** envoltorio de paginación estándar; cada ítem trae `targetType` (`artwork` | `comment`) además de los campos propios del reporte — necesario para saber qué pasarle a 16.4/16.5.
- **Errores:** `FORBIDDEN` (403) si el usuario no es moderador/admin.

### 16.4 Detalle de un Reporte (CU14)
- **Ruta:** `GET /api/v1/moderation/reports/:id?targetType=artwork|comment` — **Acceso:** igual que 16.3.
- `targetType` es **requerido**: los IDs de `artwork_reports` y `comment_reports` son independientes entre sí (ver `docs/fase-8-moderacion.md` sección 2.1), así que el mismo número puede referirse a dos reportes distintos según la tabla.
- **Respuesta de éxito (200):** el reporte más un objeto `target` con el contexto del contenido denunciado:
```json
{
  "id": 1,
  "targetType": "artwork",
  "targetId": 5,
  "reporterId": 8,
  "reason": "spam",
  "comment": "parece spam",
  "status": "pending",
  "resolvedBy": null,
  "resolution": null,
  "createdAt": "2026-09-18T00:00:00.000Z",
  "resolvedAt": null,
  "target": { "id": 5, "title": "...", "thumbnailUrl": "...", "authorId": 7, "deleted": false }
}
```
  `target` para un reporte de comentario trae `{ id, comment, authorId, artworkId, deleted }` en su lugar.
- **Errores:** `NOT_FOUND` (404).

### 16.5 Resolver un Reporte (CU14)
- **Ruta:** `POST /api/v1/moderation/reports/:id/resolve` — **Acceso:** igual que 16.3.
- **Body:**
```json
{ "targetType": "artwork", "action": "warning" }
```
  `action` es una de: `dismissed`, `warning`, `content_removed`, `comment_removed`, `user_banned`. `content_removed` sólo es válida con `targetType: "artwork"`; `comment_removed` sólo con `targetType: "comment"`.
- **Efecto según `action`:** ver `docs/fase-8-moderacion.md` sección 3 para el detalle completo (qué se elimina, quién recibe qué notificación). El reportante **siempre** recibe una notificación `report_resolved`, sin importar la acción elegida.
- **Respuesta de éxito (200):** el reporte actualizado (`status: "resolved"`).
- **Errores:** `INVALID_REPORT_ACTION` (400 — la acción no corresponde al tipo de contenido), `NOT_FOUND` (404), `REPORT_ALREADY_RESOLVED` (409, CU14-A4 — otro moderador ya lo resolvió).

### 16.6 Notificaciones (CU15)

Forma común de una notificación:
```json
{
  "id": 1,
  "type": "user_follow",
  "title": "Nuevo seguidor",
  "content": "alguien comenzó a seguirte.",
  "isRead": false,
  "createdAt": "2026-09-18T00:00:00.000Z"
}
```
`type` es una de: `user_follow`, `follow_upload`, `qualification_ended`, `artwork_converted`, `artwork_comment`, `artwork_vote_summary`, `comment_removed`, `artwork_removed`, `user_warned`, `user_banned`, `report_resolved`.

**`data` (opcional)**: referencia al recurso afectado, para que el cliente arme un link directo sin parsear el texto de la notificación:
```json
{ "targetType": "artwork", "targetId": 26 }
```
Sólo presente en `artwork_removed`, `comment_removed`, `artwork_restored`, `comment_restored` (`targetType: "artwork"` o `"comment"`, `targetId` = id de la obra/comentario) y `appeal_resolved` (`targetType: "appeal"`, `targetId` = id de la apelación — consultable vía `GET /users/me/appeals`, sección 18.5). Ausente en el resto de los tipos, incluido `artwork_vote_summary` (que usa su propio mecanismo de apilado, no relacionado).

**`artwork_vote_summary` es apilable**: cada voto sobre una obra notifica a su autor, pero mientras la notificación siga sin leerse, un nuevo voto **actualiza la misma fila** en vez de crear una nueva — el `content` se reescribe para nombrar a todos los votantes acumulados ("ana ha votado tu obra…" → "ana y beto han votado tu obra…" → "ana, beto y 2 más han votado tu obra…") y su `createdAt` se adelanta, así vuelve a aparecer arriba del listado. Una vez leída, el siguiente voto abre una notificación nueva. Ver `docs/mejoras-post-fase-8.md` sección 6.

- **`GET /api/v1/notifications`** — Protegido. Paginación estándar. Devuelve las notificaciones del usuario autenticado, más recientes primero.
- **`GET /api/v1/notifications/unread-count`** — Protegido. Respuesta: `{ "count": 3 }`.
- **`PATCH /api/v1/notifications/:id/read`** — Protegido. Marca una notificación propia como leída (idempotente). **Errores:** `NOTIFICATION_NOT_FOUND` (404, incluye el caso de una notificación de otro usuario).
- **`POST /api/v1/notifications/read-all`** — Protegido. Respuesta: `{ "updated": 4 }` (cantidad de notificaciones que pasaron de no leídas a leídas).

No existe ningún endpoint para crear notificaciones manualmente: siempre son un efecto secundario de otra acción (seguir a alguien, publicar una obra, comentar, que termine una calificación, que se resuelva un reporte, etc.).

---

## 17. Contrato de Endpoints de Favoritos (CU11)

RG CU11: sólo se puede destacar una obra propia o una carta que se posee — no es un "me gusta" general sobre contenido ajeno (ver `docs/mejoras-post-fase-8.md` sección 4).

### 17.1 Destacar/quitar destacado de una obra
- **`POST /api/v1/artworks/:id/favorite`** — Protegido. Idempotente (CU11-A3). **Errores:** `NOT_FOUND` (404), `FORBIDDEN` (403, CU11-A1 — la obra no es propia).
- **`DELETE /api/v1/artworks/:id/favorite`** — Protegido. No falla si no estaba destacada.

### 17.2 Destacar/quitar destacado de una carta
- **`POST /api/v1/cards/:id/favorite`** — Protegido. Idempotente. **Errores:** `NOT_FOUND` (404), `CARD_NOT_OWNED` (403, CU11-A2 — no se posee la carta).
- **`DELETE /api/v1/cards/:id/favorite`** — Protegido. No falla si no estaba destacada.

### 17.3 Listar obras destacadas
- **`GET /api/v1/users/me/favorites/artworks`** — Protegido. Paginación estándar. Cada ítem usa la misma forma de obra que el resto de la API (ver 9.3).

### 17.4 Listar cartas destacadas
- **`GET /api/v1/users/me/favorites/cards`** — Protegido. Paginación estándar. Cada ítem usa la misma forma de carta que 13.1 — dado que sólo se pueden destacar cartas poseídas, `owned` es siempre `true` aquí.

---

## 18. Contrato de Endpoints de Administración y Apelaciones

Ver `docs/mejoras-admin-apelaciones.md` para el razonamiento detrás de estas decisiones (por qué una única tabla de apelaciones, por qué un usuario baneado sí puede autenticarse, por qué "reabrir" es una reversión de un solo sentido, etc.).

### 18.1 Gestión de moderadores (Admin)
- **`POST /api/v1/roles/moderators/:userId`** — Protegido, rol `admin`. Otorga el rol `moderator` al usuario indicado. Idempotente. **Errores:** `USER_NOT_FOUND` (404).
- **`DELETE /api/v1/roles/moderators/:userId`** — Protegido, rol `admin`. Revoca el rol `moderator`. Idempotente.

Un admin **no** puede otorgar/revocar el rol `admin` mismo por esta vía — sólo `moderator`.

### 18.2 Reapertura directa de un reporte resuelto (Admin)
- **`POST /api/v1/moderation/reports/:id/reopen`** — Protegido, rol **`admin`** (a diferencia del resto de `/moderation/*`, que acepta `moderator` o `admin`).
- **Body:** `{ "targetType": "artwork" | "comment" }` (mismo motivo que en el resto de `/moderation/reports/*`: los IDs de `artwork_reports`/`comment_reports` no son globalmente únicos).
- **Efecto:** reversión de un solo sentido — restaura la obra/comentario o desbanea al usuario según la `resolution` original, y marca el reporte como `status: "overturned"`. No vuelve a poner el reporte en el flujo normal de resolución ni permite elegir una sanción distinta en el mismo paso.
- **Respuesta de éxito (200):** el reporte actualizado.
- **Errores:** `NOT_FOUND` (404), `REPORT_NOT_OVERTURNABLE` (409 si el reporte no está `resolved` o ya fue revertido; 400 si su `resolution` es `dismissed`/`warning` — no hay nada que revertir).

### 18.3 Apelar una obra eliminada
- **`POST /api/v1/artworks/:id/appeals`** — Protegido. Sólo el autor de la obra puede apelar su propia eliminación.
- **Body:**
```json
{ "reason": "excessive_punishment", "detail": "Texto opcional, máx. 500 caracteres" }
```
  `reason` es una de: `excessive_punishment`, `mistaken_identity`, `missing_context`, `false_report`, `other`.
- **Respuesta de éxito (201):** la apelación creada, con `status: "pending"`.
- **Errores:** `NOT_FOUND` (404 — la obra no existe, o no hay ninguna sanción apelable: no fue eliminada por un reporte resuelto con `content_removed`), `FORBIDDEN` (403 — no sos el autor), `APPEAL_ALREADY_EXISTS` (409 — esa sanción ya fue apelada).

### 18.4 Apelar un comentario eliminado
- **`POST /api/v1/comments/:id/appeals`** — Protegido. Mismo contrato que 18.3, para el autor del comentario.

### 18.5 Apelar el propio baneo
- **`POST /api/v1/users/me/ban-appeal`** — Protegido, **sin `BannedUserGuard`**: es la vía por la que una cuenta baneada recupera acceso. Mismo body/respuesta que 18.3. Localiza automáticamente el reporte que causó el baneo — no hace falta indicar cuál.
- **`GET /api/v1/users/me/appeals`** — Protegido, sin `BannedUserGuard`. Paginación estándar. Lista las apelaciones propias (de cualquier tipo: obra, comentario o baneo), para poder consultar su estado sin acceso de moderación.

### 18.6 Panel de moderación de apelaciones
- **`GET /api/v1/moderation/appeals`** — Protegido, rol `moderator`/`admin`. Paginación estándar + `status` opcional. **Excluye automáticamente** las apelaciones cuyo reporte original resolvió el propio moderador que hace la consulta — salvo que sea el único miembro de moderación activo (`moderator` + `admin` combinados).
- **`GET /api/v1/moderation/appeals/:id`** — Protegido, rol `moderator`/`admin`. Incluye el reporte original referenciado (`report`).
- **`POST /api/v1/moderation/appeals/:id/resolve`** — Protegido, rol `moderator`/`admin`.
  - **Body:** `{ "action": "approve" | "reject" }`.
  - **Aprobar:** aplica la misma reversión de 18.2 (restaura contenido / desbanea) y marca el reporte original como `overturned`. **Rechazar:** no modifica nada del reporte.
  - En ambos casos se notifica al apelante (`appeal_resolved`).
  - **Respuesta de éxito (200):** la apelación actualizada.
  - **Errores:** `NOT_FOUND` (404), `APPEAL_REVIEWER_CONFLICT` (403 — sos el mismo moderador que resolvió el reporte original, y hay más de un moderador/admin), `APPEAL_ALREADY_RESOLVED` (409 — ya fue resuelta).

### 18.7 Notificaciones nuevas
`type` gana cuatro valores: `artwork_restored`, `comment_restored`, `user_unbanned` (al afectado, cuando una sanción se revierte por cualquiera de los dos caminos de arriba) y `appeal_resolved` (al apelante, siempre, sin importar el resultado).

### 18.8 Carga directa de carta (Admin)
- **Ruta:** `POST /api/v1/artworks/admin-upload` — **Acceso:** Protegido, rol **`admin`**.
- **Content-Type:** `multipart/form-data`, mismos campos de imagen/título/descripción/tags que 9.1 (`POST /artworks`), más los datos de la carta:

| Campo | Tipo | Obligatorio | Notas |
|---|---|---|---|
| `title` | string | Sí | 1–50 caracteres |
| `description` | string | No | Hasta 300 caracteres |
| `image` | file | Sí | PNG/JPG/JPEG/WEBP, máx. 25 MB |
| `tags` | string | No | Mismo formato que 9.1 |
| `rarity` | string | Sí | Una de `common`, `uncommon`, `rare`, `epic`, `legendary` |
| `attack`, `defense`, `hp`, `speed` | number | No | Enteros ≥ 0. Cualquiera que se omita se completa con el mismo cálculo por rareza que usa el catálogo sembrado (`computeStatsForRarity`), campo por campo — no es todo o nada |

- La obra se crea directamente con `conversionStatus: "converted"` (no pasa por el período de calificación real) y `conversionRequest: false`; la carta se crea en el mismo paso. El autor de la obra es el admin que hizo la carga.
- A diferencia de 9.1, no notifica a los seguidores del admin (no es un upload orgánico).
- **Respuesta de éxito (201):** la carta creada, misma forma que 13.1 (Forma Común de una Carta) — `stats` viaja en `null` salvo que el admin ya posea una copia (no ocurre automáticamente al crearla).
- **Errores:** `VALIDATION_ERROR` (400), `FORBIDDEN` (403, no es admin).

### 18.9 Disparo manual del ciclo de conversión (Admin)
- **Ruta:** `POST /api/v1/conversion/run` — **Acceso:** Protegido, rol **`admin`**.
- Ejecuta inmediatamente el mismo ciclo que corre el cron diario (RG 16.5): convierte en carta toda obra cuyo período de calificación ya terminó. Pensado para demos, para no tener que esperar al reloj real.
- **Respuesta de éxito (200):** `{ "processed": number, "converted": number, "failed": number, "skipped": number }`.
- **Errores:** `FORBIDDEN` (403, no es admin).
