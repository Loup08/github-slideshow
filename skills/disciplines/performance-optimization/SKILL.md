---
name: performance-optimization
description: Find and fix game performance problems in any engine — profiling first (CPU vs GPU bound, frame time budgets), engine profilers, reducing draw calls (batching, instancing, atlases), overdraw, LOD and culling, object pooling and GC/allocation spikes, physics cost, update throttling, spatial partitioning, loading hitches and shader compilation stutter, memory and texture budgets, and mobile/web specifics. Use when a game stutters, runs at low FPS, uses too much memory, or before shipping.
---

# Performance Optimization

## 1. Measure before changing anything

- Think in **frame time**, not FPS: 60 FPS = 16.67 ms, 30 = 33.3 ms, 120 = 8.33 ms. Budget it (e.g. gameplay 3 ms, physics 2 ms, rendering 8 ms, headroom 3 ms).
- Profile a **release/development build on target hardware**, not the editor.
- Determine the bottleneck:
  - **CPU-bound**: lowering resolution doesn't help; main thread time ≈ frame time.
  - **GPU-bound**: lowering resolution/render scale helps.
  - **Spikes/hitches** (stutters) vs consistently low FPS are different problems — look at frame-time graphs, not averages.

Profilers: Godot (Debugger → Profiler/Visual Profiler/Monitors), Unity Profiler + Frame Debugger + Memory Profiler, Unreal Insights + `stat unit/gpu/scenerendering`, Chrome DevTools Performance + Spector.js (web), RenderDoc/PIX/Nsight (GPU), Roblox MicroProfiler, `perf`/Superluminal/Tracy (native).

## 2. CPU

- **Allocations/GC**: no per-frame allocations in hot paths (strings, closures, LINQ, new arrays/vectors in managed languages). Reuse buffers; object pools for bullets, particles, enemies, damage numbers.
- **Update throttling**: not everything needs every frame — AI perception at 5–10 Hz, pathfinding staggered across frames, far entities updated less often (LOD for logic).
- **Spatial partitioning** for "nearby" queries: uniform grid / spatial hash (2D), BVH/octree; let the physics engine do broadphase queries instead of O(n²) loops.
- **Cache lookups** (component/node lookups, `find` calls).
- Data-oriented layouts (arrays of structs/ECS) for thousands of entities.
- Move heavy work off the main thread (pathfinding, procgen, loading) when the engine allows.
- Physics: simpler colliders (primitives > convex > mesh), fewer layers interacting, larger fixed step if possible, sleeping bodies.

## 3. GPU

- **Draw calls/batches**: use texture atlases, shared materials, static batching, GPU instancing (`MultiMesh`, `InstancedMesh`, `Graphics.RenderMeshInstanced`, HISM in Unreal).
- **Overdraw**: large transparent sprites/particles, full-screen effects stacked. Visualize overdraw; trim sprite meshes (tight/polygon sprites), fewer/smaller particles.
- **Resolution**: dynamic resolution / render scale + upscaling (FSR/DLSS/TSR) for 3D.
- **LOD**: mesh LODs, impostors for far objects, shadow distance limits, fewer shadow-casting lights.
- **Culling**: frustum (automatic), occlusion culling for interiors/cities, distance culling for small props.
- **Shaders**: cheaper variants on mobile; avoid many texture samples and dynamic loops per pixel.
- Post-processing: each pass costs a full-screen read/write — merge or drop on low settings.

## 4. Hitches

- **Instantiation spikes**: pre-instantiate/pool; spread spawning over frames.
- **Loading**: async/background loading with loading screens or streaming; avoid synchronous loads mid-gameplay.
- **Shader compilation stutter**: precompile/warm up (PSO caches in Unreal, shader variant collections in Unity, render everything once during loading in Godot/three.js).
- **GC**: incremental GC (Unity), reduce garbage; trigger collection during loading screens.
- **Audio decoding**: decompress short SFX on load; stream music.

## 5. Memory

- Textures dominate: correct max sizes, GPU compression (BCn desktop, ASTC/ETC2 mobile, KTX2/Basis web), mipmaps for 3D only, no Read/Write copies.
- Unload unused assets between levels; watch for leaks (references kept in static lists/event handlers).
- Budget per platform (e.g. mobile < 1–1.5 GB total for wide device support; web tabs crash earlier).

## 6. Platform specifics

- **Mobile**: thermal throttling — test 15+ min sessions; cap FPS at 30/60 to save battery; fewer lights, baked lighting; avoid alpha-test/discard heavy foliage on tile-based GPUs.
- **Web**: single main thread (unless threads + COOP/COEP); keep downloads small; WebGL draw calls are pricier; avoid `getImageData`/sync GPU readbacks.
- **Low-end PC**: offer scalability presets; test on integrated GPUs.

## Workflow

1. Reproduce with a repeatable benchmark scene (fixed camera path/replay).
2. Profile → identify the single biggest cost.
3. Fix that one thing → re-measure → commit with numbers in the message.
4. Repeat until within budget. Don't micro-optimize code that isn't in the profile.
