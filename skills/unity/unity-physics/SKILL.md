---
name: unity-physics
description: Unity 3D (PhysX) and 2D (Box2D) physics — Rigidbody vs CharacterController, kinematic movement, collision layers and matrix, triggers vs collisions, raycasts and overlap queries with non-alloc APIs, interpolation, and continuous collision. Use for collisions, projectiles, character movement, and physics bugs in Unity.
---

# Unity Physics

3D and 2D are **separate** systems: `Rigidbody`/`Collider`/`Physics` vs `Rigidbody2D`/`Collider2D`/`Physics2D`. They never interact.

## Choose movement

| Need | Use |
|---|---|
| Physically simulated objects | `Rigidbody` (dynamic) |
| Player that pushes things, slides on walls, simple | `CharacterController` (3D) + `Move()` |
| Full control 2D platformer | `Rigidbody2D` dynamic with velocity control, or Kinematic + `Cast`/`Slide` |
| Moving platforms, doors | Kinematic Rigidbody + `MovePosition`/`MoveRotation` |

## Rigidbody rules

- Move in `FixedUpdate`. Read input in `Update`, store it, apply in `FixedUpdate`.
- Set `rb.linearVelocity` (Unity 6 name; `velocity` is obsolete) or `AddForce(..., ForceMode.Impulse)` — never `transform.position` on a dynamic body.
- `interpolation = Interpolate` on the player/camera target to remove jitter.
- `collisionDetectionMode = Continuous` (or `ContinuousDynamic`) for fast projectiles.
- Freeze rotation axes for characters.

```csharp
private void FixedUpdate()
{
    Vector3 target = moveInput * speed;
    Vector3 v = rb.linearVelocity;
    Vector3 delta = target - new Vector3(v.x, 0, v.z);
    rb.AddForce(Vector3.ClampMagnitude(delta, maxAccel * Time.fixedDeltaTime), ForceMode.VelocityChange);
}
```

## Layers & collision matrix

Define layers (Player, Enemy, PlayerProjectile, Environment, Trigger). Project Settings → Physics (and Physics 2D) → Layer Collision Matrix: uncheck pairs that never interact. Use `LayerMask` fields in scripts:

```csharp
[SerializeField] private LayerMask groundMask;
bool grounded = Physics.CheckSphere(feet.position, 0.2f, groundMask, QueryTriggerInteraction.Ignore);
```

## Triggers vs collisions

- `OnCollisionEnter(Collision c)` — both have colliders, at least one has a non-kinematic Rigidbody.
- `OnTriggerEnter(Collider other)` — one collider `isTrigger`, at least one Rigidbody (kinematic is fine) involved.
- 2D: `OnCollisionEnter2D`, `OnTriggerEnter2D`.

## Queries

```csharp
if (Physics.Raycast(origin, dir, out RaycastHit hit, 100f, hitMask))
{
    var health = hit.collider.GetComponentInParent<Health>();
    if (health != null) health.TakeDamage(10);
}

private readonly Collider[] buffer = new Collider[32];
int n = Physics.OverlapSphereNonAlloc(pos, radius, buffer, enemyMask);
for (int i = 0; i < n; i++) { /* explosion */ }
```

Avoid `?.` on Unity objects (it skips Unity's destroyed-object check). When the component is on the same object, `hit.collider.TryGetComponent(out Health h)` is the cheapest lookup.

2D: `Physics2D.Raycast` returns `RaycastHit2D` (check `hit.collider`), `Physics2D.OverlapCircle`, and `ContactFilter2D` with list overloads.

## Settings

- Fixed Timestep: 0.02 default (50 Hz); 1/60 for tighter action games.
- 3D gravity -9.81; scale objects to meters. Unity 6 also allows choosing physics simulation mode (FixedUpdate/Update/Script).
- Physics Materials: friction 0 on character colliders to avoid wall sticking.

## Pitfalls

- Moving a static collider (no Rigidbody) each frame is expensive — add a kinematic Rigidbody.
- Non-uniform scale on MeshColliders/primitive colliders gives unexpected shapes.
- MeshCollider must be `convex` to collide with other MeshColliders or be on a dynamic Rigidbody.
