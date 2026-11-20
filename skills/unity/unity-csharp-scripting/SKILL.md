---
name: unity-csharp-scripting
description: Idiomatic Unity 6 C# scripting — MonoBehaviour lifecycle, serialization with [SerializeField], component caching, coroutines vs Awaitable async, events, object pooling, assembly definitions, and allocation-free Update loops. Use for any Unity gameplay script.
---

# Unity C# Scripting (Unity 6 LTS)

## Lifecycle

`Awake` (self-init, cache components) → `OnEnable` (subscribe events) → `Start` (cross-object init) → `FixedUpdate` (physics, fixed step) → `Update` (input, logic) → `LateUpdate` (camera follow) → `OnDisable` (unsubscribe) → `OnDestroy`.

## Component template

```csharp
using System;
using UnityEngine;

[RequireComponent(typeof(Rigidbody2D))]
public sealed class Health : MonoBehaviour
{
    [SerializeField, Min(1)] private int maxHealth = 100;
    [SerializeField] private ParticleSystem hitVfx;

    public int Current { get; private set; }
    public event Action<int, int> Changed;   // current, max
    public event Action Died;

    private void Awake() => Current = maxHealth;

    public void TakeDamage(int amount)
    {
        if (Current <= 0) return;
        Current = Mathf.Max(0, Current - amount);
        Changed?.Invoke(Current, maxHealth);
        if (hitVfx) hitVfx.Play();
        if (Current == 0) Died?.Invoke();
    }
}
```

- `[SerializeField] private` over `public` fields. Use `[field: SerializeField]` for auto-properties.
- `[Header]`, `[Tooltip]`, `[Range]`, `[Min]` for designer-friendly inspectors.
- Unity overloads `==` for destroyed objects: use `if (obj)` / `obj == null`, **not** `obj?.Method()` or `??` on UnityEngine.Object.

## Caching

Cache `GetComponent` in `Awake`. Never call `GetComponent`, `Find`, `FindObjectOfType` (deprecated → `FindFirstObjectByType`/`FindAnyObjectByType`), or `Camera.main` repeatedly inside `Update` (`Camera.main` is cached in recent versions but still cheaper to store).

## Async: Awaitable (Unity 6) vs coroutines

```csharp
private async Awaitable FlashAsync()
{
    sprite.color = Color.white;
    await Awaitable.WaitForSecondsAsync(0.1f, destroyCancellationToken);
    sprite.color = baseColor;
}

private IEnumerator SpawnWaves()
{
    for (int i = 0; i < waves; i++) { Spawn(i); yield return new WaitForSeconds(5f); }
}
```

Pass `destroyCancellationToken` so work stops when the object dies. Coroutines stop when the GameObject is disabled. `Awaitable.NextFrameAsync()`, `FixedUpdateAsync()`, `BackgroundThreadAsync()` / `MainThreadAsync()` for thread hops.

## Object pooling

```csharp
using UnityEngine.Pool;
private ObjectPool<Bullet> pool;
void Awake() => pool = new ObjectPool<Bullet>(
    () => Instantiate(prefab), b => b.gameObject.SetActive(true),
    b => b.gameObject.SetActive(false), b => Destroy(b.gameObject), maxSize: 256);
```

## Project structure

```
Assets/_Project/{Scripts/{Runtime,Editor,Tests}, Prefabs, Scenes, Art, Audio, Data}
```

Add **Assembly Definitions** (`.asmdef`) per folder to cut compile times and enforce dependencies; Editor code in an Editor-only asmdef.

## Enter Play Mode settings

With domain reload disabled (fast play mode), static fields persist between plays — reset them with `[RuntimeInitializeOnLoadMethod(RuntimeInitializeLoadType.SubsystemRegistration)]`.

## Pitfalls

- `Time.deltaTime` in `Update`, `Time.fixedDeltaTime` implicitly in `FixedUpdate` (use `Time.deltaTime` there too — Unity returns the fixed value).
- Allocations per frame (string concat, LINQ, `new List`, boxing) → GC spikes. Use the Profiler's GC Alloc column.
- Script execution order between objects is undefined; use `Awake`/`Start` split or `[DefaultExecutionOrder]`.
- Serialized field renames lose data — use `[FormerlySerializedAs("oldName")]`.
