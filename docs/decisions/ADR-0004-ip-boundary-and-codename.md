# ADR-0004: IP boundary and the retro-maze codename

**Status:** Accepted — 2026-10-16.

## Context

The repository is public from its creation (owner rule). The genre's reference game, its name,
characters, sprites, sounds and maze are protected intellectual property of their owner, and
its publisher is known to enforce it. Game rules and mechanics are not protected; their
expression is.

Options: **A** neutral codename `retro-maze`, every asset and name original; **B** a title
punning on the reference; **C** choose the final commercial title now.

## MCDA

| Criterion | Weight | A Codename + originals | B Pun title | C Final title now |
|---|---|---|---|---|
| Legal / IP risk | 35 | 5 | 1 | 4 — clearance needed early |
| Consistency with retro-shmup naming | 20 | 5 | 2 | 2 |
| Discoverability | 15 | 2 | 5 | 4 |
| Reversibility | 20 | 4 — rename the repo later | 3 | 1 |
| Effort now | 10 | 5 | 4 | 2 |
| **Total / 500** | 100 | **435** | 250 | 280 |

Points: A = 175 + 100 + 30 + 80 + 50; B = 35 + 40 + 75 + 60 + 40; C = 140 + 40 + 60 + 20 + 20.

Sensitivity: C overtakes A only if *Discoverability* outweighs legal risk — never for a public repository.

## Decision

- Codename **retro-maze** until a commercial title is chosen (repository renamed then).
- Original cast (Pip; Ember, Bloom, Tide, Dusk), original maze (GDD §4), original art, sound
  and bonus items. No reference name, nickname, logo, sprite, sound or maze anywhere in the
  repository — commit messages and issue titles included. The behavioural reference is cited
  by title and author (GDD §2), never used as a name for anything of ours.
- README disclaims any affiliation.

## Consequences

- Reviews check the IP rule like a security rule (CLAUDE.md non-negotiable).
- Marketing cannot lean on the reference's name.
