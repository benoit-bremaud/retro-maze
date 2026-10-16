# stm — game session

## Context

Lifecycle of a game session, from the title screen to the hall of fame (GDD §10, §9.4).

```mermaid
---
title: stm GameSession
---
stateDiagram-v2
  [*] --> Title
  Title --> Ready : start pressed / new game (3 lives, level 1)
  Ready --> Playing : after 2 s
  Playing --> Paused : pause pressed
  Paused --> Playing : resume pressed
  Playing --> GhostEaten : hero eats a frightened ghost
  GhostEaten --> Playing : after 1 s
  Playing --> Dying : collision with a non-frightened ghost
  Dying --> Ready : lives > 0 / reset positions, global counter on
  Dying --> GameOver : lives == 0
  Playing --> LevelClear : 274 pellets eaten
  LevelClear --> Ready : after 2 s / level + 1, counters and Fury reset
  GameOver --> NameEntry : score in top 10
  GameOver --> HallOfFame : otherwise
  NameEntry --> HallOfFame : name confirmed / save
  HallOfFame --> Title : any key
  Title --> Options : options chosen
  Options --> Title : back
```

## Notes

- The scatter/chase timer and fright timer only advance in `Playing` (GDD §6.4).
- `GhostEaten` and `Dying` are freeze states: the simulation steps but actors do not move.
