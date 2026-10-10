# CP21 — Cargar carta directamente (Admin) (CU29)

**Caso de uso:** [CU29 — Cargar carta directamente (Admin)](/casos-de-uso/cu29-cargar-carta-directamente.md)

- **Precondición:** actor `admin`.
- **Pasos:** completar título/descripción/tags → seleccionar imagen → indicar rareza → opcionalmente estadísticas explícitas → backend sube la imagen, crea la obra `converted` y la carta en el mismo paso; estadísticas faltantes se completan según la rareza.
- **Resultado esperado:** obra + carta creadas atómicamente; sin período de calificación; admin no posee automáticamente la carta.
- **Casos negativos:** archivo inválido/ausente (A1) · título inválido (A2) · rareza inválida → 400 (A3) · estadística negativa o no entera → 400 (A4) · actor sin `admin` → 403 (A5).
- **Verificación en vivo realizada:** confirmado por HTTP real (no sólo test unitario) — override explícito de estadísticas respetado campo a campo, y default por rareza correctamente calculado cuando se omiten.
