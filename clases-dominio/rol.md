# Rol

Etiqueta de autorización que un Usuario puede poseer, independiente de su identidad. Un Usuario puede tener más de un Rol a la vez (p. ej. `user` + `moderator`).

- **Valores:** `user` (rol por defecto), `moderator`, `admin`.
- **Responsabilidades:** habilitar o restringir funcionalidad según el conjunto de roles del Usuario que la solicita.
- **Relaciones:** N Usuarios ↔ N Roles.
- **Nota de negocio:** el rol `admin` es de confianza y se asigna fuera de la aplicación; nunca se otorga ni se revoca a través de un flujo propio del sistema.
