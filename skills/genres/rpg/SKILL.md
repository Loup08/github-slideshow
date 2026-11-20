---
name: rpg
description: Compositional template for RPGs (action RPG, JRPG/turn-based, CRPG) — stats and formulas, leveling and XP curves, inventory and equipment, quests and flags, dialogue, combat (turn-based or real-time), party, shops and economy, save system, world structure, and which engine/discipline skills to load. Use when building any game with character progression, quests, and stats.
---

# RPG Template

## Core loop

Explore → fight/quest → gain XP/loot → improve character (levels, gear, skills) → access harder areas/story. Long-term motivation = **visible growth** + **story/world curiosity**.

## Systems & skill map

| System | Notes | Load skill |
|---|---|---|
| Stats & modifiers | Base + flat + % modifiers with sources (gear, buffs) | engine data skill |
| Leveling | XP curve, stat growth, skill points | — |
| Inventory/equipment | Item definitions (data assets) + instances (qty, durability, rolls) | `godot-resources`, `unity-scriptableobjects` |
| Combat | Turn-based (ATB/CTB/initiative) or action (hitboxes, i-frames) | `game-ai`, `game-feel` |
| Quests | State machine per quest + global flags | `dialogue-systems` |
| Dialogue | Branching with conditions on flags/stats | `dialogue-systems` |
| Shops/economy | Buy/sell ratios, gold sinks | — |
| Saving | Large state: flags, inventory, map states, party | `save-systems` |
| UI | Menus heavy: inventory, equip compare, quest log, map | `game-ui-ux` |
| World | Towns (safe), dungeons (danger), overworld/hub | `level-design` |

## Stats with modifiers

```python
class Stat:
    def __init__(self, base): self.base, self.mods = base, []   # (source, kind, value)
    def value(self):
        flat = sum(v for _, k, v in self.mods if k == "flat")
        pct = sum(v for _, k, v in self.mods if k == "pct")
        mult = 1.0
        for _, k, v in self.mods:
            if k == "mult": mult *= v
        return (self.base + flat) * (1 + pct) * mult
    def remove_source(self, src): self.mods = [m for m in self.mods if m[0] is not src]
```

## Formulas (starting points)

- XP to next level: `round(base * level ** 1.5)` (gentle) or `base * 1.1 ** level` (steep). Aim: each level takes slightly longer; early levels fast (10–15 min).
- Damage: `atk * (100 / (100 + def))` — defense never fully negates, diminishing returns. Or `max(1, atk - def/2)` for simple JRPG feel.
- Crits: chance × multiplier (1.5–2×); hit/evasion only if it creates decisions (misses frustrate).
- Enemy stats scaled by area level; player power should outpace slightly when grinding (reward effort) but not trivialize.

Prototype curves in a spreadsheet; simulate fights (time-to-kill both ways) per level band.

## Quests

```python
QUESTS = {"lost_ring": {"stages": ["talk_to_mira", "find_ring", "return_ring", "done"],
                        "rewards": {"xp": 100, "gold": 50, "items": ["amulet_luck"]}}}
state.quests["lost_ring"] = "find_ring"           # saved
# triggers: on item pickup, on dialogue node, on area enter → advance stage
```

Journal UI shows current objective; markers optional (accessibility).

## Turn-based combat structure

Initiative order (speed) → player chooses action (attack/skill/item/defend/flee) → target → resolve with animations → status effects tick → check win/lose. Show turn order UI; telegraph enemy big attacks one turn ahead.

## Scope warning

RPGs explode in content. Define a vertical slice: 1 town, 1 dungeon, 1 boss, 5 quests, 20 items, 6 enemies, 2–3 hours. Data-driven everything (items, enemies, quests, dialogue) so content is added without code.

## Testing

Balance tables in data; automated fight simulations; save/load round-trips across all quest states; check for flag combinations that block progression (softlocks).
