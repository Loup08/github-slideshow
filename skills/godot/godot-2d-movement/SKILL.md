---
name: godot-2d-movement
description: Implement 2D character movement in Godot 4 with CharacterBody2D — platformer run/jump with coyote time and jump buffering, top-down 8-way movement, acceleration/friction, slopes, one-way platforms, and the InputMap. Use for any 2D player or enemy controller.
---

# Godot 2D Movement

## InputMap first

Project Settings → Input Map: define `move_left`, `move_right`, `move_up`, `move_down`, `jump`, with keyboard + gamepad bindings. Never hardcode keys.

## Platformer controller

```gdscript
extends CharacterBody2D

@export var max_speed := 180.0
@export var accel := 1400.0
@export var friction := 1600.0
@export var jump_height := 64.0        # pixels
@export var time_to_apex := 0.35       # seconds
@export var fall_gravity_mult := 1.8
@export var coyote_time := 0.1
@export var jump_buffer := 0.12

@onready var jump_velocity := -2.0 * jump_height / time_to_apex
@onready var gravity := 2.0 * jump_height / (time_to_apex * time_to_apex)

var _coyote := 0.0
var _buffer := 0.0

func _physics_process(delta: float) -> void:
	var dir := Input.get_axis("move_left", "move_right")

	# horizontal
	if dir != 0.0:
		velocity.x = move_toward(velocity.x, dir * max_speed, accel * delta)
	else:
		velocity.x = move_toward(velocity.x, 0.0, friction * delta)

	# timers
	_coyote = coyote_time if is_on_floor() else _coyote - delta
	_buffer = jump_buffer if Input.is_action_just_pressed("jump") else _buffer - delta

	# jump
	if _buffer > 0.0 and _coyote > 0.0:
		velocity.y = jump_velocity
		_buffer = 0.0
		_coyote = 0.0

	# variable jump height: cut on release
	if Input.is_action_just_released("jump") and velocity.y < 0.0:
		velocity.y *= 0.5

	# gravity (heavier when falling)
	var g := gravity * (fall_gravity_mult if velocity.y > 0.0 else 1.0)
	velocity.y = minf(velocity.y + g * delta, 900.0)

	move_and_slide()
```

Deriving gravity/jump velocity from height and time-to-apex makes tuning intuitive (see `physics-tuning`, `platformer`).

## Top-down 8-way

```gdscript
func _physics_process(delta: float) -> void:
	var input := Input.get_vector("move_left", "move_right", "move_up", "move_down")
	var target := input * max_speed           # get_vector already normalizes + applies deadzone
	var rate := accel if input != Vector2.ZERO else friction
	velocity = velocity.move_toward(target, rate * delta)
	move_and_slide()
```

Set `motion_mode = MOTION_MODE_FLOATING` for top-down so walls aren't treated as floors.

## CharacterBody2D settings

- `up_direction = Vector2.UP` (default) defines floor.
- `floor_max_angle` (default 45°) — steeper surfaces count as walls.
- `floor_snap_length` > 0 keeps the body glued on slopes/downhill.
- `floor_stop_on_slope = true` prevents sliding while idle.
- After `move_and_slide()`: `is_on_floor()`, `is_on_wall()`, `get_slide_collision(i)`, `get_last_slide_collision()`.

## One-way platforms & drop-through

Enable `one_way_collision` on the platform's CollisionShape2D. Drop through by temporarily disabling the player's collision mask bit for that layer:

```gdscript
if Input.is_action_pressed("move_down") and Input.is_action_just_pressed("jump"):
	set_collision_mask_value(PLATFORM_LAYER, false)
	await get_tree().create_timer(0.2).timeout
	set_collision_mask_value(PLATFORM_LAYER, true)
```

## Pitfalls

- Do not multiply `velocity` by `delta` before `move_and_slide()` — it already uses delta.
- Read input in `_physics_process` with `is_action_just_pressed` — fine at 60 Hz; for high-refresh precision buffer from `_unhandled_input`.
- Pixel-art jitter: enable *Rendering → 2D → Snap 2D Transforms to Pixel* and use a Camera2D with position smoothing carefully.
