---
name: level-design
description: Level design principles for any engine and genre — goals and flow, teaching mechanics without text (introduce, practice, twist, master), pacing and intensity curves, guiding the player with landmarks/light/lines, gating and loops (hub, metroidvania), greyboxing and metrics, encounter design, secrets and rewards, difficulty ramps, and playtesting. Use when designing, critiquing, or building game levels.
---

# Level Design

## Start from metrics

Before building, measure the player's capabilities: run speed, jump height & distance, dash length, attack range, camera view size. Write them down (`metrics.md`) and build to them:

- Gaps: comfortable at ~70% of max jump distance, challenging at ~90%.
- Corridors ≥ 2× character width (3D: 3–4 m for third-person cameras).
- Cover height (shooters): crouch vs standing cover clearly distinct.

## Teach without text: 4-step pattern

1. **Introduce** in a safe space (failure costs nothing).
2. **Develop/practice** with mild risk.
3. **Twist**: combine with another mechanic or add pressure.
4. **Master/test**: a culminating challenge.

One new idea per segment. Use the environment to demonstrate (enemy walks into spikes before the player does).

## Flow & guidance

- Clear goal visible early (tower in the distance, door glowing).
- **Weenies/landmarks** for orientation; light, color, contrast, and motion attract attention.
- **Leading lines** (paths, cables, edges) point to the critical path.
- Breadcrumbs: pickups/coins along the intended route.
- "Gating" signals: locked doors with visible keys elsewhere, unreachable ledges hinting at future abilities.
- Avoid dead ends without reward; loop back to earlier areas with shortcuts (unlock from the far side).

## Pacing

Alternate tension and release: combat → exploration/rest → puzzle → set piece. Plot intensity over the level; it should rise with valleys, peaking near the end. Place checkpoints before hard sections and after long uninterrupted challenges.

## Structure patterns

- **Linear** (platformers, story shooters): tight pacing control.
- **Hub & spoke**: central area with branching levels; easy to gate progression.
- **Metroidvania**: interconnected map gated by abilities; design the lock/key graph first, then space.
- **Open/sandbox**: points of interest spaced so a new one is visible from each one ("rule of the next thing").
- **Arenas**: encounter spaces with clear boundaries, cover, verticality, and flanking routes.

## Encounter design

- Mix enemy roles (melee rusher, ranged, tank, support) to force decisions.
- Telegraph spawns; avoid spawning behind the player without warning.
- Give the player the first move (enemies unaware, elevated entry point).
- Provide resources (ammo/health) proportional to the difficulty just before or after.

## Secrets & rewards

Reward curiosity: off-path areas with collectibles, lore, shortcuts. Hint subtly (cracked wall, odd tile, coin trail). ~10–20% of content optional keeps completionists engaged.

## Workflow

1. Paper sketch / beat chart (what happens where, which mechanic is tested).
2. **Greybox/blockout** with primitives or placeholder tiles — no art.
3. Playtest the greybox; iterate layout until fun.
4. Art pass (keep collision simple and readable — art must not hide walkable vs non-walkable).
5. Polish: lighting, audio, set dressing, final tuning.

Tools: engine tilemaps, Tiled, LDtk (2D); ProBuilder (Unity), CSG (Godot), BSP/Modeling Mode (Unreal), Blender for modular kits (3D). Build 3D with **modular kits** on a grid (e.g. 2 m or 4 m modules) for fast iteration.

## Difficulty ramp

- Introduce 1 element per level early; combine later.
- Sawtooth curve: each new world starts easier than the previous world's end.
- Optional harder routes for skilled players.

## Playtesting

Watch silently; note where players hesitate, get lost, or die repeatedly. Heatmaps of deaths/time (log positions). If many players miss the critical path, the level is wrong, not the players. Test with fresh players regularly — you are blind to your own levels.
