---
name: godot-csharp
description: Write Godot 4 games in C# (.NET 8+) — partial classes, [Export], signals as C# events, node access, GDScript interop, async with ToSignal, collections marshalling, and platform limits. Use when a Godot project has a .csproj or the user wants C# in Godot.
---

# Godot C#

Requires the **.NET edition** of the editor and the .NET SDK. Scripts must be `partial` and the class name must match the file name.

## Skeleton

```csharp
using Godot;

public partial class Player : CharacterBody2D
{
    [Signal] public delegate void HealthChangedEventHandler(int newValue);

    [Export] public float Speed { get; set; } = 200f;
    [Export] public PackedScene BulletScene { get; set; }

    private AnimatedSprite2D _sprite;
    private int _health = 100;

    public override void _Ready()
    {
        _sprite = GetNode<AnimatedSprite2D>("AnimatedSprite2D");
        GetNode<Area2D>("%Hurtbox").AreaEntered += OnHurtboxAreaEntered;
    }

    public override void _PhysicsProcess(double delta)
    {
        Vector2 input = Input.GetVector("move_left", "move_right", "move_up", "move_down");
        Velocity = input * Speed;
        MoveAndSlide();
    }

    private void OnHurtboxAreaEntered(Area2D area)
    {
        _health -= 10;
        EmitSignal(SignalName.HealthChanged, _health);
    }
}
```

- Signal delegates must end in `EventHandler`; the source generator creates the `HealthChanged` event and `SignalName.HealthChanged`.
- Subscribe with `+=`; unsubscribe in `_ExitTree` when the publisher outlives the subscriber (lambdas can't be unsubscribed — keep a method reference).
- `delta` is `double`; cast to float for math with `Vector2`.
- Use `SignalName`, `MethodName`, `PropertyName` generated constants instead of strings.

## Async

```csharp
await ToSignal(GetTree().CreateTimer(0.5), SceneTreeTimer.SignalName.Timeout);
await ToSignal(animPlayer, AnimationPlayer.SignalName.AnimationFinished);
```

## Collections

`Godot.Collections.Array<T>` / `Dictionary<TKey,TValue>` marshal to the engine (needed for `[Export]` and passing to GDScript). Use `System.Collections.Generic` internally — faster, no marshalling.

## Interop with GDScript

```csharp
var gd = GetNode("SomeGdNode");
gd.Call("take_damage", 5);
int hp = gd.Get("health").AsInt32();
gd.Connect("died", Callable.From(OnDied));
```

GDScript calls C# methods by their C# name (PascalCase).

## Performance notes

- Avoid allocations in `_Process` (LINQ, closures, `GetNode` lookups) — cache nodes in `_Ready`.
- `Variant` boxing on engine calls: prefer typed APIs.
- Strings like `"move_left"` → cache as `StringName` static readonly to avoid conversions per frame.

## Platform support

Desktop: full. Android/iOS: supported in Godot 4.2+ (experimental at first; check current status). **Web export with C# is not supported** in Godot 4 at the time of writing — use GDScript for web targets.

## Tooling

Use Rider or VS Code (C# Dev Kit) as external editor (Editor Settings → Dotnet). `dotnet build` works from CLI; unit-test pure C# logic with xUnit/NUnit in a separate project, and engine-dependent code with GdUnit4Net.
