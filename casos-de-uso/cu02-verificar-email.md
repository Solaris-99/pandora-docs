# CU02 — Verificar email

**Actor principal:** Invitado (titular de una cuenta recién creada o de una cuenta pendiente que solicita un nuevo envío)
**Precondición:** Existe una cuenta en estado pendiente de verificación.
**Postcondición:** La cuenta queda verificada (`email_verified = true`) y activa.

## Flujo principal

1. Se genera un token/challenge de verificación (al registrarse en [CU01](cu01-registrarse.md), o al solicitar un reenvío).
2. Se envía el correo de verificación.
3. El usuario abre el enlace o introduce el código recibido.
4. El backend valida el challenge.
5. La cuenta queda verificada y activa.
6. Se informa que la verificación fue exitosa.

## Flujos alternativos / excepciones

- **A1:** El envío del correo falla. La cuenta queda pendiente y se permite reintentar el envío.
- **A2:** Challenge expirado. Se solicita uno nuevo.
- **A3:** Challenge inválido. Se rechaza la confirmación.
- **A4:** La cuenta ya estaba verificada. Operación idempotente, se informa que no había nada pendiente.

## Notas de implementación

Los tokens de verificación deben expirar y ser de un solo uso. Ver [CU05](cu05-limpieza-de-cuentas-no-verificadas.md) para la limpieza automática de las cuentas que nunca completan este paso.
