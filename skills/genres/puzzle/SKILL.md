---
name: puzzle
description: Compositional template for puzzle games — defining rules and the core mechanic, grid/state representation, undo/redo and restart, deterministic step simulation, level design by teaching and twisting, difficulty curves, hint systems, solvers for validation and generation, level editor/data format, juice for satisfying solutions, and which skills to load. Use when building a logic, sokoban-like, physics, match-3, or rule-based puzzle game.
---

# Puzzle Game Template

## Core loop

Understand the rules → observe the level → form a hypothesis → test (move/act) → fail and learn or succeed → "aha!" → next level with a new twist. Fun = **insight**, not trial and error or execution difficulty.

## Systems & skill map

| System | Notes | Load skill |
|---|---|---|
| Rules engine | Pure, deterministic `step(state, action) -> state` | — |
| State representation | Grid of cells + entities; immutable or copy-on-step | — |
| Undo/redo & restart | History stack of states (or of actions) | — |
| Level data | Text/JSON format, editor | `level-design` |
| Solver | BFS/A* on state space to verify solvability and find minimal moves | `procedural-gen` |
| Hints | Tiered hints, optional skip | `game-ui-ux` |
| Feedback | Smooth tweened moves, satisfying completion, clear invalid-move feedback | `game-feel`, `audio-design` |
| Progress | Level select, completion/stars, save | `save-systems` |

## Deterministic rules core

```python
from dataclasses import dataclass, replace

@dataclass(frozen=True)
class State:
    player: tuple[int, int]
    boxes: frozenset[tuple[int, int]]

WALLS: set[tuple[int, int]] = ...
GOALS: frozenset[tuple[int, int]] = ...

def step(s: State, d: tuple[int, int]) -> State:
    nx, ny = s.player[0] + d[0], s.player[1] + d[1]
    if (nx, ny) in WALLS: return s
    if (nx, ny) in s.boxes:
        bx, by = nx + d[0], ny + d[1]
        if (bx, by) in WALLS or (bx, by) in s.boxes: return s
        return replace(s, player=(nx, ny), boxes=(s.boxes - {(nx, ny)}) | {(bx, by)})
    return replace(s, player=(nx, ny))

def solved(s: State) -> bool: return s.boxes == GOALS
```

Keep the rules engine **separate from presentation**: the view animates the transition between states. This gives you undo, replays, solvers, and tests for free.

## Undo

`history.append(state)` before each step; undo pops. Unlimited undo + instant restart is expected — never punish experimentation.

## Level design for puzzles

- Each level should have **one core insight**. Remove everything not needed for it.
- Teach a mechanic in isolation, then combine with previous mechanics, then invert expectations.
- Small levels beat big ones; big levels often just add tedium.
- Order levels by playtest solve times, not designer intuition; let players skip/choose among a few unlocked levels to avoid hard walls.

## Solver for validation

```python
from collections import deque
def solve(start, max_states=1_000_000):
    seen, q = {start: None}, deque([start])
    while q and len(seen) < max_states:
        s = q.popleft()
        if solved(s): return reconstruct(seen, s)
        for d in DIRS:
            n = step(s, d)
            if n not in seen:
                seen[n] = (s, d); q.append(n)
    return None
```

Run the solver in CI over every level: assert solvable, record minimal move count (for par/star ratings), detect unintended shortcuts (solution much shorter than intended).

## Hints

Tier 1: point to the relevant object/area. Tier 2: describe the idea. Tier 3: show the next move(s) from the solver. Hints should cost nothing or little; frustration → quitting.

## Feel

- Moves tween quickly (60–120 ms) with easing; inputs during a tween are buffered.
- Distinct sounds for move, push, blocked, goal reached; a celebratory but short completion.
- Invalid moves: small bump animation + soft sound (not an error buzzer).

## Formats & editor

ASCII levels are great for grid puzzles:

```
#######
#.@ $ #
#  #  #
#######
```

Build an in-game editor early (place tiles, test-play, run solver) — iteration speed is everything.

## Scope tiers

- **Jam**: 1 mechanic, 10–15 levels.
- **Small**: 3–5 mechanics, 80–150 levels across worlds, each world centered on a mechanic.
