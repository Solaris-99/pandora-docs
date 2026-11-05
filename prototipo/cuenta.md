# Cuenta

Alta, acceso y recuperación de la cuenta, y configuración del perfil propio.

## Registro

**Ruta:** `/register` | **Actor:** Invitado

Formulario de alta con nombre de usuario, email y contraseña, y la política de contraseñas visible. La cuenta se crea pendiente de verificación, lo que dispara el envío del correo.

**Casos de uso:** [CU01 — Registrarse](/casos-de-uso/cu01-registrarse.md) · [CU02 — Verificar email](/casos-de-uso/cu02-verificar-email.md)

| Escritorio | Móvil |
|---|---|
| ![Registro en escritorio](img/desktop/registro.webp ':size=520') | ![Registro en móvil](img/mobile/registro.webp ':size=180') |

---

## Inicio de sesión

**Ruta:** `/login` | **Actor:** Invitado

Acceso con email y contraseña o con Google. Enlaza a la recuperación de contraseña y al registro.

**Casos de uso:** [CU03 — Login](/casos-de-uso/cu03-login.md) · [CU04 — Recuperar contraseña olvidada](/casos-de-uso/cu04-recuperar-contrasena-olvidada.md)

| Escritorio | Móvil |
|---|---|
| ![Inicio de sesión en escritorio](img/desktop/login.webp ':size=520') | ![Inicio de sesión en móvil](img/mobile/login.webp ':size=180') |

---

## Error de inicio de sesión con Google

**Ruta:** `/auth/callback` | **Actor:** Invitado

Destino de la redirección de Google. Normalmente sólo resuelve la sesión y lleva al inicio; esta es la variante de error, con un acceso para volver a intentar.

**Casos de uso:** [CU03 — Login](/casos-de-uso/cu03-login.md)

| Escritorio | Móvil |
|---|---|
| ![Error de inicio de sesión con Google en escritorio](img/desktop/callback-google-error.webp ':size=520') | ![Error de inicio de sesión con Google en móvil](img/mobile/callback-google-error.webp ':size=180') |

---

## Verificar email: pendiente

**Ruta:** `/verify-email` | **Actor:** Usuario con email sin verificar

Se llega aquí al intentar una acción que exige email verificado, como subir una obra. Permite reenviar el enlace a la dirección de la sesión. El aviso superior acompaña a la cuenta en todo el sitio hasta que se verifique.

**Casos de uso:** [CU02 — Verificar email](/casos-de-uso/cu02-verificar-email.md)

| Escritorio | Móvil |
|---|---|
| ![Verificar email: pendiente en escritorio](img/desktop/verificar-email-pendiente.webp ':size=520') | ![Verificar email: pendiente en móvil](img/mobile/verificar-email-pendiente.webp ':size=180') |

---

## Verificar email: confirmado

**Ruta:** `/verify-email?token=…` | **Actor:** Invitado

Destino del enlace recibido por correo cuando el token es válido. Confirma la verificación e inicia la sesión.

**Casos de uso:** [CU02 — Verificar email](/casos-de-uso/cu02-verificar-email.md)

| Escritorio | Móvil |
|---|---|
| ![Verificar email: confirmado en escritorio](img/desktop/verificar-email-exito.webp ':size=520') | ![Verificar email: confirmado en móvil](img/mobile/verificar-email-exito.webp ':size=180') |

---

## Verificar email: enlace inválido

**Ruta:** `/verify-email?token=…` | **Actor:** Usuario con email sin verificar

Variante para un token inválido o vencido: informa el error y ofrece pedir un enlace nuevo.

**Casos de uso:** [CU02 — Verificar email](/casos-de-uso/cu02-verificar-email.md)

| Escritorio | Móvil |
|---|---|
| ![Verificar email: enlace inválido en escritorio](img/desktop/verificar-email-error.webp ':size=520') | ![Verificar email: enlace inválido en móvil](img/mobile/verificar-email-error.webp ':size=180') |

---

## Recuperar contraseña

**Ruta:** `/forgot-password` | **Actor:** Invitado

Solicitud del enlace de recuperación a partir del email.

