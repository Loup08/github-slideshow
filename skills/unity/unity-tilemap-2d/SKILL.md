---
name: unity-tilemap-2d
description: Build 2D levels in Unity with Grid, Tilemap, Tile Palette, Rule Tiles and Animated Tiles (2D Tilemap Extras), TilemapCollider2D + CompositeCollider2D, sorting, and setting tiles from code for procedural maps. Use for any grid-based 2D Unity level.
---

# Unity 2D Tilemaps

## Setup

1. GameObject → 2D Object → Tilemap → Rectangular (also Isometric, Hexagonal). Creates `Grid` + `Tilemap`.
2. Separate Tilemaps per layer: `Ground`, `Walls` (with collision), `Decor`, `Foreground`.
3. Window → 2D → Tile Palette → create palette, drag sliced sprites to make Tile assets.
4. Sprites: Pixels Per Unit = tile size (e.g. 16), Filter = Point, Compression = None, Mesh Type = Full Rect.

## Collision

On the `Walls` tilemap: `TilemapCollider2D` → set *Composite Operation* = **Merge** and add `CompositeCollider2D` (adds a Rigidbody2D → set Body Type **Static**). Merged outlines stop characters snagging on tile seams and are much cheaper.

Per-tile collider type: Tile asset `Collider Type` = Sprite / Grid / None.

## Rule Tiles (2D Tilemap Extras package)

Create → 2D → Tiles → Rule Tile. Define neighbor rules (green arrow = this tile, red X = not this tile) for autotiling. Also: Animated Tile (water, torches), Random Tile, Rule Override Tile (reskin rules).

## Code

```csharp
[SerializeField] private Tilemap ground;
[SerializeField] private TileBase floorTile, wallTile;

public void Generate(bool[,] solid)
{
    int w = solid.GetLength(0), h = solid.GetLength(1);
    var positions = new Vector3Int[w * h];
    var tiles = new TileBase[w * h];
    for (int x = 0, i = 0; x < w; x++)
        for (int y = 0; y < h; y++, i++)
        {
            positions[i] = new Vector3Int(x, y, 0);
            tiles[i] = solid[x, y] ? wallTile : floorTile;
        }
    ground.SetTiles(positions, tiles);   // batch: far faster than SetTile in a loop
    ground.CompressBounds();
}

Vector3Int cell = ground.WorldToCell(worldPos);
Vector3 center = ground.GetCellCenterWorld(cell);
TileBase t = ground.GetTile(cell);
```

Custom data per tile: subclass `Tile` (or RuleTile) with extra fields (`isHazard`, `footstepSound`), or keep a `Dictionary<TileBase, TileData>` lookup.

## Sorting

- Top-down: set `Transparency Sort Mode = Custom Axis (0,1,0)` in URP 2D Renderer / Graphics settings, Tilemap Renderer `Mode = Individual` for tall tiles that characters walk behind; `Chunk` mode for flat ground (faster).
- Use Sorting Layers (Background, Ground, Characters, Foreground) rather than z-position.

## Lighting & URP 2D

URP 2D Renderer: Light 2D (Global, Spot, Freeform), Shadow Caster 2D (Tilemap shadow casting supported via composite shadow caster in recent versions).

## Pitfalls

- Gaps/lines between tiles when the camera moves: use a Sprite Atlas with padding, Pixel Perfect Camera (URP), Point filtering.
- Huge tilemaps: split into chunks or use multiple tilemaps; avoid TilemapCollider2D regenerating every frame by batching edits.
