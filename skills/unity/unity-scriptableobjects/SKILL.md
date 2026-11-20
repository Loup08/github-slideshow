---
name: unity-scriptableobjects
description: Use ScriptableObjects in Unity for data-driven design — item/enemy definitions, configs, runtime sets, event channels, and their serialization pitfalls (runtime changes persisting in editor, not for save files). Use when designing game data, decoupled architecture, or designer-editable content in Unity.
---

# Unity ScriptableObjects

A `ScriptableObject` (SO) is an asset that holds data (and optionally logic) independent of scenes.

## Data definitions

```csharp
[CreateAssetMenu(fileName = "Weapon", menuName = "Game/Weapon")]
public sealed class WeaponDefinition : ScriptableObject
{
    [field: SerializeField] public string Id { get; private set; }
    [field: SerializeField] public string DisplayName { get; private set; }
    [field: SerializeField] public Sprite Icon { get; private set; }
    [field: SerializeField, Min(0)] public int Damage { get; private set; } = 10;
    [field: SerializeField] public float FireRate { get; private set; } = 0.2f;
    [field: SerializeField] public Projectile ProjectilePrefab { get; private set; }

    private void OnValidate() { if (string.IsNullOrEmpty(Id)) Id = name; }
}
```

Many prefabs reference one asset → memory saved, designers tune in one place.

## Runtime state ≠ definition

Changes made to an SO in Play Mode **persist in the editor asset**; in builds they reset on restart. Never store mutable per-entity state in a shared SO. Pattern:

```csharp
public sealed class WeaponInstance
{
    public WeaponDefinition Def { get; }
    public int Ammo;
    public WeaponInstance(WeaponDefinition def) { Def = def; Ammo = def.MagazineSize; }
}
```

Or `Instantiate(so)` for a runtime clone.

## Event channels

```csharp
[CreateAssetMenu(menuName = "Game/Events/Int Event")]
public sealed class IntEventChannel : ScriptableObject
{
    public event Action<int> Raised;
    public void Raise(int value) => Raised?.Invoke(value);
}
```

Publisher and listener both reference the asset — no scene references needed (UI ↔ gameplay ↔ audio). Unsubscribe in `OnDisable`. Don't overuse: debugging "who raised this?" gets harder.

## Runtime sets

An SO holding a `List<Enemy>` that enemies add/remove themselves to in `OnEnable/OnDisable` — replaces `FindObjectsByType`. Clear it on play start (domain reload may be disabled).

## Databases & lookups

A `ItemDatabase` SO with `List<ItemDefinition>` and a dictionary built in `OnEnable`, used to resolve IDs from save files. Prefer stable string/GUID IDs over list indices.

Loading: direct references (simplest), `Resources.Load` (avoid for large projects), or **Addressables** (`Addressables.LoadAssetAsync<WeaponDefinition>(key)`) for content streaming/DLC.

## Saving

SOs are **not** a save system. Serialize plain data (IDs + values) to JSON (`JsonUtility`, Newtonsoft) into `Application.persistentDataPath`. See `save-systems`.

## Pitfalls

- `OnEnable` on SOs runs when the asset loads (editor: on domain reload), not per scene.
- Unreferenced SOs are not included in builds unless in Resources/Addressables.
- Renaming serialized fields loses data: `[FormerlySerializedAs]`.
