# Project Log — retro-maze

Reverse-chronological operational logbook (most recent first). It complements `git log` with the
human context: what was done, why, and by which PR.

---

## 2026-10-16

### Brick 0 — design study (branch `docs/study`)

- Owner decision (MCDA, 2026-10-16): a maze-chase arcade game under the codename retro-maze,
  sibling of `../retro-shmup`. Only the study now; implementation waits for retro-shmup's
  vertical slice, to avoid two half-finished games.
- Repository skeleton: README, MIT LICENSE, CLAUDE.md (non-negotiables carried over from
  retro-shmup), `.gitignore`, this log.
- GDD v0.1: arcade rules after The Pac-Man Dossier (Pittman), with the original bugs removed;
  our own 28 × 31 maze (270 dots + 4 energizers, validated by script: fully connected, no dead
  end), renamed cast (hero Pip; ghosts Ember, Bloom, Tide, Dusk).
- ADR-0001 stack (TS + Canvas 2D + Vite, 450 / 500 vs Phaser 395, Godot 265), ADR-0002
  fidelity without bugs (455), ADR-0003 copy & adapt from retro-shmup `f698786` (430),
  ADR-0004 IP-safe codename (435), ADR-0005 test strategy.
- UML: use cases, game state machine, ghost mode state machine, domain class diagram,
  component diagram.
- Pending: pre-push review, owner approval to create the public GitHub repository and the PR.
