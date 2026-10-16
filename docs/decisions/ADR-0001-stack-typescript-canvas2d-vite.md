# ADR-0001: Stack — TypeScript strict, native Canvas 2D, Vite

**Status:** Accepted — 2026-10-16.

## Context

retro-maze is a 224 × 288 tile-based game with six moving actors. Its hard requirements
(CLAUDE.md, GDD §12) are a fixed 60 Hz deterministic simulation, a domain that never imports
the browser, no allocation in the loop, and headless simulation tests. The sibling project
retro-shmup already ships a TypeScript + Canvas 2D + Vite stack that meets the same
requirements, with generic modules ready to copy (ADR-0003).

Options considered: **A** TypeScript strict + native Canvas 2D + Vite; **B** Phaser 3 (TS);
**C** Godot 4 with web export.

## MCDA

Scores 0–5. Weights proposed in brick 0, validated by the owner at the study PR review.

| Criterion | Weight | A Canvas 2D | B Phaser | C Godot |
|---|---|---|---|---|
| Determinism / fixed-step control | 25 | 5 — we own the loop | 4 — fixed step available, engine timers to avoid | 4 — `_physics_process` fixed, engine owns the loop |
| Reuse of retro-shmup modules | 20 | 5 — direct copy | 3 — ports re-wrapped around the engine | 0 — different language |
| Bundle size / first load | 10 | 5 — tens of kB | 3 — ~1 MB | 0 — tens of MB of wasm |
| Headless testability (Vitest) | 20 | 5 — pure domain | 4 — domain testable, scene glue not | 1 — engine runtime required |
| Ecosystem / tooling for free | 25 | 3 — we write sprites, input, audio ourselves (already done once) | 5 | 5 |
| **Total / 500** | 100 | **450** | 395 | 265 |

Points: A = 125 + 100 + 50 + 100 + 75; B = 100 + 60 + 30 + 80 + 125; C = 100 + 0 + 0 + 20 + 125.

Sensitivity: B only overtakes A if *Ecosystem* dominates the other criteria combined — not the case for a project whose
tooling already exists in the sibling repo.

## Decision

TypeScript in strict mode, native Canvas 2D behind a `RenderPort`, Vite for dev server and
build, Vitest for tests, pnpm. No game framework or rendering library.

## Consequences

- Same toolchain as retro-shmup: one mental model, copyable modules and configs.
- We own sprite batching, input and audio — already written once.
- Adding a framework later requires a superseding ADR (CLAUDE.md *Forbidden*).
