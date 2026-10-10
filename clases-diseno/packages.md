# Módulo `packages`

## Entidad: `CardPack`

`id`, `user_id` (único), `amount: number (0–10)`, `last_regeneration_at: Date | null`, `updated_at`.

## Servicio: `PackagesService`

`getInventory(userId)` (aplica el cálculo de regeneración de [diseño de arquitectura](/diseno-arquitectura/regeneracion-paquetes.md#cálculo) antes de responder), `openPackage(userId)` (consumo atómico + sorteo de 5 cartas por rareza ponderada + alta/incremento en `UserCard`).

Utilidades puras: `rarity-roll.util` (sorteo ponderado 64/20/10/5/1), `packages-regen.util` (cálculo de minutos/paquetes regenerados).
