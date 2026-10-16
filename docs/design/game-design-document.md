# retro-maze — Game Design Document

**Version:** 0.1 — 2026-10-16 (brick 0, design study).
**Status:** draft for owner review.

This document is the design contract: no rule is implemented before it is written here, and
the code follows these tables. Technical choices live in the ADRs under
[`docs/decisions/`](../decisions/); the UML study under
[`docs/architecture/`](../architecture/README.md).

## 1. Vision

A single-screen maze-chase arcade game for the browser. The player steers a hungry hero through
a maze, eats every dot to clear the level, and dodges four ghosts whose distinct personalities
make the chase readable and learnable. An energizer turns the tables for a few seconds.

Pillars:

1. **Arcade-faithful rules.** Movement, ghost AI, timings and tables reproduce the behaviour of
   the 1980 arcade reference, as documented by the community, so that classic strategies
   (cornering, patterns, ghost manipulation) work.
2. **Without the bugs.** The known arcade defects are removed (ADR-0002, §11).
3. **Original identity.** Our own maze, cast, art, sound and name (ADR-0004).
4. **Deterministic and testable.** Fixed 60 Hz simulation and a seeded RNG: the same inputs
   replay the same game (CLAUDE.md non-negotiables, ADR-0005).

## 2. Behavioural reference and IP boundary

