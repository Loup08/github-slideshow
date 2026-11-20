---
name: physics-tuning
description: Tune game physics and movement feel — deriving gravity and jump velocity from jump height and time, acceleration/deceleration curves, air control, fall-speed caps, fixed timestep and interpolation, collision tunneling and CCD, stable stacking, mass ratios, friction and bounce, character controller vs rigidbody trade-offs, and debugging jitter. Use when movement feels floaty/stiff or physics is unstable in any engine.
---

# Physics Tuning

## Jump from designer-friendly parameters

Pick **jump height** `h` (units) and **time to apex** `t`:

```
gravity        g  = 2h / t²
jump velocity  v0 = 2h / t        (= g·t)
```

Example: h = 3 tiles × 16 px = 48 px, t = 0.35 s → g ≈ 784 px/s², v0 ≈ 274 px/s.

Then shape the arc:

- **Fall gravity multiplier** 1.5–2.5× for snappy descent (asymmetric arc feels less floaty).
- **Variable jump**: on button release while rising, multiply vy by 0.4–0.6 (or switch to higher gravity).
- **Apex hang**: reduce gravity ×0.5 when |vy| is small near the apex for precision.
- **Terminal velocity** cap (e.g. 1.5–2× v0).
- Coyote time 0.08–0.12 s, jump buffer 0.1–0.15 s.

Horizontal jump distance ≈ `max_speed × total_air_time` — design gaps from this number.

## Ground movement

- Time to max speed (accel) 0.05–0.2 s for responsive characters; deceleration faster than acceleration.
- Turn-around boost: higher accel when input opposes velocity.
- Air control: 50–80% of ground acceleration; keep momentum feel.
- Use `move_toward(v, target, accel*dt)` (linear) for precise control; exponential smoothing `v = lerp(v, target, 1 - exp(-k*dt))` for floaty/vehicle feel (frame-rate independent form!).

## Timestep

- Run physics at a **fixed step** (50–120 Hz). Render with **interpolation** between the last two physics states to avoid stutter on high-refresh displays.
- Classic loop: accumulate frame time, step while `acc >= dt`, cap steps per frame (spiral of death protection), interpolate by `acc/dt`.
- Never scale forces by variable frame dt inside a fixed step; and never apply `delta` twice (e.g. velocity already integrated by the engine).

## Tunneling

Fast objects pass through thin colliders when `speed × dt > thickness`:

- Enable continuous collision detection (CCD) on fast bodies.
- Raycast/shape-cast from previous to current position for bullets (hitscan or swept).
- Thicken colliders, cap max speed, or substep.

## Stability (rigidbodies)

- Keep mass ratios < ~10:1 between interacting bodies (heavy on light causes jitter/explosions).
- Real-world scale for 3D (1 unit = 1 m); tiny objects (< 5 cm) and huge ones destabilize solvers.
- Increase solver iterations for stacks/joints; let bodies sleep.
- Avoid overlapping spawn positions — bodies spawned inside each other explode apart.
- Moving platforms: kinematic bodies moved via velocity/MovePosition so contacts get proper friction.

## Friction & bounce

- Characters: zero friction on the collider (avoid sticking to walls), handle deceleration in code.
- Bounce (restitution) 0 for most things; 0.3–0.8 for balls/pickups; combine modes matter (min/max/avg).

## Character controller vs rigidbody

| Kinematic/character controller | Dynamic rigidbody |
|---|---|
| Precise, predictable, tight platformers/shooters | Emergent physics, pushes, ragdoll-like |
| You handle slopes, steps, pushing | Engine handles collisions, harder to make crisp |
| Recommended for most player characters | Vehicles, physics puzzles, rolling balls |

## Slopes & steps

Snap to ground when walking down slopes (floor snap length), max walkable angle 45–50°, step offset for stairs (or invisible ramps on stairs). Project velocity onto the ground plane for consistent speed on slopes.

## Debugging jitter checklist

1. Camera following in the wrong update (follow in late update/after physics, or interpolate).
2. Physics moved in render loop / transform set directly on dynamic bodies.
3. Missing interpolation with physics rate ≠ display rate.
4. Sub-pixel positions in pixel art without snapping.
5. Two systems writing the same transform (animation + physics).

Make every number an exported/tweakable variable and tune live with sliders; record "golden" values once it feels right.
