# CP19 — Apelar una sanción (CU27)

**Caso de uso:** [CU27 — Apelar una sanción](/casos-de-uso/cu27-apelar-una-sancion.md)

- **Precondición:** reporte resuelto con sanción punitiva sobre el contenido/cuenta del actor, aún no apelada.
- **Pasos:** elegir motivo predefinido → opcionalmente agregar texto → enviar → apelación queda `pending`.
- **Resultado esperado:** una única `Appeal` por sanción, visible para moderación.
- **Casos negativos:** apelación previa para la misma sanción → 409 (A1) · sin sanción punitiva apelable → 404 (A2) · actor no es el afectado → 403 (A3) · cuenta baneada apelando su propio baneo → permitido explícitamente (A4).
