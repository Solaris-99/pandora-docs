# CU06 — Ver/editar perfil

**Actor principal:** Usuario autenticado (consulta pública disponible para cualquier actor).

## Flujo principal

1. El usuario abre un perfil.
2. El sistema devuelve información pública.
3. Si el perfil es propio, se habilita edición.
4. Se guardan los cambios válidos.

## Alternativo

- **A1:** Username elegido por otro usuario -> conflicto de unicidad.
- **A2:** Datos fuera de rango -> 400.
- **A3:** Usuario no autenticado intentando editar -> 401.

## Observaciones

El cambio de email y el cambio de contraseña tienen flujos propios ([CU07](cu07-cambiar-email.md) y [CU08](cu08-cambiar-contrasena.md), respectivamente) por los pasos de confirmación adicionales que requieren.
