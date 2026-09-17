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
| `TOO_MANY_REQUESTS` | 429 | Límite de peticiones excedido (rate limiting). |
| `INTERNAL_SERVER_ERROR` | 500 | Error no previsto en el servidor. |
| `DATABASE_ERROR` | 500 | Falla en la persistencia de datos. |

---

## 4. Validaciones Globales

El backend implementa rechazo estricto de campos no permitidos (**whitelisting**):
- Si el cliente envía propiedades adicionales o no reconocidas en el cuerpo JSON, el backend responderá con `400 Bad Request` (`VALIDATION_ERROR`).
- Las cadenas de texto son sanitizadas y validadas según las siguientes reglas base:
  - **username**: Obligatorio, entre 1 y 30 caracteres.
  - **email**: Formato de email válido (`user@domain.com`), máximo 30 caracteres.
  - **password**: Mínimo 6 caracteres, máximo 50 caracteres.

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
  "updatedAt": null
}
```
- `conversionStatus` es uno de: `pending`, `processing`, `converted`, `skipped`, `failed` (la conversión a carta en sí llega en la Fase 5; por ahora toda obra nueva queda en `pending`).
- `qualification.active` indica si la obra todavía admite valoraciones Q2Q (el sistema de ratings llega en la Fase 4; por ahora es informativo).
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

- **Respuesta:** envoltorio de paginación estándar (8.5) con `items` en el mismo formato que 9.3.
- **Nota:** el filtro por `rarity` mencionado en el diseño original queda pendiente hasta que exista el modelo de Cartas (Fase 5).

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
- **Respuesta de éxito (200 OK):**
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
