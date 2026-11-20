---
name: card-game
description: Compositional template for card games and deckbuilders — card data model and effect system, deck/hand/discard/exhaust zones, turn structure and energy, targeting, keywords and status effects, enemy intents, shuffling with seeded RNG, card rewards and deck thinning, UI (hand fan, drag-to-play, tooltips), AI opponents, balancing, and which skills to load. Use when building a deckbuilder, collectible card game, or card-based tactics game.
---

# Card Game / Deckbuilder Template

## Core loop (roguelite deckbuilder)

Draw hand → spend energy to play cards → resolve effects → enemy acts (intent shown) → end turn/discard → win fight → choose card reward → map → … Fun = **combinatorial synergies** and **meaningful choices each turn**.

## Systems & skill map

| System | Notes | Load skill |
|---|---|---|
| Card definitions | Data assets: id, name, cost, type, rarity, target, effects list, keywords, upgrade | engine data skill |
| Effect engine | Small command list resolved via a queue/stack | — |
| Zones | Draw pile, hand, discard, exhaust, (play area) | — |
| Turn system | Start-of-turn triggers → player phase → end-of-turn → enemy phase | — |
| RNG | Seeded shuffle (Fisher–Yates); separate streams for draws/rewards | `procedural-gen` |
| Run structure | Map, rewards, shops, relics (roguelite) | `roguelike` |
| UI | Hand fan, hover zoom, drag to target with arrow, keyword tooltips, pile counts | `game-ui-ux`, `game-feel` |
| AI | Enemy intent patterns; for PvP, heuristic or MCTS | `ai-behavior-trees-utility-ai` |
| Save | Mid-run saves including pile order and RNG state | `save-systems` |

## Data model

```json
{ "id": "strike", "name": "card.strike", "cost": 1, "type": "attack", "rarity": "basic",
  "target": "enemy", "effects": [ { "op": "damage", "amount": 6 } ],
  "upgrade": { "effects": [ { "op": "damage", "amount": 9 } ] } }
{ "id": "flurry", "cost": 2, "type": "attack", "target": "all_enemies",
  "effects": [ { "op": "repeat", "times": 3, "do": [ { "op": "damage", "amount": 3 } ] },
               { "op": "apply", "status": "weak", "stacks": 1 } ], "keywords": ["exhaust"] }
```

Effects as data → hundreds of cards with a few dozen ops (damage, block, draw, discard, apply status, gain energy, add card to pile, repeat, conditional).

## Resolution queue

```python
queue = deque()
def play(card, target):
    pay(card.cost); move(card, hand, play_zone)
    for e in card.effects: queue.append((e, card, target))
    trigger("on_card_played", card)
    while queue:
        effect, src, tgt = queue.popleft()
        resolve(effect, src, tgt)                 # may enqueue more (triggers, statuses)
        if check_combat_end(): break
    move(card, play_zone, exhaust if "exhaust" in card.keywords else discard)
```

A queue (not recursion) keeps ordering predictable and lets you animate each step. Damage pipeline: base → attacker modifiers (strength, weak) → defender modifiers (vulnerable) → block → HP.

## Statuses

Stacks + duration semantics per status (decrement at end of turn, or consumed on trigger). Statuses register hooks (on_turn_start, on_damage_dealt...).

## Enemy intents

Enemies pick next action at end of the player's turn and display it (attack 12, block, buff, debuff). Patterns: fixed cycles, weighted random without repeats, conditional (below 50% HP → enrage).

## Balancing

- Value baseline: 1 energy ≈ 6 damage ≈ 5 block (pick your own and stay consistent); rarer cards exceed curve or enable synergies.
- Card rewards: choose 1 of 3, option to skip (deck thinning matters).
- Simulate fights with scripted/greedy bots over thousands of seeds to find broken cards and impossible fights.

## UI/feel

Hand arranged in an arc; hovered card scales up and pushes neighbors; drag beyond a threshold to play; invalid play snaps back with a shake; damage numbers, shake on big hits; end-turn button clearly visible; speed setting for animations.

## Scope tiers

- **Jam**: 20–30 cards, 5 enemies, 3–5 fights.
- **Small**: 1 character × 75 cards, 20 relics, 3 acts, 30 enemies, 3 bosses.
