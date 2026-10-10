# Lógica de regeneración de paquetes

Se requiere un máximo de 10 paquetes y una regeneración de 1 paquete por minuto.

## Estado mínimo

```text
amount
last_regeneration_at
```

## Cálculo

Al consultar o abrir paquetes:

1. calcular minutos completos transcurridos;
2. determinar cuántos paquetes deben regenerarse;
3. limitar el resultado a 10;
4. actualizar `last_regeneration_at` de forma consistente;
5. si se llega a 10, evitar acumular tiempo pendiente de manera que al bajar a 9 aparezcan paquetes de golpe no previstos por el diseño.

La apertura debe ejecutarse con protección de concurrencia para que dos requests no consuman un único paquete dos veces.
