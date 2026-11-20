---
name: godot-resources
description: Use custom Resources in Godot 4 as data containers — item/enemy/weapon definitions, editor-friendly data, sharing vs duplicating, saving and loading with ResourceSaver/ConfigFile/JSON, and security caveats. Use for game data design, inventories, stats, settings, and save files.
---

# Godot Resources

A `Resource` is a serializable, reference-counted data object (`.tres` text / `.res` binary). Custom resources are Godot's equivalent of Unity ScriptableObjects.

## Define data types

```gdscript
# item_data.gd
class_name ItemData
extends Resource

@export var id: StringName
@export var display_name: String
@export_multiline var description: String
@export var icon: Texture2D
@export var max_stack: int = 1
@export var value: int = 0
```

```gdscript
# weapon_data.gd
class_name WeaponData
extends ItemData
@export var damage: int = 10
@export var fire_rate: float = 0.25
@export var projectile: PackedScene
```

Create instances in FileSystem → New Resource → WeaponData; edit in the Inspector. Reference with `@export var weapon: WeaponData`.

## Shared vs unique

Resources loaded from the same path are **shared**. Mutating `weapon.damage` at runtime changes it for everyone *and* in the editor during play. For per-instance state:

```gdscript
var stats: Stats = base_stats.duplicate(true)   # deep copy (subresources too)
```

Or tick **Local to Scene** on the resource. Rule: definitions = shared & read-only; runtime state = duplicated or plain variables.

## Databases / registries

```gdscript
# autoload ItemDB
var items: Dictionary[StringName, ItemData] = {}
func _ready() -> void:
	for f in ResourceLoader.list_directory("res://data/items/"):   # 4.4+; else DirAccess.get_files_at
		if f.ends_with(".tres"):
			var it: ItemData = load("res://data/items/" + f)
			items[it.id] = it
```

In exported builds, files may be remapped (`.tres.remap`); strip `.remap` when listing, or keep an explicit `Array[ItemData]` exported on a registry resource (most robust).

## Saving game state

Option A — Resource save (fast to write, typed):

```gdscript
class_name SaveGame extends Resource
@export var player_pos: Vector2
@export var inventory: Array[StringName] = []
@export var flags: Dictionary = {}

func write() -> void: ResourceSaver.save(self, "user://save.tres")
static func read() -> SaveGame:
	if ResourceLoader.exists("user://save.tres"):
		return ResourceLoader.load("user://save.tres", "", ResourceLoader.CACHE_MODE_IGNORE)
	return SaveGame.new()
```

**Security:** `.tres` files can embed scripts; loading a save from an untrusted source can execute code. For shareable saves or mods use JSON or `FileAccess.store_var(value, false)` (objects disabled).

Option B — JSON:

```gdscript
var f := FileAccess.open("user://save.json", FileAccess.WRITE)
f.store_string(JSON.stringify(data))
var parsed = JSON.parse_string(FileAccess.get_file_as_string("user://save.json"))
```

JSON turns ints into floats and Vector2 into strings — convert explicitly (`var_to_str`/`str_to_var` handle Godot types).

Settings: `ConfigFile` (`user://settings.cfg`) with sections/keys.

`user://` maps to the OS app-data folder; `res://` is read-only in exports.

See `save-systems` for versioning/migration strategy.
