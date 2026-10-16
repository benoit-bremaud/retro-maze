# stm — ghost mode

## Context

Lifecycle of one ghost (GDD §6, §7, §8). The global scatter/chase phase is owned by the
`ModeSchedule`; each ghost follows it while not frightened, eaten or in the house.

```mermaid
---
title: stm GhostMode
---
stateDiagram-v2
  [*] --> InHouse : Bloom, Tide, Dusk
  [*] --> Scatter : Ember
  InHouse --> Leaving : dot counter limit, global counter or idle timer
  Leaving --> Scatter : at exit tile [schedule = scatter]
  Leaving --> Chase : at exit tile [schedule = chase]
  Scatter --> Chase : schedule tick / reverse
  Chase --> Scatter : schedule tick / reverse
  Scatter --> Frightened : energizer [fright time > 0] / reverse
  Chase --> Frightened : energizer [fright time > 0] / reverse
  Frightened --> Frightened : energizer / restart timer
  Frightened --> Scatter : timer ends [schedule = scatter]
  Frightened --> Chase : timer ends [schedule = chase]
  Frightened --> Eaten : hero on same tile or swap / score combo
  Eaten --> Entering : at exit tile
  Entering --> Leaving : revived at house centre
  state Chase {
    [*] --> Normal
    Normal --> Elroy1 : Ember and remaining ≤ threshold 1
    Elroy1 --> Elroy2 : remaining ≤ threshold 2
  }
```

## Notes

- Energizer with fright time 0: Scatter/Chase reverse but stay in place (no `Frightened`).
- Elroy applies to Ember only, also in scatter (Ember keeps the chase target); suspended after
  a death until Dusk leaves the house (GDD §8). Drawn inside `Chase` for readability.
- Ghosts in the house during an energizer turn blue but stay `InHouse` (GDD §6.5).
- Leaving frightened never reverses (GDD §5.3).
