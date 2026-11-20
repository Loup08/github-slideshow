---
name: ai-behavior-trees-utility-ai
description: Implement behavior trees and utility AI from scratch or with engine tools — node types (selector, sequence, parallel, decorators, conditions, actions), running status and ticking, blackboards, reactive aborts, utility scoring with response curves and considerations, combining utility selection with BT execution, GOAP overview, debugging and data-driven authoring. Use when an FSM is no longer enough for enemy/NPC decision-making in any engine.
---

# Behavior Trees & Utility AI

## Behavior tree core

Each node returns **SUCCESS**, **FAILURE**, or **RUNNING** when ticked.

```python
SUCCESS, FAILURE, RUNNING = "S", "F", "R"

class Sequence:                      # AND: run children in order until one fails
    def __init__(self, *children): self.children, self.i = children, 0
    def tick(self, ctx):
        while self.i < len(self.children):
            status = self.children[self.i].tick(ctx)
            if status == RUNNING: return RUNNING
            if status == FAILURE: self.i = 0; return FAILURE
            self.i += 1
        self.i = 0
        return SUCCESS

class Selector:                      # OR / priority list; re-evaluated from the top each tick (reactive)
    def __init__(self, *children): self.children = children
    def tick(self, ctx):
        for c in self.children:
            status = c.tick(ctx)
            if status != FAILURE: return status
        return FAILURE

class Condition:
    def __init__(self, fn): self.fn = fn
    def tick(self, ctx): return SUCCESS if self.fn(ctx) else FAILURE

class Action:
    def __init__(self, fn): self.fn = fn     # fn returns S/F/R
    def tick(self, ctx): return self.fn(ctx)

class Cooldown:                      # decorator
    def __init__(self, child, seconds): self.child, self.cd, self.ready_at = child, seconds, 0
    def tick(self, ctx):
        if ctx.time < self.ready_at: return FAILURE
        s = self.child.tick(ctx)
        if s == SUCCESS: self.ready_at = ctx.time + self.cd
        return s
```

```python
tree = Selector(
    Sequence(Condition(lambda c: c.bb["hp"] < 0.25), Action(flee)),
    Sequence(Condition(sees_player), Selector(
        Sequence(Condition(in_attack_range), Cooldown(Action(attack), 1.2)),
        Action(chase))),
    Sequence(Condition(has_last_known), Action(investigate)),
    Action(patrol),
)
```

- **Blackboard**: shared key/value memory per agent (target, last_known_pos, hp) — conditions read it, sensors/services write it.
- **Reactivity**: a reactive selector re-ticks higher-priority branches each tick and **aborts** the running lower branch when a higher one succeeds/runs (call `abort()`/`reset()` on the interrupted subtree so actions clean up — stop moving, cancel animation).
- Decorators: Inverter, Succeeder, Repeat(n), Timeout, Cooldown, Limit.
- Parallel: run main action + background (e.g. move while aiming); define success policy.
- Tick rate: 5–20 Hz is enough for most AI; actions themselves can update every frame.

Engine options: Unreal Behavior Trees (see `unreal-behavior-trees`), Godot **LimboAI** or **Beehave** addons, Unity Behavior package (graph-based) or NodeCanvas/Behavior Designer, Roblox: BehaviorTree3 modules.

## Utility AI

Score each candidate action 0–1 from **considerations** with response curves; pick the highest (or weighted random among the top).

```python
def linear(x, m=1, b=0): return max(0, min(1, m * x + b))
def quadratic(x, k=2): return max(0, min(1, x ** k))
def logistic(x, k=10, mid=0.5): return 1 / (1 + math.exp(-k * (x - mid)))

ACTIONS = {
  "attack":  [lambda c: logistic(1 - c.dist_to_player / 10), lambda c: c.hp_ratio ** 0.5],
  "heal":    [lambda c: quadratic(1 - c.hp_ratio, 3), lambda c: 1.0 if c.potions else 0.0],
  "flee":    [lambda c: quadratic(1 - c.hp_ratio, 2), lambda c: linear(c.enemies_near / 5)],
  "wander":  [lambda c: 0.1],
}

def score(considerations, ctx):
    s = 1.0
    for f in considerations:
        s *= f(ctx)
        if s == 0: break
    n = len(considerations)                         # compensate multiplying many factors
    mod = 1 - 1 / n
    return s + (1 - s) * mod * s

best = max(ACTIONS, key=lambda a: score(ACTIONS[a], ctx) * weight[a])
```

- Normalize inputs to 0–1 (distance / max_distance, hp / max_hp).
- **Momentum/inertia**: add a bonus (e.g. +10–25%) to the current action to prevent rapid flip-flopping.
- Re-evaluate on a timer (0.2–0.5 s) or on significant events.
- Great for sims, companions, RTS units, and enemies with many options; designers tune curves, not code.

## Combining

Common hybrid: **utility selects the goal/behavior**, **BT or FSM executes it** (e.g. utility picks "Heal", a small subtree handles find cover → drink → return). Or a BT with a *utility selector* composite.

## GOAP (brief)

Actions with preconditions/effects over world-state; an A* planner finds a sequence reaching a goal (e.g. `has_weapon & near_target → kill`). Powerful for emergent behavior; harder to debug and tune. Consider HTN for more designer control.

## Debugging & authoring

- Draw the active branch / chosen action with scores above each agent.
- Log transitions with reasons ("flee: score 0.82 (hp 0.18)").
- Author trees in data (JSON/resources) or the engine editor so designers can iterate without code.
- Unit-test conditions and utility curves with fixed contexts.
