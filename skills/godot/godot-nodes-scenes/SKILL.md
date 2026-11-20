---
name: godot-nodes-scenes
description: Structure Godot 4 projects with nodes and scenes — composition, instancing PackedScenes at runtime, scene tree navigation, ownership, changing scenes, and folder layout. Use when designing scene hierarchy, spawning objects, or switching levels.
---

# Godot Nodes & Scenes

## Mental model

A **scene** is a saved tree of nodes (`.tscn`). Scenes are reusable prefabs: a Player scene, a Bullet scene, a Level scene. Compose behavior by **child nodes**, not deep inheritance.

```
Player (CharacterBody2D)  ← root type defines what the scene "is"
├── Sprite (AnimatedSprite2D)
├── Collision (CollisionShape2D)
├── Hurtbox (Area2D)
│   └── CollisionShape2D
├── HealthComponent (Node)     ← reusable component scene
└── Camera2D
```

## Instancing at runtime

```gdscript
const BULLET := preload("res://entities/bullet/bullet.tscn")   # compile-time load

func shoot() -> void:
	var b: Bullet = BULLET.instantiate()
	b.global_position = muzzle.global_position
	b.direction = Vector2.RIGHT.rotated(global_rotation)
	get_tree().current_scene.add_child(b)   # don't parent bullets to the shooter
```

Set properties **before** `add_child` when `_ready` depends on them. Use `load()` / `ResourceLoader.load_threaded_request()` for large or dynamic paths.

## Finding nodes

| Need | Use |
|---|---|
| Fixed child | `$Sprite` or `%Sprite` (unique name) |
| Designer-assigned reference | `@export var target: Node2D` |
| All of a kind | groups: `get_tree().get_nodes_in_group("enemies")` |
| Parent talks to child | call methods directly |
| Child talks to parent / siblings | **signals** ("call down, signal up") |

## Changing scenes

```gdscript
get_tree().change_scene_to_file("res://levels/level_2.tscn")
get_tree().change_scene_to_packed(LEVEL_2)
get_tree().reload_current_scene()
```

For persistent HUD/music across levels, use a root `Main` scene that swaps a `LevelContainer` child, or autoloads.

## Ownership & saving scenes from code

Nodes added from code are only saved into a `PackedScene` if `node.owner = root`. Matters for editor tools (`@tool`) and runtime level editors.

## Folder layout (feature-based)

```
res://
  entities/player/{player.tscn, player.gd, player.png}
  entities/enemies/slime/...
  levels/
  ui/
  autoload/
  resources/
```

## Pitfalls

- Adding children during `_ready` of a parent that's still entering: use `add_child.call_deferred(node)`.
- Removing physics bodies inside collision callbacks: `queue_free()` or `call_deferred`.
- Editable Children / "Make Local" breaks the link to the source scene — prefer inherited scenes or exported properties.
- Too many nodes (10k+) costs; for thousands of bullets/particles consider `MultiMeshInstance2D`, `GPUParticles2D`, or servers (`PhysicsServer2D`, `RenderingServer`).