**Casos de uso:** [CU04 — Recuperar contraseña olvidada](/casos-de-uso/cu04-recuperar-contrasena-olvidada.md)

| Escritorio | Móvil |
|---|---|
| ![Recuperar contraseña en escritorio](img/desktop/recuperar-contrasena.webp ':size=520') | ![Recuperar contraseña en móvil](img/mobile/recuperar-contrasena.webp ':size=180') |

---

## Recuperar contraseña: solicitud enviada

**Ruta:** `/forgot-password` | **Actor:** Invitado

Confirmación con un mensaje neutro, que no revela si la cuenta existe.

**Casos de uso:** [CU04 — Recuperar contraseña olvidada](/casos-de-uso/cu04-recuperar-contrasena-olvidada.md)

| Escritorio | Móvil |
|---|---|
| ![Recuperar contraseña: solicitud enviada en escritorio](img/desktop/recuperar-contrasena-enviado.webp ':size=520') | ![Recuperar contraseña: solicitud enviada en móvil](img/mobile/recuperar-contrasena-enviado.webp ':size=180') |

---

## Restablecer contraseña

**Ruta:** `/reset-password?token=…` | **Actor:** Invitado

Destino del enlace de recuperación: pide la contraseña nueva dos veces.

**Casos de uso:** [CU04 — Recuperar contraseña olvidada](/casos-de-uso/cu04-recuperar-contrasena-olvidada.md)

| Escritorio | Móvil |
|---|---|
| ![Restablecer contraseña en escritorio](img/desktop/restablecer-contrasena.webp ':size=520') | ![Restablecer contraseña en móvil](img/mobile/restablecer-contrasena.webp ':size=180') |

---

## Confirmar cambio de email

**Ruta:** `/confirm-email-change?token=…` | **Actor:** Titular del nuevo email

Destino del enlace enviado a la dirección nueva. Recién al abrirlo se aplica el cambio; funciona aunque no haya sesión abierta.

**Casos de uso:** [CU07 — Cambiar email](/casos-de-uso/cu07-cambiar-email.md)

| Escritorio | Móvil |
|---|---|
| ![Confirmar cambio de email en escritorio](img/desktop/confirmar-cambio-email.webp ':size=520') | ![Confirmar cambio de email en móvil](img/mobile/confirmar-cambio-email.webp ':size=180') |

---

## Configuración de perfil

**Ruta:** `/settings/profile` | **Actor:** Usuario autenticado

Reúne en una sola pantalla la edición de nombre de usuario y bio, la subida de avatar, el cambio de email y el cambio de contraseña. La bio se consulta al abrir la pantalla: hasta recibirla, el campo queda deshabilitado con un indicador de carga.

**Casos de uso:** [CU06 — Ver/editar perfil](/casos-de-uso/cu06-ver-editar-perfil.md) · [CU07 — Cambiar email](/casos-de-uso/cu07-cambiar-email.md) · [CU08 — Cambiar contraseña](/casos-de-uso/cu08-cambiar-contrasena.md) · [CU31 — Subir avatar](/casos-de-uso/cu31-subir-avatar.md)

| Escritorio | Móvil |
|---|---|
| ![Configuración de perfil en escritorio](img/desktop/configuracion-perfil.webp ':size=520') | ![Configuración de perfil en móvil](img/mobile/configuracion-perfil.webp ':size=180') |

---

## Diálogo: confirmar cambio de email

**Ruta:** `/settings/profile` | **Actor:** Usuario autenticado

Confirmación que aparece al pedir el cambio, con la dirección nueva a la vista. El correo sólo se envía al confirmar; si se cancela, no hay cambios.

**Casos de uso:** [CU07 — Cambiar email](/casos-de-uso/cu07-cambiar-email.md)

| Escritorio | Móvil |
|---|---|
| ![Diálogo: confirmar cambio de email en escritorio](img/desktop/dialogo-confirmar-cambio-email.webp ':size=520') | ![Diálogo: confirmar cambio de email en móvil](img/mobile/dialogo-confirmar-cambio-email.webp ':size=180') |
