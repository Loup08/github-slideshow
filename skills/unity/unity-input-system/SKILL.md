---
name: unity-input-system
description: Use Unity's Input System package — Input Action assets, action maps, PlayerInput vs generated C# class, reading values, rebinding with persistence, local multiplayer, and UI integration. Use for player controls, gamepad support, and key rebinding in Unity.
---

# Unity Input System

Unity 6 projects use the Input System by default (Player Settings → Active Input Handling = *Input System Package (New)*). The project-wide actions asset lives in Project Settings → Input System Package.

## Actions asset

Create `Controls.inputactions`:

- Action map **Gameplay**: `Move` (Value, Vector2; WASD composite + Left Stick), `Look` (Value, Vector2; Mouse delta + Right Stick), `Jump` (Button), `Fire` (Button), `Interact` (Button, Hold interaction for 0.4 s).
- Action map **UI**: Navigate, Submit, Cancel, Point, Click.
- Control schemes: Keyboard&Mouse, Gamepad.
- Tick **Generate C# Class**.

## Reading input (generated class)

```csharp
public sealed class PlayerController : MonoBehaviour
{
    private Controls controls;
    private Vector2 move;

    private void Awake()
    {
        controls = new Controls();
        controls.Gameplay.Jump.performed += _ => jumpBuffered = true;
    }
    private void OnEnable() => controls.Gameplay.Enable();
    private void OnDisable() => controls.Gameplay.Disable();
    private void OnDestroy() => controls.Dispose();

    private void Update()
    {
        move = controls.Gameplay.Move.ReadValue<Vector2>();
        if (controls.Gameplay.Fire.IsPressed()) TryFire();
        if (controls.Gameplay.Interact.WasPerformedThisFrame()) Interact();
    }
}
```

Callbacks: `started`, `performed`, `canceled`. Polling: `ReadValue<T>()`, `WasPressedThisFrame()`, `WasReleasedThisFrame()`, `IsPressed()`.

## PlayerInput component (alternative)

Add `PlayerInput`, assign the asset, Behavior = *Invoke Unity Events* or *Send Messages* (`void OnMove(InputValue v)`). Required for **local multiplayer** with `PlayerInputManager` (join on button press, auto device assignment, split-screen).

## Switching maps

```csharp
controls.Gameplay.Disable();
controls.UI.Enable();    // when opening a menu
```

With `PlayerInput`: `playerInput.SwitchCurrentActionMap("UI")`.

## Rebinding

```csharp
action.Disable();
action.PerformInteractiveRebinding(bindingIndex)
    .WithControlsExcluding("<Mouse>/position")
    .WithCancelingThrough("<Keyboard>/escape")
    .OnComplete(op => { op.Dispose(); action.Enable(); Save(); })
    .Start();

void Save() => PlayerPrefs.SetString("binds", controls.asset.SaveBindingOverridesAsJson());
void Load() => controls.asset.LoadBindingOverridesFromJson(PlayerPrefs.GetString("binds"));
```

Display: `action.GetBindingDisplayString(bindingIndex)`. The *Rebinding UI* sample in Package Manager is a good starting point.

## UI

Replace `StandaloneInputModule` with `InputSystemUIInputModule` on the EventSystem. Set `EventSystem.current.SetSelectedGameObject(firstButton)` when menus open for gamepad navigation.

## Pitfalls

- Reading `Look` mouse delta: it's already per-frame — **don't** multiply by `Time.deltaTime`; do multiply stick input by deltaTime.
- Forgetting `Enable()` → nothing fires.
- Old `Input.GetAxis` calls throw when only the new system is active.
- Deadzones: set via processors (`StickDeadzone`) on the binding, not in code.
