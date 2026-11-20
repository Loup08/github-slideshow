---
name: unreal-blueprints
description: Build gameplay in Unreal Engine 5 Blueprints cleanly — class choice (Actor, Pawn, Character, GameMode, PlayerController, GameState, PlayerState), event graph vs functions, interfaces and event dispatchers, components, timers, avoiding Tick and casting abuse, and when to move to C++. Use for any Blueprint gameplay work.
---

# Unreal Blueprints

## Gameplay framework — where logic goes

| Class | Responsibility | Exists on |
|---|---|---|
| `GameMode` | Rules, spawning, win/lose | Server only |
| `GameState` | Replicated match state (score, timer) | All |
| `PlayerController` | Input routing, UI ownership, possession | Owning client + server |
| `PlayerState` | Replicated per-player data (name, score) | All |
| `Pawn` / `Character` | The body: movement, mesh, abilities | All |
| `HUD` / UMG Widgets | UI | Local client |
| `GameInstance` (+ Subsystems) | Persists across level loads | Local |

Characters use `CharacterMovementComponent` (walking, jumping, crouching, swimming, network prediction built in).

## Graph hygiene

- **Functions** for reusable, pure, or returning logic (local variables, no latent nodes). **Events** for things that need Delay/Timeline/latent actions.
- **Macros** sparingly; **Collapsed graphs** for readability only.
- Comment boxes (C), reroute nodes, align (Q). One graph shouldn't need horizontal scrolling.
- Mark variables *Instance Editable* + *Expose on Spawn* for designer setup; categories for organization.

## Communication without hard casts

- **Blueprint Interfaces**: `BPI_Interactable` with `Interact(Instigator)`. Call via *Message* node on any Actor — no cast, no hard reference.
- **Event Dispatchers**: `OnDied` on the enemy; GameMode binds to it.
- **Components**: `BP_HealthComponent` reusable across Actors; find with *Get Component by Class*.
- Casting to a Blueprint class creates a **hard reference** that loads that asset (and its meshes/sounds) into memory with the caller. Cast to native C++ base classes or interfaces instead.

## Avoid Tick

- Disable *Start with Tick Enabled* by default.
- Use **Timers** (`Set Timer by Event` with Looping) for periodic logic, Timelines for animated values, overlap events for proximity, and delegates for state changes.
- If Tick is needed, lower *Tick Interval*.

## Common recipes

**Damage**: `Apply Damage` → target's `Event AnyDamage` (or `Apply Point Damage`/`Radial Damage`). Built in, works with any Actor.

**Spawning**: `Spawn Actor from Class` with Spawn Transform; set *Collision Handling Override*; expose-on-spawn variables are set before `BeginPlay`.

**Traces**: `Line Trace by Channel` (Visibility/Camera or a custom *Trace Channel* defined in Project Settings → Collision), `Sphere Trace`, `Multi` variants. Draw Debug Type for visualization.

**Saving**: `SaveGame` subclass + `Save Game to Slot` / `Load Game from Slot` (async versions available).

**UI**: create widget in PlayerController `BeginPlay` → Add to Viewport; bind via events, not *property binding* functions (those run every frame).

## Data

Data Tables (struct row type, import CSV/JSON) and **Primary Data Assets** for items/enemies; Soft Object References + async load for heavy assets.

## When to move to C++

Heavy math loops, many-actor iteration per frame, core systems reused everywhere, custom movement, or anything needing source control merges (Blueprints are binary). A common split: C++ base classes with `BlueprintCallable`/`BlueprintImplementableEvent`, Blueprint children for content and tuning. See `unreal-cpp-gameplay`.

## Debugging

Breakpoints (F9) on nodes, *Debug Filter* to pick an instance, Print String with keys, Visual Logger, `stat unit`/`stat game` for cost. Blueprint profiling: *Tools → Session Frontend → Insights* (`Unreal Insights` with `-trace=default`).

## Pitfalls

- Logic in Level Blueprint isn't reusable — prefer Actors.
- Get All Actors of Class every frame is slow — cache or use subsystems/registries.
- Accessed None errors: validate with *Is Valid* (macro version) before using references.
