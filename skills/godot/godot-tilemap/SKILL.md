---
name: godot-tilemap
description: Build 2D levels in Godot 4 with TileMapLayer and TileSet — atlas setup, physics/navigation/custom data layers, terrains (autotiling), painting tiles from code, coordinate conversion, and migrating from the deprecated TileMap node. Use for any grid-based 2D level work.
---

# Godot TileMapLayer

Since Godot 4.3 each layer is its own **`TileMapLayer`** node; the old multi-layer `TileMap` node is deprecated. Use one node per layer (Ground, Walls, Decor) sharing one `TileSet` resource.

## TileSet setup

1. Create a `TileSet` resource (save it as `.tres` to share between layers). Set `tile_size` (e.g. 16×16).
2. TileSet panel → add an **Atlas** source from your spritesheet; Godot auto-creates tiles.
3. Add layers on the TileSet (Inspector):
   - **Physics Layers** → paint collision polygons per tile in the TileSet editor (*Paint → Physics Layer 0*).
   - **Navigation Layers** → navigation polygons for `NavigationAgent2D`.
   - **Custom Data Layers** → e.g. `damage: int`, `is_ladder: bool`, `footstep: String`.
   - **Terrain Sets** → autotiling (mode *Match Corners and Sides* for 47-tile blob sets, *Match Sides* for simple 16-tile sets).
4. Pixel art: Project Settings → Rendering → Textures → Default Texture Filter = *Nearest*.

## Painting from code

```gdscript
@onready var ground: TileMapLayer = $Ground

const SOURCE_ID := 0
const GRASS := Vector2i(1, 0)   # atlas coords

func fill_rect(rect: Rect2i) -> void:
	for x in range(rect.position.x, rect.end.x):
		for y in range(rect.position.y, rect.end.y):
			ground.set_cell(Vector2i(x, y), SOURCE_ID, GRASS)

func paint_terrain(cells: Array[Vector2i]) -> void:
	ground.set_cells_terrain_connect(cells, 0, 0)  # terrain_set, terrain → autotiles neighbours

func clear(cell: Vector2i) -> void:
	ground.erase_cell(cell)
```

## Querying

```gdscript
var cell := ground.local_to_map(ground.to_local(global_pos))   # world → cell
var world := ground.to_global(ground.map_to_local(cell))       # cell centre → world
var data := ground.get_cell_tile_data(cell)
if data and data.get_custom_data("is_ladder"): ...
var used := ground.get_used_cells()
var rect := ground.get_used_rect()       # handy for camera limits
```

## Scene tiles

An atlas source can't hold scripts. For interactive tiles (chests, spawners) add a **Scenes Collection** source, or place real scenes in a separate node and keep tiles purely visual/collision.

## Y-sort (top-down)

Enable `y_sort_enabled` on the parent and on the TileMapLayer, and set per-tile `y_sort_origin` so the player walks behind tall tiles.

## Performance

- Collision is merged per quadrant; huge maps are fine. Toggle `collision_enabled` off on decor layers.
- `rendering_quadrant_size` controls batching chunk size (default 16).
- For runtime procedural maps, batch `set_cell` calls then let the layer update once per frame (it's deferred automatically).

## Migrating from TileMap

Select the old `TileMap` node → TileMap bottom panel → toolbar menu → *Extract TileMap layers as individual TileMapLayer nodes*. Replace `tilemap.set_cell(layer, ...)` with `layer_node.set_cell(...)`.

## Pitfalls

- Tile seams/bleeding: add 1px padding/extrusion in the atlas or enable *Use Texture Padding* on the atlas source.
- Character snagging on tile edges: use rectangle collision with slightly rounded/capsule player shapes, or merge collision on a separate simplified layer.
