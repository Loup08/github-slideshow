---
name: roblox-physics
description: Roblox physics — assemblies and anchoring, mover constraints (LinearVelocity, AngularVelocity, AlignPosition, AlignOrientation, VectorForce) replacing legacy BodyMovers, mechanical constraints and vehicles, collision groups, raycasting and spatial queries, network ownership, CFrame math, and stability tips. Use for vehicles, projectiles, moving platforms, ragdolls, or collision setup on Roblox.
---

# Roblox Physics

## Basics

- `Anchored = true` → not simulated (static world). Unanchored parts joined by welds form an **assembly** with one root part.
- Read/write velocity: `part.AssemblyLinearVelocity`, `AssemblyAngularVelocity` (`Velocity` is deprecated).
- Impulses: `part:ApplyImpulse(dir * mass * speed)`, `ApplyAngularImpulse`. Mass: `part.AssemblyMass`.
- Materials & `CustomPhysicalProperties` (density, friction, elasticity).
- Moving anchored parts by setting CFrame each frame teleports them — players don't ride along. Use constraints or `TweenService` on a welded model with `AlignPosition`, or `PrismaticConstraint` with servo actuator for elevators.

## Mover constraints (modern)

All need `Attachment`s.

| Constraint | Use |
|---|---|
| `LinearVelocity` | Constant velocity (conveyors, projectiles, hover) |
| `AngularVelocity` | Spin |
| `VectorForce` | Thrust, custom gravity (`Force = Vector3.new(0, mass*workspace.Gravity, 0)` cancels gravity) |
| `AlignPosition` / `AlignOrientation` | Move/rotate toward target smoothly (platforms, held objects, pets following) |
| `LineForce`, `Torque` | Magnets, custom forces |

`BodyVelocity`, `BodyPosition`, `BodyGyro` etc. are legacy — avoid in new work.

```lua
local att = Instance.new("Attachment", platform)
local align = Instance.new("AlignPosition")
align.Mode = Enum.PositionAlignmentMode.OneAttachment
align.Attachment0 = att
align.MaxForce = math.huge
align.Responsiveness = 50
align.Position = targetPosition
align.Parent = platform
```

## Mechanical constraints

`HingeConstraint` (doors, wheels with Motor/Servo actuator), `PrismaticConstraint` (sliders, elevators), `SpringConstraint` (suspension), `RopeConstraint`, `BallSocketConstraint` (ragdolls), `CylindricalConstraint` (steering + wheel), `WeldConstraint`/`RigidConstraint` (glue).

Vehicle recipe: chassis + 4 `CylindricalConstraint` wheels with `AngularActuatorType = Motor` for drive, `SpringConstraint` suspension, `VehicleSeat` (`Throttle`, `Steer`) read by a script; set network owner to the driver.

## Collision groups

```lua
local PhysicsService = game:GetService("PhysicsService")
PhysicsService:RegisterCollisionGroup("Players")
PhysicsService:RegisterCollisionGroup("Projectiles")
PhysicsService:CollisionGroupSetCollidable("Players", "Players", false)   -- no player-player collision
part.CollisionGroup = "Projectiles"
```

`CanCollide`, `CanTouch` (Touched events), `CanQuery` (raycasts) — disable what you don't need (performance).

## Queries

```lua
local params = RaycastParams.new()
params.FilterType = Enum.RaycastFilterType.Exclude
params.FilterDescendantsInstances = { character }
local result = workspace:Raycast(origin, direction * 500, params)
if result then print(result.Instance, result.Position, result.Normal, result.Material) end

local overlap = OverlapParams.new()
local parts = workspace:GetPartBoundsInRadius(center, 10, overlap)   -- explosions, hitboxes
workspace:Blockcast(cframe, size, direction, params)                  -- thick rays
workspace:Spherecast(origin, radius, direction, params)
```

Spatial-query hitboxes (`GetPartBoundsInBox` on a timer) are more reliable than `Touched` for melee combat.

## Projectiles

Fast projectiles: raycast-based (step a ray each frame, e.g. FastCast pattern) instead of physical parts — no tunneling, no ownership issues. Render a cosmetic part on clients.

## CFrame essentials

```lua
CFrame.new(pos)                         -- position
CFrame.lookAt(pos, target)              -- face target
cf * CFrame.new(0, 0, -5)               -- 5 studs forward in local space
cf:ToObjectSpace(other)                 -- relative
model:PivotTo(cf)                       -- move models (not SetPrimaryPartCFrame)
CFrame.Angles(0, math.rad(90), 0)
```

## Ownership & stability

- Server-owned (`SetNetworkOwner(nil)`) for authoritative physics; player-owned for things they directly control.
- Very light parts attached to heavy ones jitter — use `Massless = true` on decorative parts.
- Huge numbers of unanchored parts = server lag; anchor debris after it settles or make it client-only.
- `workspace.Gravity` default 196.2 studs/s²; 1 stud ≈ 0.28 m.
