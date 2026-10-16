# cmp — domain / adapters / app

## Context

Structural boundaries. The dependency rule: `domain` imports nothing from `adapters` or `app`
and no browser API; adapters implement the domain's ports; `app` composes them.

```mermaid
---
title: cmp retro-maze
---
flowchart TB
  subgraph app["app (composition root)"]
    main["main.ts"]
    loop["frame-loop"]
  end
  subgraph domain["domain (pure TS)"]
    game["game: Game, Maze, Hero, Ghost, rules, tables"]
    scenes["scenes: title, name-entry, hall-of-fame"]
    meta["save, hall-of-fame, i18n, input bindings, audio synth"]
    ports["ports: RenderPort, AudioPort, InputPort, StoragePort, Clock, RandomPort"]
    rng["random: seeded-random"]
  end
  subgraph adapters["adapters (browser)"]
    render["canvas2d-renderer, sprite-atlas, viewport"]
    audio["web-audio-player"]
    input["input: keyboard, gamepad, pointer"]
    store["local-storage store"]
    clock["performance-clock"]
  end
  main --> loop
  main --> adapters
  main --> domain
  loop --> clock
  loop --> game
  game --> ports
  scenes --> ports
  meta --> ports
  game --> rng
  render -. implements .-> ports
  audio -. implements .-> ports
  input -. implements .-> ports
  store -. implements .-> ports
  clock -. implements .-> ports
```

## Notes

- Module origins: ADR-0003 table (retro-shmup `f698786`).
- Only `main.ts` touches `window` / `document`.
