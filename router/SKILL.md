---
name: router
description: Master router for game development. Use at the start of ANY game-dev request (Godot, Unity, Unreal, Ebitengine, Phaser, PixiJS, three.js, Bevy, pygame, LÖVE, Defold, Roblox, or engine-agnostic design) to detect the engine, the task type and the genre, then load the 1-4 specific skills that cover it instead of answering from memory.
---

# Game-Dev Router

Detect **engine → task → genre → workflow**, then load the matching skills (by name, with the Skill tool) before writing code. Load the fewest skills that cover the request: usually one engine skill + at most one discipline + at most one genre/workflow.

## Step 1 — Detect the engine from the project

Check files before asking the user. First hit wins.

| Evidence in the repo | Engine | Skill prefix |
|---|---|---|
| `project.godot`, `*.gd`, `*.tscn`, `*.tres` | Godot 4.x | `godot-*` |
| `*.csproj` + `project.godot` | Godot C# | `godot-csharp` (+ other `godot-*`) |
| `ProjectSettings/ProjectVersion.txt`, `Assets/`, `*.unity`, `*.asmdef` | Unity | `unity-*` |
| `*.uproject`, `Source/*.Build.cs`, `Content/*.uasset` | Unreal Engine 5 | `unreal-*` |
| `go.mod` requiring `github.com/hajimehoshi/ebiten/v2` | Ebitengine | `ebitengine-game` |
| `package.json` with `phaser` | Phaser | `phaser-core`, `phaser-arcade-physics` |
| `package.json` with `pixi.js` | PixiJS | `pixijs-rendering` |
| `package.json` with `three` | three.js | `threejs-*` |
| `Cargo.toml` with `bevy` | Bevy | `bevy-ecs` |
| `import pygame` / `pygame-ce` in requirements | pygame-ce | `pygame-core` |
| `main.lua` + `conf.lua` / `love.load` | LÖVE | `love2d-core` |
| `game.project`, `*.script`, `*.collection` | Defold | `defold-core` |
| `default.project.json` (Rojo), `*.luau`, `ServerScriptService` | Roblox | `roblox-*` |

No project yet? Ask once which engine they want, or recommend: **Godot** (general 2D/3D indie), **Phaser** (browser 2D), **three.js** (browser 3D), **pygame/LÖVE** (learning, tiny scope), **Unity/Unreal** (team already uses it), **Roblox** (social/UGC platform).

## Step 2 — Map the task to skills

| The user wants… | Engine-specific | Cross-engine discipline |
|---|---|---|
| Player movement, jump, controls | `godot-2d-movement`, `unity-input-system`, `unreal-enhanced-input`, `phaser-arcade-physics` | `input-systems`, `game-feel`, `physics-tuning` |
| Collisions, rigidbodies, raycasts | `godot-physics`, `unity-physics`, `roblox-physics` | `physics-tuning` |
| Enemies, NPC behavior, pathfinding | `unity-navmesh`, `unreal-behavior-trees` | `game-ai`, `ai-behavior-trees-utility-ai` |
| Levels, tilemaps | `godot-tilemap`, `unity-tilemap-2d` | `level-design`, `procedural-gen` |
| Menus, HUD, inventory UI | `godot-ui-control`, `roblox-ui` | `game-ui-ux` |
| Animation, sprites, state machines | `godot-animation`, `unity-animation` | `game-feel` |
| Shaders, VFX | `godot-shaders`, `unreal-niagara`, `threejs-materials-lighting` | `shader-programming` |
| Save / load, persistence | `godot-resources`, `unity-scriptableobjects`, `roblox-datastores` | `save-systems` |
| Sound, music | `godot-audio` | `audio-design` |
| Online multiplayer | `godot-multiplayer`, `roblox-networking` | — |
| Camera follow, shake, framing | — | `camera-systems` |
| Dialogue, quests, branching story | — | `dialogue-systems` |
| Art, sprites, placeholder assets | — | `create-game-assets` |
| Slow frame rate, memory, GC spikes | `unity-build-size` (size only) | `performance-optimization` |
| Build / export / ship | `godot-export`, `unity-build-pipeline`, `unreal-packaging` | `steam-publish`, `itch-publish` |
| Automated tests | `godot-gdscript-headless-testing`, `ebitengine-game` | — |

## Step 3 — Genre and workflow overlays

Genre skills are **compositional**: they list the systems a genre needs and point back to discipline/engine skills. Load one when the user names a genre or describes its core loop.

- Jump-and-run → `platformer` · permadeath runs, procedural floors → `roguelike` · stats, quests, inventory → `rpg`
- First-person combat → `fps-shooter` · waves + placed towers → `tower-defense` · decks, hands, turns → `card-game`
- Branching story with portraits → `visual-novel` · gather/craft/hunger → `survival-crafting` · rules-based levels → `puzzle`

Workflow skills shape *how* to work: `game-jam` (time-boxed), `prototype-fast` (find the fun), `game-docs` (GDD/one-pager), `steam-publish`, `itch-publish`.

## Step 4 — Execute

1. Load the selected skills; state in one line which ones and why.
2. Follow the engine skill for API correctness and the discipline skill for design defaults.
3. Prefer small, runnable increments: get one playable loop working, then iterate.
4. Verify: run the engine's headless/test command when one exists (`godot --headless`, `go test`, `cargo test`, `npm run build`), otherwise explain exactly how to test in-editor.

## Rules

- Never mix engine APIs (e.g. Unity `Update()` advice in a Godot project). If the engine is unclear, ask.
- Version-sensitive APIs: the skills target the versions in the marketplace description; if the project pins a different version (check `project.godot` `config/features`, `ProjectVersion.txt`, `package.json`, `Cargo.toml`), follow the project.
- Do not recreate a specific existing commercial game (names, characters, art, levels). Genre mechanics are fine.
