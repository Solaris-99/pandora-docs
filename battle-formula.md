# Fórmula de batalla

> Especificación de la mecánica de combate del CU de Batalla. La batalla es **100% client-side y nunca se persiste ni se envía al backend** (`api-conventions.md` §15, RG 7.8): el backend sólo entrega el rival (`GET /battle/opponent`). Por eso, para que web y mobile muestren el mismo resultado dado el mismo par de cartas (mismas stats), ambos clientes tienen que implementar exactamente esta fórmula — este documento es la fuente de verdad para eso. Implementación de referencia (frontend web): `src/features/battle/simulate.ts`.

Acordada el 2026-10-07. Las decisiones marcadas como **"(decisión de implementación)"** no estaban explícitas en la propuesta original y fueron resueltas al implementar — mobile debe replicarlas tal cual para que el resultado sea idéntico dado el mismo seed/rolls.

## Stats usadas

Cada carta participante tiene `attack`, `defense`, `hp`, `speed` (enteros ≥ 0, ver `cardStatsSchema` / api-conventions.md §13.1).

## Orden de turnos

- Al iniciar la batalla, la carta con mayor `speed` ataca primero. Ese orden se fija una sola vez al principio y se repite ronda tras ronda (no se vuelve a calcular por turno).
- **Empate de velocidad (decisión de implementación):** gana la prioridad la carta del propio jugador (la que inició la batalla), no la del rival.

## Resolución de un ataque

Dadas una carta atacante (AC) y una defensora (DC):

1. **Golpe crítico.** Se calcula la probabilidad de crítico del atacante:

   ```
   crit_chance = (AC.attack * AC.speed) / (AC.attack * AC.speed + (DC.speed + DC.defense) + 100)
   ```

   Se tira un número aleatorio uniforme en `[0, 1)`; si es menor a `crit_chance`, el golpe es crítico.
   - Por construcción, `crit_chance` siempre es menor a 1 (el denominador es el numerador más un resto positivo), así que no necesita un tope explícito.

2. **Esquive — sólo si NO fue crítico.** Los críticos nunca se pueden esquivar. Si el golpe no fue crítico, se calcula la probabilidad de esquive de la defensora:

   ```
   dodge_chance = min(0.75, DC.speed / (DC.speed + 7 + AC.speed))
   ```

   Tope: **75%**. Se tira otro número aleatorio uniforme en `[0, 1)`; si es menor a `dodge_chance`, el ataque falla por completo (0 de daño) y termina ahí — no se calcula reducción de daño ni nada más.

3. **Daño base.** Si el golpe fue crítico o si no esquivó:

   ```
   base_damage = AC.attack                              si no fue crítico
   base_damage = (AC.attack + 5) * (AC.speed + 1)        si fue crítico
   ```

4. **Reducción de daño.** Se calcula a partir de la defensa de la defensora (afecta tanto a golpes normales como a críticos):

   ```
   damage_reduction = min(0.90, DC.defense / (DC.defense + 100))
   ```

   Tope: **90%**.

5. **Daño final:**

   ```
   damage = round(base_damage * (1 - damage_reduction))
   damage = max(1, damage)   // (decisión de implementación, ver abajo)
   ```

   - **Redondeo:** se usa redondeo al entero más cercano (`round`), no truncamiento ni techo.
   - **Piso de 1 de daño (decisión de implementación):** un golpe que conecta (no esquivado) siempre hace como mínimo 1 de daño, aunque la reducción de daño lo deje matemáticamente en 0. La razón es de UX: un golpe que no fue esquivado pero que "no hace nada" es confuso para quien juega — se ve como si el ataque hubiera fallado sin indicación de por qué. El esquive sigue siendo la única forma de bloquear el daño por completo (0 de daño explícito, distinto de este piso).

6. Se aplica `damage` al HP de la defensora, nunca por debajo de 0 (`max(0, hp - damage)`).

## Bucle de batalla

- Cada ronda, ambas cartas atacan una vez, en el orden fijado al principio (ver "Orden de turnos").
- Si una carta llega a 0 HP, la batalla termina inmediatamente — la otra carta no llega a atacar en esa ronda si todavía no le tocaba.
- **Tope de rondas (decisión de implementación):** 500 rondas. Si ninguna carta llega a 0 HP en ese lapso (sólo posible con combinaciones de stats muy extremas donde esquive/reducción de daño absorben casi todo), gana la carta con mayor porcentaje de HP restante (`hp_actual / hp_máximo`); si ese porcentaje también empata, gana quien tenía prioridad de turno.

## Resumen de topes

| Valor | Tope |
| --- | --- |
| Esquive (`dodge_chance`) | 75% |
| Reducción de daño (`damage_reduction`) | 90% |
| Crítico (`crit_chance`) | sin tope explícito (siempre < 100% por construcción) |

## Fuera de alcance de esta versión

Multi-golpe, efectos de estado, bonus por tipo de emoción/rareza más allá de las stats base que ya determina la conversión — no están contemplados. Si se agregan más adelante, hay que actualizar este documento antes de tocar la implementación, para que ambos clientes se actualicen en paralelo.
