# Módulo `conversion`

## Servicio: `ConversionService`

`determineRarity(totalVotes, totalRegisteredUsers)`, `convert(input)` → `{attack, defense, hp, speed, rarity}`. Ambos puros, sin efectos secundarios.

## Función pura: `computeStatsForRarity(rarity, averageStars?, multipliers?)`

Usada tanto por la conversión real como por el sembrado de catálogo (rareza sin votos reales de por medio).
