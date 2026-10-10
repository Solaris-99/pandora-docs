# CU07 — Cambiar email

**Actor principal:** Usuario autenticado, con email verificado.
**Precondición:** El usuario tiene sesión autenticada.
**Postcondición:** El email de la cuenta queda actualizado al nuevo valor, una vez verificado.
**Caso de prueba:** todavía sin caso de prueba dedicado.

## Flujo principal

1. El usuario entra a la configuración de su perfil.
2. Elige "Cambiar email".
3. Introduce un nuevo email válido.
4. El sistema muestra un popup de confirmación.
5. El usuario confirma.
6. Se envía un correo de verificación a la nueva dirección (mismo mecanismo que [CU02](cu02-verificar-email.md)).
7. El email de la cuenta se actualiza una vez que la nueva dirección es verificada.

## Alternativo

- **A1:** El nuevo email coincide con el actual -> rechazar o no-op.
- **A2:** El nuevo email ya está en uso por otra cuenta -> rechazar (unicidad).
- **A3:** Email inválido -> 400.
- **A4:** El usuario cancela el popup de confirmación -> no se envía nada, no hay cambios.
- **A5:** El usuario no verifica la nueva dirección -> el email de la cuenta permanece siendo el anterior hasta que se verifique o se solicite un nuevo envío.

## Observaciones

Mientras la verificación esté pendiente, el email anterior sigue siendo el activo (login, notificaciones): el cambio sólo se aplica al confirmar, igual que en [CU02](cu02-verificar-email.md).
