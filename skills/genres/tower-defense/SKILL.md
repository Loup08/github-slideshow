---
name: tower-defense
description: Compositional template for tower defense games — paths vs mazing, wave design and spawn tables, enemy types and armor, tower types/targeting priorities/upgrades, economy (kill gold, interest, sell refund), flow fields for pathing, projectiles and damage types, balancing with DPS/HP budgets, UI for placement, and which engine/discipline skills to load. Use when building a tower defense or lane defense game.
---

# Tower Defense Template

## Core loop

Earn currency → place/upgrade towers → survive wave → earn more → harder wave → … → final wave/boss. Fun = **planning + optimization** with escalating pressure and **satisfying chain reactions**.

## Systems & skill map

| System | Notes | Load skill |
|---|---|---|
| Map & paths | Fixed path(s) (spline/waypoints) or open-field mazing (blocking allowed if a path remains) | `level-design`, `procedural-gen` |
| Enemy movement | Waypoint follow; for mazing use a **flow field** recomputed on placement | `game-ai` |
| Waves | Data-driven wave tables | engine data skill |
| Towers | Range, fire rate, damage type, targeting mode, upgrade tree | engine data skill |
| Projectiles | Homing vs ballistic vs instant; splash; pooling | `performance-optimization` |
| Economy | Start gold, kill bounty, wave-clear bonus, sell refund (50–80%) | — |
| UI | Placement ghost with range circle, valid/invalid tint, upgrade panel, wave preview, speed toggle | `game-ui-ux` |
| Feel | Hit flashes, death pops, gold popups, wave start sting | `game-feel`, `audio-design` |

## Wave data

```json
{ "waves": [
  { "groups": [ { "enemy": "grunt", "count": 10, "interval": 0.8, "delay": 0 } ] },
  { "groups": [ { "enemy": "grunt", "count": 12, "interval": 0.7, "delay": 0 },
                { "enemy": "runner", "count": 6, "interval": 0.4, "delay": 5 } ] },
  { "groups": [ { "enemy": "tank", "count": 2, "interval": 3.0, "delay": 0, "boss": true } ] }
]}
```

Endless mode: HP scales `base * 1.1^wave`, counts rise, new types every N waves.

## Enemy roles

Grunt (baseline), Runner (fast, low HP — tests coverage), Tank (high HP/armor — tests DPS), Flyer (ignores path — needs anti-air), Swarm (many weak — splash), Healer/Shielder (priority targets), Splitter (spawns on death), Boss.

## Tower roles

Single-target DPS, Splash/AoE, Slow/control, Anti-armor (pierce), Anti-air, Support (buff range/damage), Economy (generates gold). Each role should be the best answer to some enemy type.

## Targeting modes

First (furthest along path — default), Last, Strongest, Weakest, Closest. Compute "progress along path" per enemy for First/Last.

```python
def acquire(tower, enemies):
    in_range = [e for e in spatial_grid.query(tower.pos, tower.range) if e.alive]
    if not in_range: return None
    return {"first": max, "last": min}.get(tower.mode, max)(in_range, key=lambda e: e.path_progress)
```

Use a spatial grid for range queries when enemy counts are high.

## Balancing

- Compute **DPS budget**: total enemy HP per wave / time the wave spends in kill zones ≈ required DPS; compare to DPS the player could afford with income up to that wave.
- Upgrades: cost grows ~1.5–2× per level; damage ~1.4–1.8× — upgrading should compete with building new.
- Interest (optional): % of banked gold per wave rewards greed vs safety.
- Lives/leaks: each leaked enemy costs lives (bosses cost more).

## Flow fields (mazing)

BFS/Dijkstra from the goal over the grid; each cell stores direction to the next cell. On placement, recompute; **reject placement** if the spawn can no longer reach the goal.

## Scope tiers

- **Jam**: 1 map, 4 towers, 5 enemy types, 15 waves.
- **Small**: 10–20 maps, 8–12 towers with 3 upgrades/branches, 12+ enemies, difficulty modes, star ratings.

## Testing

Speed-up (2×/4×) in debug, auto-play bots that place towers by simple heuristics to verify waves are winnable, telemetry on which towers are used (dominant strategies → rebalance).
