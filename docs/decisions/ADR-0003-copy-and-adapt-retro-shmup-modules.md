# ADR-0003: Copy and adapt the generic retro-shmup modules, no shared package yet

**Status:** Accepted — 2026-10-16.

## Context

retro-shmup contains generic, tested modules: the frame loop, the clock, the seeded RNG, the
ports, input with rebinding, save, hall of fame, name entry, i18n and the audio synth. retro-maze
needs the same. Sharing code between two repositories can be done by copying, by a shared
package, or by rewriting.

Options: **A** copy and adapt, recording the source SHA; **B** extract a shared npm package now;
**C** rewrite from scratch.

## MCDA

| Criterion | Weight | A Copy & adapt | B Shared package | C Rewrite |
|---|---|---|---|---|
| Time to first playable | 25 | 5 | 3 — package, versioning, publish first | 1 |
| Decoupling (one game's change never breaks the other) | 25 | 5 | 2 | 5 |
| One source of truth (DRY) | 20 | 2 — two copies drift | 5 | 1 |
| Fit to maze needs (free to adapt) | 20 | 5 | 3 — generic API compromises | 4 |
| Simplicity (KISS / YAGNI) | 10 | 4 | 2 | 3 |
| **Total / 500** | 100 | **430** | 305 | 280 |

Points: A = 125 + 125 + 40 + 100 + 40; B = 75 + 50 + 100 + 60 + 20; C = 25 + 125 + 20 + 80 + 30.

Sensitivity: B overtakes A only if *DRY* becomes the dominant criterion. With a third game, DRY's weight grows
and this ADR is to be revisited (rule of three).

## Decision

Copy from retro-shmup at commit **`f698786ac751d3de73c38e73498a76630666092f`** (main,
2026-10-16), with their tests, when the corresponding brick starts:

| Module | Source path in retro-shmup |
|---|---|
| frame-loop | `src/app/frame-loop.ts` |
| performance-clock | `src/adapters/performance-clock.ts` |
| random | `src/domain/random/seeded-random.ts` |
| ports | `src/domain/ports/{audio-port,clock,input-port,random,render-port,storage-port}.ts` |
| guards | `src/domain/guards.ts` |
| canvas2d-renderer | `src/adapters/canvas2d-renderer.ts` |
| viewport | `src/adapters/viewport.ts` |
| sprite-atlas | `src/adapters/sprite-atlas.ts` |
| input | `src/adapters/input/*.ts`, `src/domain/input/*.ts` |
| save | `src/domain/save/save-store.ts` |
| hall-of-fame | `src/domain/hall-of-fame/*.ts` |
| name-entry | `src/domain/scenes/name-entry.ts` |
| i18n | `src/domain/i18n/messages.ts` |
| audio synth | `src/domain/audio/synth.ts` |
| web-audio-player | `src/adapters/web-audio-player.ts` |

If a later brick copies from a newer SHA, it appends a row to a *Copy log* section of this ADR.
Shmup-specific content (messages, bindings, sfx) is replaced, not kept.

## Consequences

- Fast start, no cross-repo release process.
- Fixes found in one copy are ported by hand; the PROJECT_LOG of each repo notes it.
- No shared package until a third game needs these modules.
