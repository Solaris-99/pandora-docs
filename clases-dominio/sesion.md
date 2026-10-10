# Sesión (Refresh Token)

Representa una sesión de larga duración que permite renovar el token de acceso sin volver a autenticarse con credenciales.

- **Atributos conceptuales:** usuario, secreto opaco, expiración, si fue revocada.
- **Responsabilidades:** rotarse en cada uso (el token usado queda revocado y se emite uno nuevo); revocarse íntegramente al cerrar sesión o al restablecer la contraseña del Usuario.
- **Relaciones:** pertenece a 1 Usuario. Un Usuario puede tener múltiples sesiones activas simultáneamente (p. ej. web + Android).
