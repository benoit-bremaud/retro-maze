# ADR-0002: Arcade-faithful rules, without the original bugs

**Status:** Accepted — 2026-10-16.

## Context

The genre's appeal rests on learnable ghost behaviour: players expect cornering, the four
personalities, scatter waves and Elroy to work as in the 1980 reference, documented in detail
by The Pac-Man Dossier (Pittman). The reference also contains defects: the up-direction
targeting overflow, ghosts passing through the hero on a swap, the global dot counter that is
never deactivated, the level-256 split screen, ROM-seeded pseudo-random fright turns.

Options: **A** faithful rules and tables, bugs removed; **B** bit-exact reproduction, bugs
included; **C** loosely inspired rules tuned by feel.

## MCDA

| Criterion | Weight | A Faithful, no bugs | B Bit-exact | C Inspired |
|---|---|---|---|---|
| Player expectations / classic strategies work | 30 | 5 | 4 — the bugs surprise new players | 2 |
| Determinism and testability (GDD is the oracle) | 25 | 5 — every rule written and seeded | 3 — bugs need emulator-level detail | 3 — no oracle, tuning drifts |
| Perceived quality | 20 | 5 | 3 | 3 |
| Effort | 15 | 2 — all tables plus bug analysis | 1 — reverse-engineering | 5 |
| IP safety | 10 | 5 — rules only | 4 — invites ROM-level copying | 5 |
| **Total / 500** | 100 | **455** | 310 | 320 |

Points: A = 150 + 125 + 100 + 30 + 50; B = 120 + 75 + 60 + 15 + 40; C = 60 + 75 + 60 + 75 + 50.

Sensitivity: C wins only if *Effort* becomes the dominant criterion.

## Decision

Implement the GDD tables (§5–§9) as written. The removed bugs are listed in GDD §11; behaviour
that looks like a quirk but is a rule players rely on (no-up zones, 1/60 s scatter reversal,
Elroy chasing in scatter, cornering) is kept.

## Consequences

- The GDD is the test oracle; each table becomes a data table with unit tests.
- Speedrunners' exact patterns from the reference will not transfer (different maze anyway).
- Any reintroduced quirk needs a GDD revision.
