---
name: platformer
description: Compositional template for 2D/3D platformers — core loop, essential systems (character controller with coyote time and jump buffer, hazards, checkpoints, collectibles, moving platforms, enemies), level structure and teaching, camera, juice, scope tiers, and which engine/discipline skills to load for each part. Use when building or designing a platformer or precision platformer.
---

# Platformer Template

## Core loop

Move → jump over obstacles/enemies → collect → reach goal → next level. Fun comes from **control precision** and **level design** — invest there first.

## Systems & skill map

| System | Notes | Load skill |
|---|---|---|
| Character controller | Accel/decel, variable jump, coyote 0.1 s, buffer 0.12 s, fall gravity ×1.5–2, terminal velocity | `physics-tuning`, engine movement skill (`godot-2d-movement`, `phaser-arcade-physics`, `unity-physics`) |
| Input | Action mapping, gamepad, rebinding | `input-systems` |
| Tilemap levels | Collision layer, one-way platforms, hazards layer | `godot-tilemap`, `unity-tilemap-2d`, `level-design` |
| Hazards | Spikes (instant death or damage + knockback), pits, saws; forgiving hitboxes (smaller than the sprite) | — |
| Checkpoints & respawn | Frequent; respawn in < 1 s for precision platformers | `save-systems` |
| Collectibles | Coins (path guidance), hidden collectibles (exploration) | `level-design` |
| Moving platforms | Kinematic bodies, carry the player (add platform velocity) | `physics-tuning` |
| Enemies | Simple patterns (walk & turn at edges, jump, shoot); stomp to kill | `game-ai` |
| Camera | Dead zone, look-ahead, platform-snapped vertical | `camera-systems` |
| Feel | Squash/stretch, dust, landing shake (tiny), sounds | `game-feel`, `audio-design` |
| UI | Lives/health, collectible count, timer (speedrun) | `game-ui-ux` |

## Optional mechanics (pick 1–2 core + a few supporting)

Wall slide/jump, dash (with refresh on landing), double jump, ledge grab, glide, grapple, ground pound, swimming, gravity flip. Each new verb needs levels that teach and remix it.

## Precision-platformer forgiveness

- Corner correction: if the head clips a ceiling corner by ≤ 4 px, nudge sideways.
- Ledge assist: if the feet barely miss a ledge, pop up onto it.
- Hitboxes: player hurtbox smaller than sprite; hazards hitboxes smaller than visuals.
- Instant restart button; death → respawn without menus.

## Level structure

Short rooms/levels (30–90 s) for precision games; longer levels with checkpoints for adventure platformers. Use the introduce → practice → twist → master pattern for each mechanic (see `level-design`). Worlds with a theme and a signature mechanic.

## Scope tiers

- **Jam (48 h)**: one mechanic twist, 5–10 short levels, one enemy type.
- **Small (1–3 months)**: 3 worlds × 8 levels, 3–5 enemies, 1 boss per world, collectibles.
- **Metroidvania**: see `level-design` (ability-gated interconnected map) + `save-systems` + `rpg` (upgrades).

## Testing checklist

- [ ] Jump height/distance match level metrics; documented
- [ ] Controls feel good at 30, 60, 144 Hz (frame-rate independence)
- [ ] No softlocks (can't get stuck in geometry; respawn always valid)
- [ ] Every level beaten by someone who isn't the designer
