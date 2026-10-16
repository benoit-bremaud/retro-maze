# uc — retro-maze player goals

## Context

Who interacts with the game, and to do what. Goals are actor-initiated; game events (a ghost
leaves the house, Elroy starts) are triggers inside *Clear a maze*, not use cases.

```mermaid
flowchart LR
  Player(["👤 Player"])
  subgraph System["retro-maze (browser game)"]
    direction TB
    subgraph Play
      UC1(["Start a game"])
      UC2(["Clear a maze"])
      UC3(["Eat frightened ghosts for a combo"])
      UC4(["Collect a bonus item"])
      UC5(["Pause and resume"])
    end
    subgraph Records
      UC6(["Enter a name in the top 10"])
      UC7(["Consult the hall of fame"])
    end
    subgraph Settings
      UC8(["Rebind controls"])
      UC9(["Adjust audio and effect options"])
      UC10(["Choose the language"])
    end
  end
  Player --- UC1
  Player --- UC2
  Player --- UC3
  Player --- UC4
  Player --- UC5
  Player --- UC6
  Player --- UC7
  Player --- UC8
  Player --- UC9
  Player --- UC10
  UC3 -. "«extend» energizer eaten" .-> UC2
  UC4 -. "«extend» 70 / 170 pellets" .-> UC2
```

## Notes

- One actor: the player. No backend, no account (CLAUDE.md *Forbidden*).
- Rules behind each goal: GDD §5–§10. Name entry and hall of fame come from retro-shmup
  (ADR-0003).
