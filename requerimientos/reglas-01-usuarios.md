# 5.1 Usuarios

1. Un username debe ser único.
2. Un email debe ser único.
3. Un usuario autenticado es simultáneamente potencial artista y coleccionista.
4. Una cuenta local debe verificar el email.
5. Google puede utilizarse como proveedor alternativo de autenticación.
6. Discord OAuth no forma parte del sistema.
7. Un usuario con contraseña local puede solicitar recuperarla mediante un enlace de un solo uso enviado por email, válido por 1 hora. Una cuenta creada únicamente vía Google (sin contraseña local) no tiene nada que recuperar por esta vía: la solicitud responde igual que para cualquier otra dirección (mensaje neutro), pero no se envía correo.
8. Restablecer la contraseña revoca todas las sesiones (`refresh_tokens`) activas del usuario; no inicia sesión automáticamente.
