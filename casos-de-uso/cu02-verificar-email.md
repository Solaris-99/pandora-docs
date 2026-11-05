# CU02 — Verificar email

**Actor principal:** Invitado (titular de una cuenta recién creada o de una cuenta pendiente que solicita un nuevo envío)
**Precondición:** Existe una cuenta en estado pendiente de verificación.
**Postcondición:** La cuenta queda verificada (`email_verified = true`) y activa, con la sesión iniciada.
**Caso de prueba:** todavía sin caso de prueba dedicado.

## Flujo principal

1. Se genera un token de verificación (al registrarse en [CU01](cu01-registrarse.md), o al solicitar un reenvío).
2. Se envía un correo con un enlace que contiene el token.
3. El usuario abre el enlace, que lleva al portal web (también si se registró desde la app Android).
4. El portal envía el token y el backend lo valida.
5. La cuenta queda verificada y activa.
6. Se inicia la sesión y se informa que la verificación fue exitosa.

## Flujos alternativos / excepciones

- **A1:** El envío del correo falla. La cuenta queda pendiente y se permite reintentar el envío.
- **A2:** Enlace expirado (vigencia de 24 horas). Se rechaza la confirmación y se puede solicitar uno nuevo.
- **A3:** Enlace inválido o ya utilizado. Se rechaza la confirmación y se puede solicitar uno nuevo.
- **A4:** Se solicita un reenvío para una cuenta ya verificada. Se rechaza con `USER_ALREADY_VERIFIED`.

## Notas de implementación

Los tokens de verificación deben expirar y ser de un solo uso. Ver [CU05](cu05-limpieza-de-cuentas-no-verificadas.md) para la limpieza automática de las cuentas que nunca completan este paso.
