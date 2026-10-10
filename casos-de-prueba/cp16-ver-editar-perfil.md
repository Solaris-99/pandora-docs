# CP16 — Ver/editar perfil (CU06)

**Caso de uso:** [CU06 — Ver/editar perfil](/casos-de-uso/cu06-ver-editar-perfil.md)

- **Pasos:** abrir perfil → se devuelve información pública → si es el propio, habilitar edición → guardar cambios válidos.
- **Resultado esperado:** perfil actualizado; consulta pública siempre disponible.
- **Casos negativos:** username ya usado por otro (A1) · datos fuera de rango → 400 (A2) · edición sin autenticación → 401 (A3).
