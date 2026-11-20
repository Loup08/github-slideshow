---
name: godot-physics
description: Use Godot 4 physics correctly — choosing between CharacterBody, RigidBody, StaticBody, AnimatableBody and Area, collision layers/masks, raycasts and shape queries, Jolt in 3D, and stable rigidbody manipulation. Use for collisions, triggers, projectiles, ragdolls, or physics bugs.
---

# Godot Physics

## Pick the body

| Node | Moved by | Use for |
|---|---|---|
| `StaticBody2D/3D` | nothing (or code teleport) | walls, floors |
| `AnimatableBody2D/3D` | code/AnimationPlayer, pushes others | moving platforms, doors |
| `CharacterBody2D/3D` | your code via `move_and_slide()` | players, enemies |
| `RigidBody2D/3D` | physics engine (forces) | crates, balls, debris, ragdolls |
| `Area2D/3D` | — detects overlaps only | triggers, hitboxes, pickups, gravity zones |

## Layers & masks

- **Layer** = what I am. **Mask** = what I collide with / detect.
- Name layers in Project Settings → Layer Names → 2D Physics: 1 world, 2 player, 3 enemy, 4 player_hitbox, 5 enemy_hitbox, 6 pickup.
- Code: `set_collision_layer_value(2, true)`, `set_collision_mask_value(1, true)`.
- Hurtbox/hitbox pattern: player Hurtbox `layer=player_hurt`, enemy Hitbox `mask=player_hurt`.

## Areas

```gdscript
func _ready() -> void:
	body_entered.connect(_on_body_entered)     # PhysicsBody entered
	area_entered.connect(_on_area_entered)     # another Area entered

func _on_body_entered(body: Node2D) -> void:
	if body.has_method("take_damage"):
		body.take_damage(10)
```

`monitoring` = this area detects others; `monitorable` = others can detect this area. Polling: `get_overlapping_bodies()` (updated once per physics frame).

## Raycasts

Node-based (persistent): `RayCast2D` with `target_position`, `force_raycast_update()`, `is_colliding()`, `get_collider()`, `get_collision_point()`, `get_collision_normal()`.

One-off query:

```gdscript
var space := get_world_2d().direct_space_state
var q := PhysicsRayQueryParameters2D.create(from, to, collision_mask, [self.get_rid()])
var hit := space.intersect_ray(q)
if hit:
	print(hit.collider, hit.position, hit.normal)
```

Shapes: `PhysicsShapeQueryParameters2D` + `intersect_shape()` for explosions/melee arcs. Only query the space state inside `_physics_process`.

## RigidBody rules

- Never set `position` directly each frame. Use `apply_central_impulse()`, `apply_force()`, `apply_torque_impulse()`, or set `linear_velocity`.
- To teleport or fully control: override `_integrate_forces(state)` and edit `state.transform` / `state.linear_velocity`.
- `freeze = true` + `freeze_mode` to make it kinematic temporarily.
- `contact_monitor = true` and `max_contacts_reported > 0` are required for `body_entered` on RigidBody.
- `continuous_cd` for fast small objects (bullets) to avoid tunneling.

## 3D: Jolt

Jolt is built in (Godot 4.4+) and is the recommended 3D engine: Project Settings → Physics → 3D → Physics Engine = *Jolt Physics*. More stable stacking and better performance than GodotPhysics3D. Re-test joint/ragdoll tuning after switching.

## Tuning

- Physics tick: Project Settings → Physics → Common → Physics Ticks per Second (60 default). Enable *Physics Interpolation* (4.3+ for 2D, 4.4+ 3D) to decouple rendering smoothness from tick rate.
- Scale: keep 3D objects near real-world meters; tiny/huge masses cause jitter.

## Pitfalls

- Changing collision shapes/disabled state inside a collision callback → use `set_deferred("disabled", true)`.
- Scaling collision shapes non-uniformly on physics bodies is unsupported — resize the shape resource instead.
- Two overlapping static bodies cause characters to snag; merge shapes.
