# Credencial temporal (Token de verificación / recuperación)

Dos variantes de un mismo concepto: un secreto de un solo uso, con expiración, emitido para completar una operación sensible sin exponer la contraseña del Usuario.

- **Token de verificación de email:** confirma que el Usuario controla la dirección de correo con la que se registró.
- **Token de recuperación de contraseña:** habilita, por única vez, establecer una contraseña nueva sin conocer la anterior.
- **Responsabilidades comunes:** expirar automáticamente; invalidarse tras un único uso; invalidar cualquier token anterior sin usar del mismo Usuario al emitir uno nuevo.
- **Relaciones:** cada instancia pertenece a exactamente 1 Usuario.
