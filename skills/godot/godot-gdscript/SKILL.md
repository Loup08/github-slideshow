---
name: godot-gdscript
description: Write idiomatic, statically typed GDScript for Godot 4.x — lifecycle callbacks, @export/@onready, typed variables and arrays, enums, inner classes, autoloads, await/coroutines, and common pitfalls. Use when writing or reviewing any .gd file.
---

# GDScript (Godot 4)

## File skeleton

```gdscript
class_name Player            # global type name (optional; avoid for one-off scripts)
extends CharacterBody2D

signal health_changed(new_value: int)

enum State { IDLE, RUN, JUMP }

const MAX_HEALTH := 100

@export var speed: float = 200.0
@export_range(0, 100, 1) var armor: int = 0
@export var bullet_scene: PackedScene

@onready var sprite: AnimatedSprite2D = $AnimatedSprite2D
@onready var hurtbox: Area2D = %Hurtbox      # % = unique name in scene

var state: State = State.IDLE
var health: int = MAX_HEALTH:
	set(value):
		health = clampi(value, 0, MAX_HEALTH)
		health_changed.emit(health)

func _ready() -> void:
	hurtbox.area_entered.connect(_on_hurtbox_area_entered)

func _physics_process(delta: float) -> void:
	pass

func _on_hurtbox_area_entered(area: Area2D) -> void:
	health -= 10
```

## Lifecycle order

`_init()` (object created, no tree) → `_enter_tree()` → children `_ready()` → own `_ready()` (once) → `_process(delta)` every frame / `_physics_process(delta)` every physics tick (default 60 Hz) → `_exit_tree()`. Use `_physics_process` for movement/physics, `_process` for visuals/UI. `_unhandled_input(event)` for gameplay input so UI can consume events first.

## Typing rules

- Always type: `var x: int`, `func f(a: Vector2) -> bool:`. Use `:=` when the type is obvious from the right side.
- Typed arrays/dicts: `var enemies: Array[Enemy] = []`, `var stats: Dictionary[String, int] = {}` (typed dictionaries exist since 4.4).
- Enable *Project Settings → Debug → GDScript → Untyped Declaration = Warning/Error* to enforce typing.
- Cast with `as` (returns null on failure): `var e := body as Enemy`; check with `is`.

## Coroutines

```gdscript
await get_tree().create_timer(0.5).timeout
await anim_player.animation_finished
var tween := create_tween()
tween.tween_property(self, "modulate:a", 0.0, 0.3)
await tween.finished
```

A function containing `await` is a coroutine; callers must `await` it to wait for the result.

## Autoloads (singletons)

Project Settings → Globals → Autoload. Use for game state, event bus, audio manager. Keep them thin; don't put scene logic there.

## Pitfalls

- `$Node` paths break when the scene is restructured — prefer `%UniqueName` or `@export var target: Node2D`.
- `@onready` vars are null in `_init()`; touch nodes only from `_ready()` onward.
- Freeing: `queue_free()` (safe, end of frame), never `free()` during physics callbacks. Check `is_instance_valid(obj)` for references that may have been freed.
- Integer division: `5 / 2 == 2`. Use `5 / 2.0`.
- `Callable` binding: `button.pressed.connect(_on_pressed.bind(index))`.
- Godot 3 → 4 renames: `KinematicBody2D`→`CharacterBody2D`, `yield`→`await`, `export`→`@export`, `onready`→`@onready`, `connect("sig", self, "m")`→`sig.connect(m)`, `instance()`→`instantiate()`.

## Style

snake_case functions/vars, PascalCase classes/nodes, CONSTANT_CASE constants, `_private` prefix, signals in past tense (`died`, `item_collected`). Tabs for indentation.
