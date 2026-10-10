# CU03 — Login

**Actor principal:** Invitado
**Precondición:** Cuenta registrada.
**Postcondición:** Sesión autenticada.
**Caso de prueba:** [CP02 — Login](/casos-de-prueba/cp02-login.md)

## Flujo principal

1. El usuario abre login.
2. Ingresa email y contraseña.
3. El backend aplica rate limit.
4. Busca el usuario.
5. Verifica credenciales.
6. Verifica el estado de la cuenta.
7. Genera sesión/JWT.
8. Devuelve el resultado de autenticación.

## Alternativo

- **A1:** Credenciales inválidas -> respuesta genérica de autenticación fallida.
- **A2:** Demasiados intentos fallidos -> bloqueo temporal.
- **A3:** Cuenta baneada -> acceso denegado.
- **A4:** Cuenta sin verificar -> se informa que debe completar la verificación.
- **A5:** Error del proveedor Google -> mensaje genérico de autenticación fallida.
