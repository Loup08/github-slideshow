---
name: procedural-gen
description: Procedural content generation for games — seeded RNG, noise (Perlin/Simplex/value, fBm, domain warp) for terrain, cellular automata caves, BSP and room-and-corridor dungeons, drunkard's walk, wave function collapse, Poisson disk scattering, loot/name generation, and validation (connectivity, solvability). Use for roguelike levels, terrain, worlds, placement, or any generated content.
---

# Procedural Generation

## Principles

1. **Seeded, deterministic RNG** — one master seed, derived sub-seeds per system (`hash(seed, "dungeon", floor)`), so runs are reproducible and shareable. Never use the global unseeded `random` in generators.
2. **Generate → validate → fix or reroll.** Check connectivity, reachability of exit, item placement, difficulty budget.
3. **Hand-authored + procedural** (chunks/rooms designed by hand, arranged procedurally) usually beats pure procedural for quality.
4. Generate **data** (grids, graphs), then render into the engine (tilemap, meshes).

## Noise terrain

```python
# fBm on top of any 2D noise(x, y) in [-1, 1] (e.g. OpenSimplex, FastNoiseLite)
def fbm(x, y, octaves=5, lacunarity=2.0, gain=0.5, freq=0.01):
    amp, total, norm = 1.0, 0.0, 0.0
    for _ in range(octaves):
        total += noise(x * freq, y * freq) * amp
        norm += amp; amp *= gain; freq *= lacunarity
    return total / norm

h = fbm(x, y)
biome = "water" if h < -0.2 else "sand" if h < -0.1 else "grass" if h < 0.4 else "rock"
```

- Combine a height map with a moisture map for biomes (Whittaker diagram).
- **Domain warping** (`noise(x + fbm(x,y)*k, y + ...)`) for organic shapes.
- Island mask: subtract distance from center.
- Engines: Godot `FastNoiseLite`, Unity `Mathf.PerlinNoise` / FastNoiseLite C#, Unreal noise in PCG framework, three.js `simplex-noise` package.

## Cellular automata caves

```python
grid = [[rng.random() < 0.45 for _ in range(W)] for _ in range(H)]   # True = wall
for _ in range(5):
    new = copy(grid)
    for y in range(H):
        for x in range(W):
            walls = count_walls_around(grid, x, y)   # 8 neighbors; out-of-bounds counts as wall
            new[y][x] = walls >= 5 or (walls == 0 and FILL_EMPTY)
    grid = new
# then: flood-fill regions, keep the largest or connect regions with tunnels
```

## Dungeons

- **BSP**: recursively split the map rectangle (random axis, min size), place a room in each leaf, connect sibling leaves with corridors. Guarantees non-overlap and connectivity.
- **Room placement + MST**: scatter rooms, reject overlaps, Delaunay triangulate room centers, minimum spanning tree for guaranteed connectivity, add ~15% extra edges for loops, carve L-shaped corridors.
- **Drunkard's walk**: random walker carves floor until X% open — organic caverns.
- **Graph-first** (mission grammar): generate the abstract graph (start → key → locked door → boss → exit), then embed into space. Best for lock-and-key progression.
- Spelunky-style: grid of hand-made room templates chosen to guarantee a path from top to bottom.

## Wave Function Collapse

Tile-based constraint solver from adjacency rules (learned from a sample or defined manually). Pick the lowest-entropy cell, collapse to a weighted tile, propagate constraints, backtrack/restart on contradiction. Great for towns, interiors, decorative coherence; needs constraints for global structure (paths, borders).

## Placement

- **Poisson disk sampling** (Bridson) for natural non-overlapping trees/rocks/enemies with minimum spacing.
- Spawn tables weighted by depth/biome: `{ "slime": 50, "bat": 30, "orc": depth*5 }`.
- Difficulty budget per room: sum of enemy costs ≤ budget(floor).
- Keep spawns away from the player start and in line with level flow.

## Validation

```python
def reachable(grid, start):
    seen, stack = {start}, [start]
    while stack:
        x, y = stack.pop()
        for nx, ny in neighbors4(x, y):
            if is_floor(grid, nx, ny) and (nx, ny) not in seen:
                seen.add((nx, ny)); stack.append((nx, ny))
    return seen
assert exit_pos in reachable(grid, start_pos)
```

Also: puzzle solvability (run a solver), item/key ordering, minimum path length, no softlocks. Generate 10 000 seeds offline in a test and assert invariants.

## Text & loot

- Names: Markov chains on syllables, or template grammars (Tracery-style): `"{adjective} {noun} of {place}"`.
- Loot affixes: rarity roll → base item → N affixes from weighted pools, with tier ranges scaled by item level.

## Performance

Generate in chunks (infinite worlds), on a background thread if the engine allows, and cache by chunk coordinate + seed. Make generation time measurable; budget < 100 ms per chunk for seamless streaming.