The behavioural reference is **Jamey Pittman, *The Pac-Man Dossier*** (2009, revised;
<https://www.gamedeveloper.com/design/the-pac-man-dossier>, mirrored at
<https://pacman.holenet.info/>). The numbers in §5–§9 come from it unless marked *(ours)*.

Game rules and mechanics are not protected by copyright; their expression is. Therefore:

- **Reproduced:** rules, timings, speed percentages, targeting logic, counters.
- **Never reproduced:** the original title, logo, character names and nicknames, sprites,
  sounds, music, maze layout, cut-scenes, fruit art. Our maze (§4) and cast (§3) are original.

## 3. Cast *(ours)*

| Role | Name | Colour | Personality (§6.3) |
|---|---|---|---|
| Hero | **Pip** | yellow-green | — |
| Ghost 1 — chaser | **Ember** | red | targets the hero's tile |
| Ghost 2 — ambusher | **Bloom** | pink | targets 4 tiles ahead of the hero |
| Ghost 3 — flanker | **Tide** | cyan | vector through Ember and a point 2 ahead |
| Ghost 4 — shy | **Dusk** | orange | chases far away, retreats when close |

Ember starts outside the house; Bloom, Tide and Dusk start inside (§7).

Colours above are role placeholders. The assets brick picks a palette and silhouettes
distinct from the reference's trade dress (ADR-0004).

## 3.1 Screen

- Logical screen **224 × 288 px** = 28 × 36 tiles of **8 × 8 px**.
- Rows 0–2: score and high score. Rows 3–33: the **28 × 31 maze**. Rows 34–35: lives and
  level fruit.
- Integer-scaled to the window, letterboxed (viewport module from retro-shmup, ADR-0003).

## 4. Maze *(ours)*

Coordinates below are maze tiles `(col, row)`, origin top-left of the maze, 0 ≤ col < 28,
0 ≤ row < 31.

```
############################   0
#......#.....##.....#......#   1
#.####.#.###.##.###.#.####.#   2
#o####.#.###.##.###.#.####o#   3
#..........................#   4
#.##.###.##########.###.##.#   5
#.##.###.##########.###.##.#   6
#....#......####......#....#   7
####.#.####.####.####.#.####   8
####.#.####.####.####.#.####   9
####.#......    ......#.####  10
####.### ###----### ###.####  11
####.### #        # ###.####  12
    .    #        #    .      13  ← tunnel row (wraps)
####.### #        # ###.####  14
####.### ########## ###.####  15
####.#                #.####  16
####.#.##############.#.####  17
####.#.##############.#.####  18
#............##............#  19
#.##.####.##.##.##.####.##.#  20
#o##.####.##.##.##.####.##o#  21
#...........    ...........#  22
###.##.#.##########.#.##.###  23
###.##.#.##########.#.##.###  24
#......#....####....#......#  25
#.###.#####.####.#####.###.#  26
#.###.#####.####.#####.###.#  27
#..........................#  28
############################  29
############################  30
```

Rows 29–30 are a solid bottom margin (keeps the maze at the reference's 31 rows).
Legend: `#` wall, `.` dot, `o` energizer, space = corridor without dot, `-` ghost-house door
(passable only by ghosts leaving or returning as eyes).

Properties (validated by script in brick 0; to be re-asserted by a unit test on the parsed
maze in the maze brick):

- Mirror-symmetric, every walkable tile reachable, **no dead end**, corridors one tile wide
  outside the house.
- **270 dots + 4 energizers = 274 pellets.** (The reference maze has 244; thresholds below
  that count *eaten* pellets keep the reference values, thresholds that count *remaining*
  pellets — Fury — keep them too, since both express the same pacing.)
- **Ghost house:** interior rows 12–14, cols 10–17; door row 11, cols 12–15. Exit tile: row
  10, between cols 13 and 14.
- **Hero start:** row 22, between cols 13 and 14, facing left.
- **Tunnel:** row 13, wraps from col 0 to col 27. **Tunnel slow zone:** cols 0–5 and 22–27 of
  row 13 (ghost speed, §5).
- **No-up zones** (ghosts in scatter or chase may not choose *up* there; frightened ghosts ignore them, §6.1):
  row 10 cols 12–15 and row 22 cols 12–15 — the two horizontal corridors directly above the
  house door and above the hero start, matching the reference's purpose (protect the house
  exit and the start).

## 5. Movement

### 5.1 Speeds

100 % = 75.757 576 px/s (reference full speed), i.e. 1.262 6 px per 60 Hz step. Movement is
integrated in fixed point (1/256 px) so that movement is deterministic (sub-pixel remainders carried, never rounded away).

| Levels | Hero | Hero eating dots | Ghost | Ghost in tunnel | Hero frightened | Hero frightened eating | Ghost frightened |
|---|---|---|---|---|---|---|---|
| 1 | 80 % | 71 % | 75 % | 40 % | 90 % | 79 % | 50 % |
| 2–4 | 90 % | 79 % | 85 % | 45 % | 95 % | 83 % | 55 % |
| 5–20 | 100 % | 87 % | 95 % | 50 % | 100 % | 87 % | 60 % |
| 21+ | 90 % | 79 % | 95 % | 50 % | — | — | — |

Eaten ghosts (eyes) move at 200 % *(reference: fastest speed in the game; exact value unknown,
ours)*. Fury speeds: §8.

The reference expresses the "eating dots" slowdown as the hero **skipping frames**: one frame
per dot, three per energizer. We implement that literal rule (1 and 3 steps of no movement);
the table's "eating" column is the resulting average and documents it.

### 5.2 Hero — cornering

- The hero keeps moving in its current direction until blocked by a wall; it never stops on
  its own except against a wall.
- The desired direction (latest input) is buffered and applied as soon as it becomes legal.
- **Cornering (pre-turn / post-turn):** when the desired direction is perpendicular and the
  target tile in that direction is open, the hero may start the turn up to **4 px before** the
  tile centre (pre-turn) or finish it up to **3 px after** (post-turn) *(ours, to tune: the
  reference's window depends on speed)*, moving diagonally — the
  movement on both axes at full speed — until it is centred on the new axis. This makes
  cornering faster than the ghosts' square turns, as in the reference.
- Reversal is instantaneous and always allowed.

### 5.3 Ghosts — direction choice

- A ghost decides its next direction **one tile in advance, when it enters a tile** (its
  centre crosses the tile boundary): it looks at the tile it is about to enter and picks the
  exit from there.
- Candidates: the three non-reverse directions whose next tile is walkable. **Never reverse**
  by choice.
- Among candidates, pick the one whose next tile minimises the **Euclidean distance** (squared,
  integers) to the current target tile (§6).
- **Tie-break order: up, left, down, right.**
- **No-up zones** (§4) remove *up* from the candidates in chase and scatter.
- **Forced reversal:** on every mode change (scatter ↔ chase, and entering frightened) every
  ghost outside the house reverses at its next tile entry. Leaving frightened does *not*
  reverse.
- Ghosts in the tunnel slow zone use the tunnel speed.

## 6. Ghost behaviour

### 6.1 Modes

- **Scatter:** target the ghost's fixed corner (§6.2); the ghost circles that corner.
- **Chase:** target computed from its personality (§6.3).
- **Frightened:** blue, slow; at each tile entry chooses a **pseudo-random** legal non-reverse
  direction from the seeded RNG (no-up zones ignored, as in the reference). Can be eaten.
- **Eaten (eyes):** target the house exit tile, then enter the house, revive, and leave.
- **In house / leaving:** bounce up and down until released (§7), then leave via the door.

### 6.2 Scatter corners *(ours, off-maze like the reference)*

| Ghost | Target tile |
|---|---|
| Ember | (25, −3) top right |
| Bloom | (2, −3) top left |
| Tide | (27, 31) bottom right |
| Dusk | (0, 31) bottom left |

### 6.3 Chase personalities

Let `H` be the hero's tile and `d` its current direction as a unit vector.

- **Ember (chaser):** target = `H`.
- **Bloom (ambusher):** target = `H + 4·d`. *Bug removed:* the reference also shifts 4 tiles
  left when `d` is up (an overflow); we do not.
- **Tide (flanker):** `P = H + 2·d`; target = `P + (P − E)` where `E` is Ember's tile — the
  vector from Ember to `P`, doubled. *Bug removed:* no extra left shift when `d` is up.
- **Dusk (shy):** if the squared distance from Dusk to `H` is ≥ 64 (8 tiles), target = `H`;
  otherwise target = its scatter corner.

### 6.4 Scatter / chase schedule

Durations in seconds; the timer pauses while frightened is active and during freeze frames.

| Phase | Level 1 | Levels 2–4 | Levels 5+ |
|---|---|---|---|
| Scatter 1 | 7 | 7 | 5 |
| Chase 1 | 20 | 20 | 20 |
| Scatter 2 | 7 | 7 | 5 |
| Chase 2 | 20 | 20 | 20 |
| Scatter 3 | 5 | 5 | 5 |
| Chase 3 | 20 | 1 033 | 1 037 |
| Scatter 4 | 5 | 1/60 | 1/60 |
| Chase 4 | ∞ | ∞ | ∞ |

The 1/60 s scatter is one step: its only effect is the forced reversal, kept on purpose.

### 6.5 Frightened

| Level | Fright time (s) | Flashes at the end |
|---|---|---|
| 1 | 6 | 5 |
| 2 | 5 | 5 |
| 3 | 4 | 5 |
| 4 | 3 | 5 |
| 5 | 2 | 5 |
| 6 | 5 | 5 |
| 7 | 2 | 5 |
| 8 | 2 | 5 |
| 9 | 1 | 3 |
| 10 | 5 | 5 |
| 11 | 2 | 5 |
| 12 | 1 | 3 |
| 13 | 1 | 3 |
| 14 | 3 | 5 |
| 15 | 1 | 3 |
| 16 | 1 | 3 |
| 17 | 0 | 0 |
| 18 | 1 | 3 |
| 19+ | 0 | 0 |

- Fright time 0: ghosts reverse but do not turn blue.
- A flash = 14 steps blue + 14 steps white *(ours: the reference value is not documented
  precisely)*. Under the reduced-flash option the white phase uses a low-contrast tint.
- Eating another energizer while frightened restarts the timer and resets the combo to
  200 (§9.2).
- Ghosts in the house during fright are frightened too (blue) but are not released faster.

## 7. Ghost house release

### 7.1 Personal dot counters

Each ghost in the house has a dot counter, active for **one ghost at a time** in the order
Bloom → Tide → Dusk. Each pellet eaten increments the active ghost's counter; when it reaches
the limit, the ghost leaves and the next one becomes active.

| Level | Bloom | Tide | Dusk |
|---|---|---|---|
| 1 | 0 | 30 | 60 |
| 2 | 0 | 0 | 50 |
| 3+ | 0 | 0 | 0 |

Counters are not reset when a level continues after a death — the global counter takes over
(§7.2).

### 7.2 Global counter after a death

After the hero loses a life, the personal counters are disabled and a single global counter
starts at 0 and counts pellets eaten:

- Bloom leaves at 7, Tide at 17, Dusk at 32.
- If Dusk is in the house when the counter reaches 32, the global counter is deactivated and
  the personal counters resume (Dusk's limit is then already met).
- *Bug removed:* in the reference, if Dusk had already left before 32, the global counter is
  never deactivated and the house can lock ghosts in. We deactivate it as soon as no ghost is
  waiting.

### 7.3 Idle timer

If the hero eats nothing for **4 s** (levels 1–4) or **3 s** (levels 5+), the next waiting
ghost is released and the timer resets. The timer resets on every pellet eaten.

## 8. Fury — two-stage speed-up of Ember

When the number of **remaining** pellets falls to the threshold, Ember changes speed, and in
Fury mode it **keeps targeting the hero during scatter** (as in the reference, a feature).

| Level | Fury 1: remaining ≤ | speed | Fury 2: remaining ≤ | speed |
|---|---|---|---|---|
| 1 | 20 | 80 % | 10 | 85 % |
| 2 | 30 | 90 % | 15 | 95 % |
| 3–4 | 40 | 90 % | 20 | 95 % |
| 5 | 40 | 100 % | 20 | 105 % |
| 6–8 | 50 | 100 % | 25 | 105 % |
| 9–11 | 60 | 100 % | 30 | 105 % |
| 12–14 | 80 | 100 % | 40 | 105 % |
| 15–18 | 100 | 100 % | 50 | 105 % |
| 19+ | 120 | 100 % | 60 | 105 % |

After a death Fury is suspended until Dusk has left the house.

## 9. Scoring, fruit, lives, levels

### 9.1 Pellets

Dot: 10 points. Energizer: 50 points.

### 9.2 Ghost-eating combo

Within one energizer's fright period: 200, 400, 800, 1 600. The combo restarts at 200 on every
energizer eaten (so 4 energizers × 3 000 = 12 000 points per level at most). Eating a ghost freezes the
game for 1 s while the points are displayed *(the reference shows the score; duration ours)*.

### 9.3 Fruit *(art and names ours)*

A bonus item appears below the house (row 16, between cols 13 and 14) after **70** and
**170** pellets eaten, and stays **9.5 s** *(the reference varies between 9 and 10 s;
we fix it for determinism)*.

| Level | Bonus | Points |
|---|---|---|
| 1 | Bonus 1 | 100 |
| 2 | Bonus 2 | 300 |
| 3–4 | Bonus 3 | 500 |
| 5–6 | Bonus 4 | 700 |
| 7–8 | Bonus 5 | 1 000 |
| 9–10 | Bonus 6 | 2 000 |
| 11–12 | Bonus 7 | 3 000 |
| 13+ | Bonus 8 | 5 000 |

The eight bonus items are drawn in the assets brick (our own designs, no fruit lookalike
required).

### 9.4 Lives and levels

- 3 lives at start; **one extra life at 10 000 points** (once).
- Collision: hero and a ghost on the **same tile**, or crossing each other within the same
  step (*bug removed:* the reference misses swaps and lets ghosts pass through). Checked after
  every hero move and every ghost move.
- Death: freeze 1 s, death animation, restart positions, global counter (§7.2).
- Level clear: all 274 pellets eaten → maze flash 2 s, next level, counters and Fury reset.
- No kill screen: levels continue indefinitely with the 21+ row (*bug removed:* level 256).
- Intermission cut-scenes: none in 1.0 (the reference's cut-scenes are expression, not rules).

## 10. Game flow and meta

Title → (attract demo, later) → Ready (2 s, the hero and ghosts visible) → Playing ⇄ Paused
→ Dying → Ready / Game over → name entry (if top 10) → Hall of fame → Title. Level clear →
Ready of the next level. Formalised in
[`docs/architecture/diagrams/game/02-state-game.md`](../architecture/diagrams/game/02-state-game.md).

Meta features copied from retro-shmup (ADR-0003): keyboard / gamepad / touch input with
rebinding, local save (options, top 10), name entry, EN/FR catalogue, synthesised audio.

Touch input *(ours)*: swipe sets the desired direction; no on-screen pad by default.

## 11. Original bugs deliberately removed

| Reference defect | Our rule |
|---|---|
| Up-direction target overflow (Bloom, Tide) | §6.3 — pure `H + n·d` |
| Pass-through on swap | §9.4 — swap detection |
| Level-256 split screen | §9.4 — no kill screen |
| Global counter never deactivated | §7.2 |
| Random fruit duration | §9.3 — fixed 9.5 s |
| Unseeded frightened turns (pseudo-random from ROM) | §6.1 — seeded RNG |

Kept on purpose because they are rules players rely on: no-up zones, the 1/60 s scatter
reversal, Fury chasing in scatter, cornering.

## 12. Non-functional

- 60 fps on a 2019 mid-range phone; ≤ 2 ms of logic per step; no allocation in the loop.
- Deterministic: a seed and an input script replay the same game (ADR-0005).
- Accessibility: reduced flash, remappable input, colour-blind-safe ghost outlines *(ours)*.

## Decision record

| Date | Decision | Ref |
|---|---|---|
| 2026-10-16 | Maze-chase genre, codename retro-maze, study only until retro-shmup's slice | owner, ADR-0004 |
| 2026-10-16 | Arcade-faithful rules without the bugs | ADR-0002 |
| 2026-10-16 | Own 28 × 31 maze, 274 pellets; cast Pip, Ember, Bloom, Tide, Dusk | §3, §4 |
