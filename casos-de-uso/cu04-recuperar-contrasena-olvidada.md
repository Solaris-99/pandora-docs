# CU04 — Recuperar contraseña olvidada

**Actor principal:** Invitado (usuario que no puede iniciar sesión)
**Precondición:** Existe una cuenta con esa dirección de email; si la cuenta tiene contraseña local configurada, la solicitud desencadena el envío del enlace.
**Postcondición:** El usuario cuenta con una nueva contraseña y puede iniciar sesión con ella; cualquier sesión abierta con la contraseña anterior quedó cerrada.
**Caso de prueba:** [CP23 — Recuperar contraseña olvidada](/casos-de-prueba/cp23-recuperar-contrasena-olvidada.md)

## Flujo principal

1. El usuario abre "¿Olvidaste tu contraseña?" desde la pantalla de login.
2. Introduce su email.
3. El backend responde con un mensaje neutro, sin revelar si la cuenta existe.
4. Si la cuenta existe y tiene contraseña local, se genera un token de un solo uso y se envía por email.
5. El usuario abre el enlace recibido.
6. Introduce una nueva contraseña.
7. El backend valida el token y la actualiza.
8. Se revocan todas las sesiones (`refresh_tokens`) activas de esa cuenta.
9. El usuario inicia sesión nuevamente con la contraseña nueva.

## Alternativo

- **A1:** El email no corresponde a ninguna cuenta -> mismo mensaje neutro que en el flujo exitoso (anti-enumeración), no se envía nada.
- **A2:** La cuenta existe pero es exclusivamente de Google (sin contraseña local) -> mismo mensaje neutro, no se envía nada: no hay contraseña que recuperar.
- **A3:** Token inválido, inexistente o ya utilizado -> 400.
- **A4:** Token expirado (más de 1 hora) -> 400, el usuario debe solicitar uno nuevo.
- **A5:** Nueva contraseña fuera de la política de longitud -> 400.

## Observaciones

No inicia sesión automáticamente al restablecer la contraseña (a diferencia de [CU02](cu02-verificar-email.md), que sí deja la cuenta verificada y lista para loguearse apenas se confirma el challenge) — es una operación más sensible y se prefiere que el usuario reingrese explícitamente con la contraseña nueva.
