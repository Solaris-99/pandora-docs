# CU01 — Registrarse

**Actor principal:** Invitado
**Precondición:** No existe una sesión autenticada.
**Postcondición:** Cuenta creada en estado pendiente de verificación (ver [CU02](cu02-verificar-email.md)).
**Caso de prueba:** [CP01 — Registro exitoso](/casos-de-prueba/cp01-registro-exitoso.md)

## Flujo principal

1. El usuario accede a la pantalla de registro.
2. Introduce email, username y contraseña.
3. El backend valida formato, longitud y unicidad.
4. Se hashea la contraseña.
5. Se crea la cuenta en estado pendiente de verificación.

## Flujos alternativos / excepciones

- **A1:** El email ya existe. Se rechaza el registro.
- **A2:** El username ya existe. Se rechaza el registro.
- **A3:** Email inválido. Se devuelve error de validación.
- **A4:** Contraseña no válida. Se devuelve error de validación.

## Notas de implementación

Crear la cuenta dispara inmediatamente el flujo de verificación de email ([CU02](cu02-verificar-email.md)). Una cuenta que nunca se verifica es eliminada automáticamente a las 72 horas ([CU05](cu05-limpieza-de-cuentas-no-verificadas.md)).
