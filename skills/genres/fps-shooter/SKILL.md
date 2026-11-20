---
name: fps-shooter
description: Compositional template for first-person shooters — FPS controller (movement, crouch, sprint, slide), mouse/gamepad aiming with aim assist, weapons (hitscan vs projectile, recoil, spread, ammo, reload), damage and hitboxes, enemy AI and encounters, viewmodel rendering, feedback (hit markers, muzzle flash, impacts), level design for combat, multiplayer considerations, and which engine/discipline skills to load. Use when building an FPS or first-person action game.
---

# FPS Template

## Core loop

Move through space → encounter enemies → aim/shoot/use cover/abilities → loot ammo/health → progress. Fun = **gun feel** + **movement** + **encounter design**.

## Systems & skill map

| System | Notes | Load skill |
|---|---|---|
| Controller | CharacterBody/CharacterController capsule, accel ground + air control, crouch (capsule resize + check headroom), sprint, jump, optional slide/mantle | engine physics skill, `physics-tuning` |
| Look | Raw mouse delta × sensitivity (no dt), pitch clamp ±89°, FOV setting | `input-systems`, `camera-systems` |
| Weapons | Data-driven: damage, fire rate, mag, reload, spread, recoil pattern, range falloff | engine data skill |
| Hit detection | Hitscan raycast from camera; projectiles for slow rockets/grenades | engine physics skill |
| Damage | Hitboxes per body part (head ×2), armor, death | `game-ai` |
| AI | Cover, flanking, suppression, attack tokens | `game-ai`, `unreal-behavior-trees`, `ai-behavior-trees-utility-ai` |
| Feedback | Muzzle flash, tracer, impact decals/particles by surface, hit marker + sound, kill confirm, damage direction indicator | `game-feel`, `audio-design` |
| HUD | Crosshair (dynamic spread), ammo, health, objective | `game-ui-ux` |

## Weapon firing (hitscan)

```python
def fire(weapon, camera):
    if now < weapon.next_fire or weapon.ammo == 0: return
    weapon.next_fire = now + 60 / weapon.rpm
    weapon.ammo -= 1
    spread = weapon.base_spread + weapon.bloom + (weapon.move_spread if moving else 0)
    for _ in range(weapon.pellets):
        dir = random_cone(camera.forward, spread)
        hit = raycast(camera.position, dir, weapon.range, mask=WORLD | HITBOXES)
        if hit:
            dmg = weapon.damage * falloff(hit.distance) * hit.collider.multiplier
            hit.entity.damage(dmg, hit.point, dir)
            spawn_impact(hit.point, hit.normal, hit.surface)
    weapon.bloom = min(weapon.bloom + weapon.bloom_per_shot, weapon.max_bloom)
    camera.add_recoil(weapon.recoil_pattern[weapon.shot_index % len(weapon.recoil_pattern)])
    play_muzzle_flash(); play_sound(weapon.fire_sfx, pitch_jitter=0.05)
```

- Raycast from the **camera**, then draw the tracer from the muzzle to the hit point (avoids muzzle-parallax misses).
- Recoil: kick the view (pitch up + random yaw), recover over time; patterns learnable.
- Reload: interruptible by sprint/switch? decide; tactical vs empty reload times.

## Viewmodel

Render arms/weapon with a separate camera/layer and lower FOV (60–70) so it doesn't clip walls or distort at high world FOV. Add sway (lag behind mouse), bob (with movement), and procedural kick on fire.

## Encounters & levels

Arenas with cover, verticality, multiple routes; enemy mixes (rushers force movement, snipers force cover). Ammo/health placement shapes pacing. Sightlines: long + short ranges for different weapons. See `level-design`.

## Gamepad

Aim assist: slowdown (reduced sensitivity) when the reticle is over/near a target + mild magnetism on ADS; response curve on right stick; separate horizontal/vertical sensitivity.

## Multiplayer notes

Server-authoritative hit validation with **lag compensation** (rewind hitboxes to the shooter's view time), client-side prediction for movement, interpolation for remote players. This is a big scope multiplier — prototype single-player first.

## Scope tiers

- **Jam**: 1–2 weapons, 2 enemy types, 1 arena or short level, wave survival.
- **Small**: 4–6 weapons, 5 enemy types, 5–8 levels, boss.

## Testing

Measure time-to-kill per weapon/enemy; record playtests for frustration (deaths with no visible attacker). Check FPS stays high — input latency matters most in this genre.
