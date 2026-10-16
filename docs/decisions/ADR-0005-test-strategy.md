# ADR-0005: Test strategy

**Status:** Accepted — 2026-10-16.

## Context

The game is a set of tables and deterministic rules (GDD §5–§9) driven by a seeded RNG and a
fixed timestep. The owner's testing doctrine is the test pyramid, FIRST unit tests, mocks only
at boundaries. retro-shmup's pyramid (domain TDD, headless simulations, one browser smoke test
in three engines) has proven itself.

Options: **A** the retro-shmup pyramid; **B** golden traces recorded from an emulator of the
reference; **C** mostly browser end-to-end tests.

## MCDA

| Criterion | Weight | A Pyramid | B Emulator golden traces | C Mostly E2E |
|---|---|---|---|---|
| Confidence in rules | 30 | 4 — GDD tables as oracle | 5 | 3 |
| Speed / feedback | 20 | 5 | 3 | 1 |
| IP / legal safety | 15 | 5 | 1 — needs the original ROM | 5 |
| Maintenance | 20 | 5 | 2 — different maze breaks traces | 2 |
| Reuse from retro-shmup | 15 | 5 | 1 | 3 |
| **Total / 500** | 100 | **470** | 280 | 270 |

Points: A = 120 + 100 + 75 + 100 + 75; B = 150 + 60 + 15 + 40 + 15; C = 90 + 20 + 75 + 40 + 45.

## Decision

1. **Domain unit tests, TDD (Red / Green / Refactor):** every GDD table (speeds, schedule,
   fright, dot counters, Fury, fruit) is data with a test per row family; targeting, the
   tie-break order, no-up zones, forced reversal, cornering, swap collisions.
2. **Maze tests:** the parsed maze is symmetric, connected, has no dead end and holds 274
   pellets.
3. **Headless simulation tests:** a seed plus a scripted input sequence run N steps and assert
   events (ghost releases at the right pellet counts, Fury thresholds, level clear), and that
   two runs produce identical state hashes.
4. **One browser smoke test** (Playwright, Chromium / Firefox / WebKit): boot, start, move,
   persistence of the top 10 across a reload.
5. Render and audio mocked at their ports; the domain is never mocked.
6. Coverage ≥ 80 % (target ~100 %), doc coverage ≥ 90 % — the owner's pre-merge gate.

## Consequences

- No emulator or ROM is ever needed.
- The GDD must stay precise enough to be an oracle; ambiguities are fixed in the GDD first.
