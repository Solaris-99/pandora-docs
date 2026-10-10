# Recomendaciones de diseño para evitar deuda técnica

1. Mantener la generación de cartas como servicio de dominio independiente del Scheduler.
2. No mezclar entidades TypeORM con DTOs de entrada.
3. No confiar en restricciones del frontend para autorización.
4. Aplicar ownership checks además de roles.
5. Usar transacciones para apertura de paquetes y conversión de cartas.
6. Registrar estados de procesos que puedan reintentarse.
7. Mantener soft delete en contenido moderado cuando la auditoría lo requiera.
8. Mantener la lógica de batalla fuera del backend mientras no haya persistencia ni interacción remota.
9. Evitar endpoints que existan sólo porque "podrían hacer falta" pero no tengan una operación de dominio real.
10. Versionar las migraciones de PostgreSQL desde el comienzo.
