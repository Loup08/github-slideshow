---
name: survival-crafting
description: Compositional template for survival and crafting games — needs (hunger, thirst, temperature, stamina), resource gathering and nodes, inventory and stacking, crafting recipes and stations, tech progression, building/placement on grids, day-night cycle and threats, world generation, persistence of a modified world, and multiplayer considerations, plus which skills to load. Use when building a survival, crafting, base-building, or sandbox game.
---

# Survival & Crafting Template

## Core loop

```
Explore → gather resources → craft tools/structures → survive threats (needs, night, enemies)
→ unlock better recipes/biomes → set bigger goals (bosses, tech tiers, escape)
```

Fun = **self-directed goals**, **tangible progress** (my base got better), and **tension** from threats. Early game must give a clear short-term objective chain.

## Systems & skill map

| System | Notes | Load skill |
|---|---|---|
| Needs | Hunger/thirst/temperature/sanity drain over time; low values apply debuffs before death | — |
| Gathering | Resource nodes (trees, rocks) with HP, tool tiers, drop tables, respawn rules | `procedural-gen` |
| Inventory | Slots, stacking (max per item), hotbar, drag/split | `game-ui-ux` |
| Crafting | Data-driven recipes, stations, unlocks | engine data skill |
| Building | Grid/snap placement, validity checks, structural support (optional) | `level-design` |
| Day/night & weather | Lighting cycle, enemy spawns at night, temperature effects | `godot-3d-essentials`, `threejs-materials-lighting`, `shader-programming` |
| World | Procedural terrain/biomes, chunk streaming | `procedural-gen`, `performance-optimization` |
| Enemies/creatures | Passive, neutral, hostile; spawn by biome/time | `game-ai` |
| Persistence | World modifications (cut trees, built structures, chest contents) per chunk | `save-systems` |

## Data examples

```json
{ "items": { "wood": { "stack": 99 }, "stone": { "stack": 99 }, "axe_stone": { "stack": 1, "tool": "axe", "tier": 1, "durability": 80 } },
  "recipes": [
    { "id": "axe_stone", "station": null, "in": { "wood": 3, "stone": 2 }, "out": { "axe_stone": 1 }, "time": 1.5 },
    { "id": "campfire", "station": null, "in": { "wood": 5, "stone": 3 }, "out": { "campfire": 1 } },
    { "id": "iron_ingot", "station": "furnace", "in": { "iron_ore": 2, "coal": 1 }, "out": { "iron_ingot": 1 }, "time": 5 }
  ] }
```

```python
def can_craft(recipe, inv, nearby_stations):
    return (recipe.station is None or recipe.station in nearby_stations) and \
           all(inv.count(i) >= n for i, n in recipe.inputs.items())
```

Recipes discovered by picking up ingredients (show what you could craft) reduce wiki-dependence.

## Needs tuning

- Drain rates so a "day" (~10–20 real minutes) requires 1–2 meals.
- Warnings at 30% and 10% (UI pulse, sound, character bark).
- Penalties escalate (slower stamina regen → health drain) — avoid instant death from needs.
- Comfort/food variety buffs reward engaging with systems.

## Building

Grid snapping (1 m or tile-based), ghost preview with valid/invalid color, rotation, collision/overlap checks, ownership (multiplayer), repair/destroy with partial refund. Store structures as lightweight data (type, cell, rotation, hp) — not full scene snapshots.

## Progression

Tech tiers gated by resources from riskier biomes (wood/stone → copper → iron → …). Each tier unlocks a new verb (better storage, farming, transport) not just bigger numbers. Bosses as milestones.

## World persistence

Chunk-based: store deltas from the procedural baseline (`removed_nodes`, `placed_structures`, `dropped_items`) keyed by chunk coords + seed. Load/unload with the player's proximity.

## Multiplayer

Massive complexity multiplier: server authority for inventories (no duplication), replicated world deltas, persistent dedicated servers. Prototype single-player with a clean separation of state and presentation first.

## Scope tiers

- **Jam**: small fixed island, 3 needs, 10 recipes, 1 night threat, survive N days goal.
- **Small**: 3 biomes, 50–80 items, 3 tech tiers, base building, 2 bosses.

## Testing

Economy sanity (time to first tool < 2 minutes), new-player onboarding test without wiki, save/load of a heavily modified world, performance with large bases.
