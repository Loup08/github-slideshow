---
name: godot-animation
description: Animate in Godot 4 with AnimatedSprite2D, AnimationPlayer, AnimationTree (state machines, blend spaces), Tweens, and call-method tracks. Use for sprite animation, character state machines, cutscenes, UI juice, and syncing gameplay events to animation frames.
---

# Godot Animation

## Pick the tool

| Need | Tool |
|---|---|
| Frame-by-frame sprite flipbook | `AnimatedSprite2D` + `SpriteFrames` |
| Animate any property, call methods, play audio, keyframed hitboxes | `AnimationPlayer` |
| Blend/transition between many animations by state & parameters | `AnimationTree` (state machine / blend space) |
| One-off procedural motion (UI pop, damage flash, camera move) | `Tween` |
| Skeletal 2D | `Skeleton2D` + `Bone2D` + AnimationPlayer |

## AnimatedSprite2D

```gdscript
@onready var sprite: AnimatedSprite2D = $AnimatedSprite2D

func _update_anim() -> void:
	if not is_on_floor():
		sprite.play("jump" if velocity.y < 0 else "fall")
	elif absf(velocity.x) > 5.0:
		sprite.play("run")
	else:
		sprite.play("idle")
	if velocity.x != 0.0:
		sprite.flip_h = velocity.x < 0.0
```

`play()` with the same name doesn't restart. Use `animation_finished` for non-looping anims (attack).

## AnimationPlayer

- Tracks: property, method call (spawn hitbox on frame 3), audio, animation (nested), bezier.
- Keyframe `CollisionShape2D:disabled` to make attack hitboxes frame-accurate.
- Use the `RESET` track to define rest pose so editor previews don't leak values into the saved scene.
- Libraries: share animations between characters via `AnimationLibrary` resources.

## AnimationTree state machine

```gdscript
@onready var tree: AnimationTree = $AnimationTree
@onready var playback: AnimationNodeStateMachinePlayback = tree.get("parameters/playback")

func _physics_process(_d: float) -> void:
	tree.set("parameters/Locomotion/blend_position", velocity.length() / max_speed)
	if Input.is_action_just_pressed("attack"):
		playback.travel("Attack")
```

- Set `anim_player` and `active = true`.
- Transitions: *Auto* advance with `advance_expression` (e.g. `is_on_floor()`) using `advance_expression_base_node`, or travel from code.
- BlendSpace2D for 8-directional top-down sets (`blend_position = input_vector`).

## Tweens

```gdscript
func hit_flash() -> void:
	var t := create_tween()
	t.tween_property(sprite, "modulate", Color(3, 3, 3), 0.05)
	t.tween_property(sprite, "modulate", Color.WHITE, 0.1)

func pop_in(node: Control) -> void:
	node.scale = Vector2.ZERO
	create_tween().tween_property(node, "scale", Vector2.ONE, 0.25)\
		.set_trans(Tween.TRANS_BACK).set_ease(Tween.EASE_OUT)
```

`create_tween()` on a node binds the tween's lifetime to that node. Kill a previous tween before starting a new one on the same property (`if _t: _t.kill()`). `set_parallel()` for simultaneous tweens, `tween_callback()` for steps, `set_loops()` for repeats.

## Pitfalls

- Animation logic in both code and AnimationTree fights — let one system own the state.
- `AnimationPlayer` property tracks hold the value after finishing; reset explicitly if needed.
- Paused tree stops tweens/animations unless their node's `process_mode` is `ALWAYS` (or `tween.set_pause_mode(Tween.TWEEN_PAUSE_PROCESS)`).
