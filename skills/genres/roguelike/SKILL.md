---
name: roguelike
description: Compositional template for roguelikes and roguelites — run structure, permadeath and meta-progression, procedural floors and rooms, seeded RNG, item/relic synergies, enemy and loot tables scaled by depth, turn-based vs real-time, difficulty ramps, save/suspend, and which engine/discipline skills to load. Use when building a roguelike, roguelite, dungeon crawler, or run-based game.
---

# Roguelike / Roguelite Template

## Core loop

```
Start run (seed) → floor 1 … N (rooms: combat / treasure / shop / event / elite / boss)
→ collect items that synergize → die or win → meta-progression (roguelite) → new run
```

The fun: **decisions under uncertainty** and **build variety**. Each run must feel different and every death should feel like the player's fault (or lesson).

## Systems & skill map

| System | Notes | Load skill |
|---|---|---|
| Seeded RNG | Master seed → sub-streams per system (map, loot, combat) for reproducibility and daily runs | `procedural-gen` |
| Map generation | Room templates + procedural arrangement; or full procedural (BSP, CA) with validation | `procedural-gen`, `level-design` |
| Combat | Turn-based grid (classic) or real-time action (roguelite) | `game-ai`, `game-feel` |
| Items/relics | Data-driven definitions + effect hooks (on_hit, on_kill, on_room_enter) | engine data skill (`godot-resources`, `unity-scriptableobjects`) |
| Loot & enemy tables | Weighted, scaled by depth; rarity tiers | `procedural-gen` |
| Meta-progression | Unlock new items/characters, small permanent upgrades | `save-systems` |
| Run save/suspend | Save between rooms; delete suspend save on load if anti-scum desired | `save-systems` |
| UI | Minimap, item tooltips with synergy hints, run summary screen | `game-ui-ux` |

## Item effect architecture

```python
class Item:
    id: str
    def on_pickup(self, player): ...
    def on_hit(self, player, target, dmg): return dmg
    def on_kill(self, player, target): ...
    def on_room_clear(self, player): ...

# Player iterates items at each hook; stats as base + additive + multiplicative layers:
final = (base + sum(add)) * prod(1 + m for m in mult)
```

Event hooks + stat modifier layers make hundreds of items composable. Guard against infinite loops (an on_hit that triggers on_hit) with recursion depth/proc flags.

## Turn-based specifics

Energy/speed system (`actor.energy += speed; act when >= 100`), grid FOV (shadowcasting), A* pathfinding, message log, one action = one turn for all.

## Difficulty & pacing

- Depth-scaled enemy budget per room; elites/bosses as checkpoints of build strength.
- Guaranteed shops/healing at intervals; pity timers for rare drops.
- Ascension/heat levels for replay difficulty after the first win.
- First run should be beatable in principle; average first win around 5–20 hours for roguelites.

## Fairness rules

- Telegraph enemy intent (Slay-the-Spire-style intent icons or clear wind-ups).
- No unavoidable damage on room entry.
- Validate generated floors (exit reachable, keys before locks).
- Show seeds in the run summary.

## Scope tiers

- **Jam**: 1 character, 10–15 items, 3 floors, 4–6 enemies, no meta.
- **Small**: 2–3 characters, 60–100 items, 3 acts + boss each, light meta unlocks.
- Content scales with items/enemies — design the effect system first.

## Testing

Simulate thousands of seeds headless: generation invariants, item interactions not crashing, damage numbers not overflowing. Log run stats (floor reached, cause of death) during playtests to balance.
