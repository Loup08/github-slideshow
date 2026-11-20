---
name: unreal-niagara
description: Create VFX with Unreal Engine 5 Niagara — systems vs emitters, module stack (spawn, update, render), user parameters set from Blueprint/C++, GPU vs CPU sim, collisions and events, ribbons and mesh renderers, pooling, scalability and performance budgets. Use for impacts, muzzle flashes, magic, trails, weather, and environment VFX.
---

# Unreal Niagara VFX

## Structure

- **Niagara System** (`NS_Impact`) — what you spawn; contains one or more **Emitters** (sparks, smoke, decal flash).
- Emitter stack groups: *Emitter Spawn/Update* (once / per frame for the emitter), *Particle Spawn*, *Particle Update*, *Event Handlers*, *Render*.
- Start from templates (Fountain, Burst, Hanging Particulates) and modify.

## Typical impact burst

1. Emitter Properties: Local Space off, Sim Target **GPU** for many particles (> few hundred) or CPU when you need events/collision with physics queries.
2. Emitter Update: *Spawn Burst Instantaneous* (count 30), loop behavior *Once*.
3. Particle Spawn: *Initialize Particle* (lifetime 0.3–0.6, size, color), *Add Velocity in Cone* (cone axis = user param `ImpactNormal`).
4. Particle Update: *Gravity Force*, *Drag*, *Scale Color* (fade alpha by Normalized Age), *Scale Sprite Size* by curve.
5. Render: *Sprite Renderer* with additive material, or *Mesh Renderer* for debris, *Ribbon Renderer* for trails.

## User parameters (drive from gameplay)

Add `User.ImpactNormal` (Vector), `User.Color` (Linear Color), `User.Intensity` (Float).

```cpp
#include "NiagaraFunctionLibrary.h"
#include "NiagaraComponent.h"

UNiagaraComponent* FX = UNiagaraFunctionLibrary::SpawnSystemAtLocation(
    GetWorld(), ImpactFX, Hit.ImpactPoint, Hit.ImpactNormal.Rotation(),
    FVector(1), /*bAutoDestroy*/true, /*bAutoActivate*/true, ENCPoolMethod::AutoRelease);
if (FX) FX->SetVariableLinearColor(TEXT("User.Color"), TeamColor);
```

Blueprint: *Spawn System at Location* / *Spawn System Attached* → *Set Niagara Variable (Float/Vector/LinearColor)*. Module: `"Niagara"` in Build.cs.

Use `ENCPoolMethod::AutoRelease` for frequent one-shots (pooling avoids spawn cost).

## Data from game to particles

- **Data Interfaces**: Skeletal Mesh (spawn on bones/surface), Static Mesh, Spline, Collision Query, Curl Noise, Audio Spectrum, Array (feed positions from code).
- **Niagara Data Channels** (UE 5.3+) to batch many small effects (bullet impacts) into one system instead of spawning hundreds of components.

## Collision & events

- *Collision* module: GPU uses depth buffer / distance fields (cheap, screen-space limits); CPU uses ray traces (accurate, costly).
- Events (CPU only): *Generate Collision Event* → another emitter's *Event Handler* spawns splashes where particles hit.

## Materials

Particle materials: Unlit/Translucent or Additive, use *Particle Color* and *Dynamic Parameter* nodes for per-particle variation, SubUV (flipbook) for animated smoke/fire. Soft particles via *DepthFade* to avoid hard intersections.

## Performance

- Set **Fixed Bounds** on GPU emitters (otherwise culling breaks or costs readback).
- Scalability: Effect Types (`EffectType_Impact`) with per-platform max distance, max instances, cull reactions; set *Detail Level / Scalability* per emitter.
- Overdraw is the main GPU cost: fewer, smaller translucent sprites; use the *Quad Overdraw* view mode.
- Profile with `stat Niagara`, `fx.Niagara.Debug.Enabled`, Niagara Debugger (Tools).

## Pitfalls

- System not visible: emitter disabled, zero spawn rate, wrong space (local vs world), bounds culled.
- Effects keep running forever: loop behavior *Infinite* with no deactivate call — use *Once* for one-shots or call `Deactivate()`.
- Spawning a new component per bullet without pooling → hitches.
