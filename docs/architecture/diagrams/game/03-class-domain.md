# cd — gameplay domain

## Context

Domain types of the simulation (no browser API). Fixed-size arrays only: one hero, four
ghosts, one bonus item — no pool needed, no allocation in the loop (CLAUDE.md).

```mermaid
---
title: cd Gameplay domain
---
classDiagram
  direction LR
  class Game {
    +step(intent: IntentFrame) void
    -level: number
    -score: number
    -lives: number
  }
  class Maze {
    +isWalkable(col, row) boolean
    +isNoUp(col, row) boolean
    +isTunnel(col, row) boolean
    +eatPellet(col, row) PelletKind
    +remainingPellets() number
  }
  class Hero {
    -tile: TileVector
    -direction: Direction
    -desired: Direction
    -skipSteps: number
    +step(maze, speed) void
  }
  class Ghost {
    -id: GhostId
    -mode: GhostMode
    -tile: TileVector
    -direction: Direction
    -pendingReverse: boolean
    +onTileEntered(maze, target, rng) void
  }
  class TargetingStrategy {
    <<interface>>
    +chaseTarget(hero, ember, out) void
  }
  class ChaserTarget
  class AmbusherTarget
  class FlankerTarget
  class ShyTarget
  class ModeSchedule {
    +phase() SchedulePhase
    +tick() boolean
  }
  class FrightTimer
  class HouseRelease {
    +onPelletEaten() void
    +onIdleTick() void
    +onDeath() void
  }
  class ElroyRule {
    +stage(remaining, level) number
  }
  class BonusItem
  class LevelTables {
    <<data>>
    speeds, schedule, fright, dotLimits, elroy, bonus
  }
  class RandomPort {
    <<interface>>
    +nextInt(n) number
  }

  Game *-- Maze
  Game *-- Hero
  Game *-- "4" Ghost
  Game *-- ModeSchedule
  Game *-- FrightTimer
  Game *-- HouseRelease
  Game *-- ElroyRule
  Game *-- "0..1" BonusItem
  Game ..> LevelTables
  Ghost --> TargetingStrategy
  TargetingStrategy <|.. ChaserTarget
  TargetingStrategy <|.. AmbusherTarget
  TargetingStrategy <|.. FlankerTarget
  TargetingStrategy <|.. ShyTarget
  Ghost ..> RandomPort : frightened turns
```

## Notes

- `TargetingStrategy` writes into a preallocated `out` vector (no allocation).
- `LevelTables` hold GDD §5–§9 as data; indexes are clamped to the last row (21+, 19+ …).
- `IntentFrame`, `RandomPort` come from the ports copied from retro-shmup (ADR-0003).
