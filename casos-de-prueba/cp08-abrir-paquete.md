# CP08 — Abrir paquete (CU17)

**Caso de uso:** [CU17 — Abrir paquete](/casos-de-uso/cu17-abrir-paquete.md)

- **Precondición:** el usuario tiene al menos un paquete disponible.
- **Pasos:** solicitar apertura → backend verifica disponibilidad → consume exactamente un paquete → sortea 5 cartas por rareza ponderada → incrementa copias en la colección → devuelve las cartas obtenidas.
- **Resultado esperado:** `amount` decrementado en 1; 5 cartas nuevas o incrementadas en `user_cards`; operación atómica.
- **Casos negativos:** sin paquetes disponibles → 409 (A1) · peticiones simultáneas no deben permitir doble consumo (A2) · fallo transaccional no deja estado parcial (A3).
