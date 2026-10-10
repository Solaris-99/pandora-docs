# CU17 — Abrir paquete

**Actor principal:** Usuario autenticado
**Precondición:** El usuario tiene al menos un paquete disponible.
**Caso de prueba:** [CP08 — Abrir paquete](/casos-de-prueba/cp08-abrir-paquete.md)

## Flujo principal

1. El usuario entra a la sección de paquetes.
2. El cliente solicita abrir un paquete.
3. El backend verifica disponibilidad.
4. Se consume exactamente un paquete.
5. Se seleccionan cartas según las reglas del sistema.
6. Se genera un número al azar y con eso se decide la rareza.
7. Se da una carta de esa rareza al azar.
8. Se incrementan las copias en la colección.
9. Se devuelve la lista de cartas obtenidas.
10. El cliente presenta la animación y el resultado.

## Alternativo

- **A1:** No hay paquetes -> 409 o respuesta de estado equivalente.
- **A2:** Peticiones simultáneas -> la operación debe protegerse contra doble consumo.
- **A3:** Error transaccional -> ni el paquete ni las recompensas deben quedar parcialmente aplicados.

## Observaciones

La regeneración de paquetes debe calcularse a partir de `last_used` y el tiempo transcurrido.
