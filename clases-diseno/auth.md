# Módulo `auth`

## Entidades

- **EmailVerificationToken**: `id`, `user_id`, `token_hash: string`, `expires_at: Date`, `used_at: Date | null`, `created_at`. Relación `ManyToOne` a `User`.
- **PasswordResetToken**: misma forma que `EmailVerificationToken` (tabla separada).
- **RefreshToken**: `id`, `user_id`, `token_hash: string`, `expires_at: Date`, `revoked_at: Date | null`, `created_at`. Relación `ManyToOne` a `User`.
- **OAuthAccount**: `id`, `user_id`, `provider: string`, `provider_user_id: string`. Relación `ManyToOne` a `User`.

## Servicio: `AuthService`

| Operación | Responsabilidad |
|---|---|
| `register(dto)` | Valida unicidad, hashea contraseña (bcrypt), crea `User`, emite token de verificación y correo, devuelve par de tokens. |
| `login(dto)` | Verifica credenciales (`bcrypt.compare`), bloquea cuentas `suspended`, devuelve par de tokens. |
| `verifyEmail(token)` | Consume un `EmailVerificationToken`, activa la cuenta, devuelve par de tokens (auto-login). |
| `resendVerification(email)` | Invalida tokens previos, emite uno nuevo — respuesta neutra si el email no existe (anti-enumeración). |
| `forgotPassword(email)` | Igual patrón anti-enumeración; emite `PasswordResetToken` sólo si la cuenta tiene contraseña local. |
| `resetPassword(token, newPassword)` | Consume el token, actualiza `password_hash`, revoca todos los `RefreshToken` activos del usuario. |
| `handleGoogleAuth(googleUser)` / `loginWithGoogleIdToken(idToken)` | Vincula o crea cuenta a partir de un perfil de Google. |
| `refreshTokens(rawToken)` | Rota el refresh token (revoca el usado, emite uno nuevo). |
| `logout(rawToken)` | Revoca un refresh token puntual. |
| `getProfile(userId)` | Devuelve el perfil del usuario autenticado. |

Métodos privados clave: `hashToken()` (SHA-256, para tokens opacos), `issueRefreshToken()`.

## Estrategias y guards

- **JwtStrategy.validate(payload)**: resuelve el usuario desde el JWT, bloquea `suspended`, permite `banned` (necesita autenticarse para apelar).
- **GoogleStrategy.validate(...)**: mapea el perfil de Google a la forma interna esperada por `handleGoogleAuth`.
- **JwtAuthGuard**: respeta `@Public()`; delega en Passport; normaliza el error 401.
- **GoogleOAuthGuard**: delega en Passport (`AuthGuard('google')`).

## DTOs

`RegisterDto`, `LoginDto`, `VerifyEmailDto`, `ResendVerificationDto`, `ForgotPasswordDto`, `ResetPasswordDto`, `RefreshTokenDto`, `GoogleLoginDto` (entrada) · `TokenPairResponseDto`, `VerifyEmailResponseDto`, `MeResponseDto` (salida).
