---
name: godot-3d-essentials
description: Core 3D in Godot 4 — CharacterBody3D third/first-person controllers, Camera3D and SpringArm3D, importing glTF, lighting with WorldEnvironment/SDFGI/LightmapGI, materials, navigation meshes, and renderer choice (Forward+, Mobile, Compatibility). Use for any 3D Godot scene or controller.
---

# Godot 3D Essentials

## Renderer choice

- **Forward+**: desktop, full features (SDFGI, volumetric fog, SSR).
- **Mobile**: mobile/low-end Vulkan/Metal, fewer lights per object.
- **Compatibility** (OpenGL/WebGL2): web exports and old hardware; no SDFGI/volumetrics.

Decide early — lighting looks differ.

## Third-person controller

```gdscript
extends CharacterBody3D

@export var speed := 5.0
@export var jump_velocity := 4.5
@export var mouse_sens := 0.003
@onready var pivot: Node3D = $CameraPivot        # pivot → SpringArm3D → Camera3D
@onready var model: Node3D = $Model
var gravity: float = ProjectSettings.get_setting("physics/3d/default_gravity")

func _ready() -> void:
	Input.mouse_mode = Input.MOUSE_MODE_CAPTURED

func _unhandled_input(event: InputEvent) -> void:
	if event is InputEventMouseMotion:
		pivot.rotation.y -= event.relative.x * mouse_sens
		pivot.rotation.x = clampf(pivot.rotation.x - event.relative.y * mouse_sens, -1.2, 0.6)

func _physics_process(delta: float) -> void:
	if not is_on_floor():
		velocity.y -= gravity * delta
	elif Input.is_action_just_pressed("jump"):
		velocity.y = jump_velocity
	var input := Input.get_vector("move_left", "move_right", "move_forward", "move_back")
	var dir := (pivot.global_basis * Vector3(input.x, 0, input.y))
	dir.y = 0
	dir = dir.normalized()
	if dir:
		velocity.x = dir.x * speed
		velocity.z = dir.z * speed
		model.rotation.y = lerp_angle(model.rotation.y, atan2(-dir.x, -dir.z), 10 * delta)
	else:
		velocity.x = move_toward(velocity.x, 0, speed)
		velocity.z = move_toward(velocity.z, 0, speed)
	move_and_slide()
```

`SpringArm3D` keeps the camera out of walls (set its `collision_mask` to world only; exclude the player). First-person: rotate the body on Y, a head node on X, camera as child of head.

Godot's forward axis is **-Z**.

## Importing models

- Prefer **glTF 2.0** (`.glb`). Blender files can be imported directly if Blender path is configured.
- Import dock: generate collision with name suffixes in Blender (`-col`, `-convcolonly`, `-navmesh`) or via *Advanced Import Settings*.
- Instance the imported scene; to edit, create an **inherited scene** (don't edit the import).
- Real-world scale: 1 unit = 1 meter.

## Lighting checklist

1. `WorldEnvironment` with `Environment`: Sky (ProceduralSkyMaterial or HDRI PanoramaSkyMaterial), tonemap **AgX/Filmic/ACES**, glow, SSAO.
2. One `DirectionalLight3D` with shadows; tune `directional_shadow_max_distance`.
3. GI: **SDFGI** (dynamic, Forward+ open worlds), **VoxelGI** (medium interiors), **LightmapGI** (static, best quality/perf, works on Mobile/Compat).
4. `ReflectionProbe` for interiors.

## Materials

`StandardMaterial3D` (PBR): albedo, metallic, roughness, normal map (enable), emission. `ORMMaterial3D` when textures are packed occlusion/roughness/metallic. Use `material_override` sparingly — prefer surface materials on the mesh.

## Navigation

`NavigationRegion3D` → bake a `NavigationMesh` from child geometry (agent radius/height must match the character). Agents use `NavigationAgent3D`:

```gdscript
agent.target_position = player.global_position
var next := agent.get_next_path_position()
velocity = global_position.direction_to(next) * speed
```

Enable `avoidance_enabled` + `velocity_computed` signal for crowd avoidance.

## Performance

Visibility ranges (LOD/HLOD) on GeometryInstance3D, automatic mesh LOD on import, `MultiMeshInstance3D` for foliage, occlusion culling (`OccluderInstance3D` + bake).
