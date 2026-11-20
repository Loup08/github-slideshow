---
name: unity-navmesh
description: AI navigation in Unity with the AI Navigation package — NavMeshSurface baking (incl. runtime), NavMeshAgent movement and tuning, NavMeshObstacle carving, off-mesh links, areas and costs, and custom character control with agent paths. Use for enemy pathfinding and NPC movement in 3D Unity projects.
---

# Unity NavMesh (AI Navigation package)

Install **AI Navigation** (com.unity.ai.navigation). The legacy Navigation window bake is deprecated; use components.

## Baking

1. Add `NavMeshSurface` to a level root. Collect Objects: *All* / *Children* / *Volume*; Use Geometry: *Render Meshes* or *Physics Colliders* (colliders are usually cleaner).
2. Agent Type (Navigation → Agents): radius, height, step height, max slope — match your character.
3. Bake. The data is stored in an asset next to the scene.
4. Exclude objects: `NavMeshModifier` with *Ignore From Build*, or override Area (e.g. `Water` cost 5, `Not Walkable`).

Runtime bake (procedural levels): `surface.BuildNavMesh();` once after generation; `surface.UpdateNavMesh(surface.navMeshData)` async for changes.

## Agent movement

```csharp
[RequireComponent(typeof(NavMeshAgent))]
public sealed class Chaser : MonoBehaviour
{
    [SerializeField] private Transform target;
    [SerializeField] private float repathInterval = 0.25f;
    private NavMeshAgent agent;
    private float timer;

    void Awake() => agent = GetComponent<NavMeshAgent>();

    void Update()
    {
        timer -= Time.deltaTime;
        if (timer <= 0f) { timer = repathInterval; agent.SetDestination(target.position); }
        bool arrived = !agent.pathPending && agent.remainingDistance <= agent.stoppingDistance;
    }
}
```

Don't call `SetDestination` every frame for many agents — throttle.

Tuning: `speed`, `angularSpeed` (raise to 720+ for snappy turning), `acceleration` (high for responsive stop), `stoppingDistance`, `autoBraking` (off for patrol loops), `obstacleAvoidanceType` + `avoidancePriority` (lower number = more important).

## Driving a custom controller

Let the agent compute paths but move with your Rigidbody/Animator:

```csharp
agent.updatePosition = false;
agent.updateRotation = false;
Vector3 desired = agent.desiredVelocity;          // feed to animator / controller
agent.nextPosition = transform.position;          // keep agent in sync
```

Or use root motion: set `agent.velocity` from `animator.deltaPosition / Time.deltaTime` in `OnAnimatorMove`.

## Obstacles & links

- `NavMeshObstacle` with **Carve** for objects that move occasionally (doors, crates); "Carve Only Stationary" to avoid constant re-carving.
- `NavMeshLink` for jumps, ladders, gaps. Detect traversal with `agent.isOnOffMeshLink` and animate manually (`autoTraverseOffMeshLink = false`, then `CompleteOffMeshLink()`).

## Queries

```csharp
NavMesh.SamplePosition(randomPoint, out NavMeshHit hit, 2f, NavMesh.AllAreas);  // snap to mesh
var path = new NavMeshPath();
bool reachable = NavMesh.CalculatePath(a, b, NavMesh.AllAreas, path) && path.status == NavMeshPathStatus.PathComplete;
```

## 2D

NavMesh is 3D-only out of the box. For 2D top-down: rotate surface (community NavMeshPlus), A* Pathfinding Project, or grid A* (see `game-ai`).

## Pitfalls

- Agent placed off the mesh → "SetDestination can only be called on an active agent that has been placed on a NavMesh". Use `agent.Warp(sampledPosition)`.
- Agent + non-kinematic Rigidbody fight each other — make the Rigidbody kinematic.
- Agents jittering in groups: reduce `radius`, stagger destinations, increase `stoppingDistance`.
