---
name: camera-systems
description: Design and implement game cameras in any engine — 2D follow with dead zones and look-ahead, platformer vertical locking, room/screen transitions, bounds clamping, frame-rate independent smoothing, pixel-perfect cameras, third-person orbit with collision, first-person, top-down/isometric, multi-target framing for co-op, cinematic rails, and screen shake integration. Use for any camera behavior or camera jitter problem.
---

# Camera Systems

## Golden rules

1. Update the camera **after** the target moved (late update / after physics), or interpolate.
2. Smooth with **frame-rate independent** damping: `pos = lerp(pos, target, 1 - exp(-speed * dt))`.
3. Separate the logical camera position from effects (shake, kick) applied as offsets.
4. Clamp to level bounds after smoothing.

## 2D follow with dead zone + look-ahead

```python
class Camera2D:
    def __init__(self):
        self.pos = Vec2(); self.lookahead = Vec2()
    def update(self, target, dt):
        # dead zone: only move when the target leaves a box around the center
        dz_w, dz_h = 40, 30
        dx = target.pos.x - self.pos.x
        dy = target.pos.y - self.pos.y
        goal = Vec2(self.pos.x, self.pos.y)
        if abs(dx) > dz_w: goal.x = target.pos.x - copysign(dz_w, dx)
        if abs(dy) > dz_h: goal.y = target.pos.y - copysign(dz_h, dy)
        # look-ahead in the facing/moving direction
        desired_la = Vec2(target.facing * 48, 0)
        self.lookahead = self.lookahead.lerp(desired_la, 1 - exp(-3 * dt))
        goal += self.lookahead
        self.pos = self.pos.lerp(goal, 1 - exp(-8 * dt))
        self.pos = clamp_to_bounds(self.pos, level_bounds, view_size)
```

## Platformer specifics

- **Vertical**: don't follow every jump. Update the vertical target only when the player lands on a new platform (platform snapping), or when falling below a threshold.
- Look-ahead horizontal shifts when the player changes direction; slow transition to avoid nausea.
- Look-down/up input after holding for 0.5 s.
- Camera zones/triggers that lock axes or set bounds for specific rooms.

## Room-based (Zelda-like / metroidvania)

Each room defines bounds; camera clamps within the current room; when the player crosses into another room, tween/pan the camera (pause gameplay briefly) or cut.

## Pixel-perfect

Render at low resolution to a texture and scale by integers; round the camera position to whole pixels **at render time** (keep a float logical position). Engine helpers: Godot snap 2D transforms/vertices to pixel, Unity Pixel Perfect Camera (URP), Phaser `roundPixels`.

## Third-person orbit

- Yaw/pitch from input (clamp pitch −70°…+60°); pivot at the character's shoulder height.
- Collision: spherecast from pivot to desired camera position; place the camera at hit distance − radius; restore distance smoothly (fast in, slow out).
- Over-the-shoulder offset for shooters; swap shoulders.
- Auto-recenter behind the player after no camera input for a few seconds (optional).
- Engine: Godot SpringArm3D, Unity Cinemachine (Third Person Follow / Orbital Follow + Deoccluder), Unreal SpringArmComponent.

## First-person

Camera at eye height as a child of the body; yaw rotates the body, pitch the head. Add subtle head bob (toggleable), FOV 80–100° (setting), weapon rendered with a separate FOV or layer to avoid clipping.

## Top-down / isometric

Fixed angle; follow with slight look-ahead toward aim/mouse (`lerp(player, mouse_world, 0.2–0.3)`); for isometric, rotate in 90° steps if allowed and fade occluding walls.

## Multi-target (co-op, fighting)

Center = average (or weighted) of targets; zoom so the bounding box + margin fits the view, clamped between min/max zoom; smooth both. If players get too far apart: leash players or split-screen.

## Cinematics

Rails/dollies (spline + look-at), blend between virtual cameras (Cinemachine, Unreal Sequencer, Godot AnimationPlayer on Camera). Keep player input disabled or let them skip.

## Comfort & accessibility

Options for shake intensity, head bob, FOV, motion blur off, invert axes, sensitivity. Avoid fast rotations not caused by the player.

## Jitter diagnosis

Target moves in physics step but camera in render step without interpolation → enable physics interpolation or follow the interpolated visual position. Also check for smoothing applied twice and sub-pixel rendering in pixel art.
