# Inventario de Paquetes

Representa la cantidad de paquetes de cartas disponibles para un Usuario y el estado de su regeneración periódica.

- **Atributos conceptuales:** usuario propietario, cantidad actual, momento de la última regeneración.
- **Responsabilidades:** regenerar un paquete por minuto hasta un máximo de 10, sin acumular deuda una vez alcanzado el máximo; permitir el consumo atómico de un paquete, que entrega 5 Cartas nuevas al Usuario.
- **Relaciones:** pertenece a exactamente 1 Usuario (1 a 1).
