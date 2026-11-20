---
name: game-feel
description: Add juice and game feel in any engine — responsive controls, anticipation and follow-through, hit stop/freeze frames, screen shake with trauma, knockback, squash and stretch, flashes, particles, impact sounds, easing and tweens, camera kick, number popups, and how to tune it without overdoing it. Use when a game feels flat, unresponsive, or unsatisfying.
---

# Game Feel (Juice)

Feel = **responsiveness** (input → visible reaction instantly) + **feedback** (every action has multi-sensory confirmation) + **polish** (easing, secondary motion). Fix responsiveness first; juice can't save laggy controls.

## Responsiveness checklist

- Action starts on the frame of input (or within 1–2 frames).
- Acceleration/turnaround fast (see `physics-tuning`).
- Input buffering + coyote time (see `input-systems`).
- Animation cancel windows; no long wind-ups for basic moves (unless the design is deliberate/weighty).

## Feedback toolbox

| Technique | Recipe |
|---|---|
| **Hit stop** | Freeze time (or attacker+victim) for 30–100 ms on impactful hits; scale with damage |
| **Screen shake** | Trauma-based (below); short, directional for recoil |
| **Flash** | Victim white flash 1–3 frames (shader `flash_amount`) |
| **Knockback** | Impulse away from the hit direction; brief loss of control |
| **Squash & stretch** | Scale (1.2, 0.8) on jump take-off, (0.8, 1.2)→(1.2, 0.8) on landing, tween back to 1 with elastic/back easing |
| **Particles** | Dust on land/turn, sparks on hit, debris on break, trails on dash |
| **Sound** | Layered impact per hit with pitch variation; louder/bassier for crits |
| **Camera kick** | Small offset/zoom on fire or big hits |
| **Number popups** | Damage numbers that pop (scale up then fade + rise) |
| **Time dilation** | Slow-mo on final kill / perfect parry (0.2–0.5 s) |
| **Rumble** | Gamepad vibration matched to impact strength |
| **Persistence** | Decals, shells, corpses that remain for a while |

## Trauma screen shake

```python
trauma = 0.0     # 0..1, add on events: trauma = min(1, trauma + 0.3)

def update_shake(dt, t):
    global trauma
    trauma = max(0.0, trauma - dt * 1.5)          # decay
    shake = trauma ** 2                            # quadratic → small hits subtle, big hits strong
    offset_x = MAX_OFFSET * shake * noise(seed, t * FREQ)       # smooth noise, not random() each frame
    offset_y = MAX_OFFSET * shake * noise(seed + 1, t * FREQ)
    angle    = MAX_ANGLE  * shake * noise(seed + 2, t * FREQ)
    camera.set_offset(offset_x, offset_y, angle)
```

Apply shake as an offset on top of the follow position (never move the actual camera target). Provide a **screen shake intensity** setting (accessibility; motion sickness).

## Hit stop implementation

Global: set time scale to 0 (or 0.05) for N real-time ms, then restore — use unscaled real time for the timer. Local: pause only the two combatants' animations/physics for richer feel without stopping particles/UI.

## Easing

Never move UI or objects linearly for polish: ease-out for things arriving (fast start, slow end), ease-in for leaving, back/elastic for playful pops. Duration 0.1–0.3 s for UI.

## Anticipation & follow-through

Enemy attacks: wind-up pose + flash/sound (anticipation) → fast strike → recovery (follow-through). Player attacks: minimal anticipation (responsive) but strong follow-through (smear frames, trailing particles).

## Tuning discipline

- Expose every juice parameter; tune in-game with a debug panel.
- Juice proportional to importance: killing a boss ≫ hitting a crate. If everything shakes, nothing does.
- Watch readability: effects must not hide enemies/projectiles.
- Record gameplay at full speed and review; compare with and without (toggle all juice off with one key).

## Fast wins (30 minutes)

1. Hit flash + 50 ms hit stop on damage.
2. Small shake on player getting hit and on explosions.
3. Squash/stretch on jump and land + dust particles.
4. Pitch-randomized impact sounds.
5. Eased UI transitions and a bouncy score popup.
