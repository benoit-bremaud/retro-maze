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
```

Fury is orthogonal to the mode above (it applies in scatter and chase alike), so it has its
own machine, owned by `FuryRule` for Ember only:

```mermaid
---
title: stm EmberFury
---
stateDiagram-v2
  [*] --> Normal
  Normal --> Fury1 : remaining ≤ threshold 1
  Fury1 --> Fury2 : remaining ≤ threshold 2
  Fury1 --> Suspended : hero dies
  Fury2 --> Suspended : hero dies
  Suspended --> Fury1 : Dusk has left the house [remaining ≤ threshold 1]
  Suspended --> Normal : Dusk has left the house [otherwise]
  Normal --> [*] : level clear
  Fury1 --> [*] : level clear
  Fury2 --> [*] : level clear
```

## Notes

- Energizer with fright time 0: Scatter/Chase reverse but stay in place (no `Frightened`).
- In Fury 1 / 2 Ember uses the Fury speed and keeps its chase target in scatter (GDD §8);
  from `Suspended` the stage is recomputed from the remaining pellets.
- Ghosts in the house during an energizer turn blue but stay `InHouse` (GDD §6.5).
- Leaving frightened never reverses (GDD §5.3).
