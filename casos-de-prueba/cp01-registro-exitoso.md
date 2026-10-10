# CP01 — Registro exitoso (CU01)

**Caso de uso:** [CU01 — Registrarse](/casos-de-uso/cu01-registrarse.md)

- **Precondición:** no existe una cuenta con ese email ni ese username.
- **Pasos:** completar email/username/contraseña válidos → backend valida formato/longitud/unicidad → hashea la contraseña → crea la cuenta.
- **Resultado esperado:** cuenta creada en estado `pending_verification` (ver [CU02](/casos-de-uso/cu02-verificar-email.md) para el flujo de verificación de email, incluido el envío y confirmación del challenge).
- **Casos negativos:** email duplicado (A1) · username duplicado (A2) · email inválido (A3) · contraseña inválida (A4).
