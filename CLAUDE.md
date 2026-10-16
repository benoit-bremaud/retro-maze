# CLAUDE.md — retro-maze

## Project overview

**Name:** retro-maze (codename, ADR-0004 — the commercial title is still open).
**Type:** maze-chase arcade game for the browser, in the tradition of the 1980 coin-op genre,
with original maze, characters, art and sound. TypeScript strict + native Canvas 2D + Vite
(ADR-0001), 28 × 31 tile maze, 8 px tiles, 224 × 288 logical screen.
**Status:** **brick 0 — design study** (GDD, ADR-0001..0005, UML study). No game code until the
sibling project `../retro-shmup` has shipped its vertical slice (owner decision 2026-10-16).
The design contract is [docs/design/game-design-document.md](docs/design/game-design-document.md);
technical decisions are ADRs under [docs/decisions/](docs/decisions/); the UML study lives under
[docs/architecture/](docs/architecture/README.md).
**Language:** English only in the repository (code, comments, commits, docs, issues, PRs).

Global rules (Conventional Commits, branch from `main`, no `any`, no default exports, pre-push
review gate, MCDA for every choice, security defaults) are inherited from `~/.claude/CLAUDE.md`
and `~/.agent-rules/common.md` — not duplicated here. Below are the rules that apply **only** to
this repository.

## Non-negotiables (project)

- **No third-party intellectual property (ADR-0004).** No original-game name, logo, sprite,
  sound, music, maze layout or character name — in code, assets, docs, commit messages or issue
  titles. Arcade *rules* are described from the public behavioural reference (The Pac-Man
  Dossier, cited in the GDD); everything shown or heard is our own.
- **Design before code.** A feature is implemented only after (1) its rules exist in the GDD and
  (2) its UML diagrams are validated. Diagrams are Mermaid under
  `docs/architecture/diagrams/<feature>/`.
- **Validated decisions are not reopened silently.** Changing an ADR or a GDD rule requires a
  superseding ADR or a GDD revision in its own PR.
- **Arcade fidelity without the original bugs (ADR-0002).** Timings and tables follow the GDD;
  the known arcade defects (up-direction target offset, pass-through collisions, level-256 split
  screen) are deliberately absent.
- **The domain never imports the browser.** Gameplay code depends on `RenderPort`, `AudioPort`,
  `InputPort`, `StoragePort` and a clock abstraction; `canvas`, `window`, `AudioContext`,
  `localStorage` appear only in adapters.
- **Fixed 60 Hz timestep, deterministic simulation.** Accumulator loop, seeded RNG (frightened
  ghosts' random turns included), no `Date.now()` / `Math.random()` inside the simulation. The
  same seed and input script must replay identically.
- **No allocation inside the game loop.** Fixed-size entity arrays (one hero, four ghosts, one
  fruit), preallocated vectors, index loops only (`for...of` banned in `src/`).
- **Every player-facing string goes through the message catalogue** (EN default, FR).
- **Every game-feel effect is toggleable** and respects the photosensitivity options (frightened
  flashing included).
- **Tests (ADR-0005):** domain unit tests (TDD), headless maze-simulation tests with scripted
  inputs and a fixed seed, exactly one browser smoke test. Render and audio are mocked at their
  ports, never the domain.

## Repository layout

```
docs/design/        GDD and design annexes
docs/decisions/     ADRs (Nygard format, ADR-NNNN-<slug>.md)
docs/architecture/  UML diagrams (Mermaid)
src/domain/         pure gameplay rules, ports (no browser API)     — later bricks
src/adapters/       browser implementations of the ports           — later bricks
src/app/            composition root and frame loop                — later bricks
tests/              Vitest, mirroring src/                         — later bricks
```

## Commit scopes (Conventional Commits)

`design` (GDD), `adr`, `uml`, `docs`, `ci`, `repo`, and — once code exists — `engine`, `game`,
`ghosts`, `maze`, `render`, `audio`, `input`, `ui`, `assets`, `i18n`, `tests`.

Branch naming: `feat/<scope>`, `fix/<scope>`, `refactor/<scope>`, `chore/<scope>`, `docs/<scope>`.

## Modules copied from retro-shmup (ADR-0003)

Generic modules are copied and adapted, not shared. Each copy records the retro-shmup source SHA
in `docs/decisions/ADR-0003-*.md`. A shared package is considered only when a third game needs
them.

## Workflow reminders

- Open a branch for every change; `main` is protected once the GitHub repository exists.
- Run the local pre-push review gate (`review-local`) before every push.
- After every merged PR: update `PROJECT_LOG.md`, post-merge cleanup, memory compression.
- The repository is **public** from creation (owner rule); never commit anything private.

## Forbidden — never without explicit owner request

- Adding a game framework or rendering library (Phaser, PixiJS…) — ADR-0001.
- Any third-party IP asset or name — ADR-0004.
- Any backend, account system or network call in 1.0.
- Starting implementation before retro-shmup's vertical slice is done.
