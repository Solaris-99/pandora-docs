# CU08 — Cambiar contraseña

**Actor principal:** Usuario autenticado, con contraseña local configurada.
**Precondición:** El usuario tiene sesión autenticada.
**Postcondición:** La contraseña de la cuenta queda actualizada.
**Caso de prueba:** todavía sin caso de prueba dedicado.

## Flujo principal

1. El usuario entra a la configuración de su perfil.
2. Elige "Cambiar contraseña".
3. Introduce su contraseña actual.
4. Introduce la nueva contraseña en dos campos: "nueva contraseña" y "repetir nueva contraseña".
5. El backend verifica que la contraseña actual sea correcta.
6. Valida que los dos campos de la contraseña nueva coincidan y cumplan la política de contraseñas.
7. Hashea y almacena la nueva contraseña.

## Alternativo

- **A1:** La contraseña actual es incorrecta -> rechazar.
- **A2:** La nueva contraseña y su repetición no coinciden -> 400.
- **A3:** La nueva contraseña está fuera de la política de longitud -> 400.
- **A4:** La cuenta no tiene contraseña local (sólo Google) -> este caso de uso no aplica.

## Observaciones

Por consistencia con [CU04](cu04-recuperar-contrasena-olvidada.md) (recuperación de contraseña), se recomienda revocar las demás sesiones (`refresh_tokens`) activas al confirmar el cambio.
