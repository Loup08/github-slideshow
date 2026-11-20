---
name: game-ai
description: Engine-agnostic game AI — finite state machines, steering behaviors (seek, flee, arrive, separation), grid A* and navmesh pathfinding, perception (sight cones, hearing, memory), group tactics and tokens, difficulty tuning and making AI fun rather than optimal. Use when designing enemies, NPCs, companions, or opponents in any engine.
---

# Game AI

Goal: AI that is **readable, fair, and fun**, not optimal. Telegraph intentions, give the player windows, and cheat in their favor when needed.

## Pick the architecture

| Complexity | Technique |
|---|---|
| 2–6 behaviors, clear transitions | **Finite State Machine** |
| Many reactive priorities, reusable sub-behaviors | **Behavior Tree** (see `ai-behavior-trees-utility-ai`) |
| Many competing options scored by context | **Utility AI** |
| Long multi-step plans (stealth, sims) | GOAP / HTN |

Start with an FSM; switch only when it hurts.

## FSM (engine-agnostic pseudocode)

```python
class Enemy:
    def __init__(self):
        self.state = "patrol"; self.state_time = 0

    def update(self, dt):
        self.state_time += dt
        getattr(self, f"update_{self.state}")(dt)

    def change(self, new):
        getattr(self, f"exit_{self.state}", lambda: None)()
        self.state, self.state_time = new, 0
        getattr(self, f"enter_{new}", lambda: None)()

    def update_patrol(self, dt):
        self.follow_waypoints(dt)
        if self.can_see(player): self.change("alert")

    def update_alert(self, dt):            # telegraph: "!" + 0.5 s pause before chasing
        if self.state_time > 0.5: self.change("chase")

    def update_chase(self, dt):
        self.move_toward(player.pos, dt)
        if self.dist(player) < self.attack_range: self.change("windup")
        elif not self.can_see(player) and self.state_time_since_seen > 3: self.change("search")

    def update_windup(self, dt):           # visible anticipation = fairness
        if self.state_time > 0.4: self.change("attack")
```

Each state: enter (anim, sound), update, exit; transitions explicit. Add hysteresis (different enter/exit distances) to avoid flicker.

## Steering behaviors

```python
def seek(pos, vel, target, max_speed, max_force):
    desired = (target - pos).normalized() * max_speed
    return (desired - vel).clamp_length(max_force)

def arrive(pos, vel, target, max_speed, max_force, slow_radius):
    to = target - pos; d = to.length()
    speed = max_speed * min(d / slow_radius, 1.0)
    desired = to.normalized() * speed if d > 0 else Vector()
    return (desired - vel).clamp_length(max_force)

def separation(pos, neighbors, radius):
    force = Vector()
    for n in neighbors:
        off = pos - n.pos; d = off.length()
        if 0 < d < radius: force += off.normalized() * (radius - d) / radius
    return force
```

Sum weighted forces → velocity. Separation prevents enemy blobs stacking. Flee, wander, pursue (seek predicted position), obstacle avoidance follow the same pattern.

## Pathfinding

- **Grid A\***: 4/8-neighbor grid, Manhattan (4-dir) or octile (8-dir) heuristic, binary heap open set. Cache paths; re-path on a timer (0.25–0.5 s), not every frame.
- **Navmesh**: use the engine's (Godot NavigationServer, Unity AI Navigation, Unreal Recast, Roblox PathfindingService).
- **Flow fields** for hundreds of units toward one target (RTS, tower defense, horde games): one BFS/Dijkstra from the goal, every unit reads the gradient.
- Path smoothing: string-pulling / line-of-sight checks to skip waypoints.

## Perception

- Sight: distance check → angle check (`dot(forward, dir_to_player) > cos(fov/2)`) → raycast for occlusion (cheapest first). Run at 5–10 Hz, not every frame.
- Hearing: gameplay events emit noise with radius (footsteps, gunshots); AI within radius investigates the source position.
- Memory: last known position + timestamp; search behavior when lost.
- Awareness meter (0→1 fill) instead of instant detection for stealth games.

## Groups & fairness

- **Attack tokens**: only N enemies may attack the player at once; others circle/reposition (classic melee brawler trick).
- Stagger reaction times and accuracy; first shots miss intentionally ("near-miss" for tension).
- Spread enemies around the player using slots on a ring.
- Announce flanks with barks/audio.

## Difficulty

Tune reaction time, accuracy, aggression, and token count — not just HP/damage. Dynamic difficulty: ease off after repeated deaths (subtle). Playtest: if players can't explain why the AI did something, it's not readable enough.

## Debugging

Draw state labels, sight cones, paths and targets in debug mode. Log state transitions. Most "AI bugs" are perception or transition bugs.
