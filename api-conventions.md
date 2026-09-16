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
| `NOT_FOUND` | 404 | El recurso solicitado no existe. |
| `USER_NOT_FOUND` | 404 | El usuario especificado no fue encontrado. |
| `CONFLICT` | 409 | Conflicto de estado o duplicidad de clave única en la base de datos. |
| `EMAIL_ALREADY_EXISTS` | 409 | El email ya está registrado por otra cuenta. |
| `USERNAME_ALREADY_EXISTS` | 409 | El username ya está registrado por otra cuenta. |
| `INVALID_TOKEN` | 400 | Token de verificación de email inválido o ya utilizado. |
| `TOKEN_EXPIRED` | 400 | Token de verificación de email expirado (vigencia: 24 horas). |
| `USER_ALREADY_VERIFIED` | 400 | Se intentó reenviar verificación a una cuenta ya verificada. |
| `CANNOT_FOLLOW_SELF` | 400 | Se intentó seguir/dejar de seguir a la propia cuenta. |
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
El token emitido por `/auth/login` y `/auth/register` contiene las siguientes claims:

```json
{
  "sub": 1,
  "username": "pandora_artist",
  "email": "artist@pandora.art",
  "roles": ["user"],
  "iat": 1726442400,
  "exp": 1727047200
}
```

### 5.3 Roles del Sistema
| Rol | Identificador | Descripción |
|---|---|---|
| Usuario estándar | `user` | Rol por defecto al registrarse. Habilita publicación de obras, calificación Q2Q, apertura de paquetes, colección y autobattler. |
| Moderador | `moderator` | Acceso a panel de reportes, resolución de denuncias, advertencias y moderación de contenido. |
| Administrador | `admin` | Permisos totales sobre configuración, roles y auditoría. |

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
