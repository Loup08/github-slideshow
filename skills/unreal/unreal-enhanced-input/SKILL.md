---
name: unreal-enhanced-input
description: Unreal Engine 5 Enhanced Input — Input Actions and value types, Input Mapping Contexts with priorities, modifiers (dead zone, negate, swizzle) and triggers (hold, tap, chorded), binding in C++ and Blueprint, context switching for vehicles/menus, and player-mappable keys. Use for any input/controls work in UE5.
---

# Unreal Enhanced Input

Enhanced Input is the default input system in UE5 (legacy Action/Axis mappings are deprecated).

## Assets

- **Input Action** (`IA_Move`): Value Type `Axis2D`; `IA_Look`: Axis2D; `IA_Jump`: Digital (bool); `IA_Interact`: Digital with *Hold* trigger (0.5 s).
- **Input Mapping Context** (`IMC_Default`): map keys to actions.
  - WASD → `IA_Move`: W = no modifier + *Swizzle Input Axis Values* (YXZ); S = Swizzle + *Negate*; A = *Negate*; D = none.
  - Gamepad Left Thumbstick 2D-Axis → `IA_Move` with *Dead Zone* modifier.
  - Mouse XY 2D-Axis → `IA_Look` (+ *Negate* Y if needed).
- Add `IMC_Vehicle`, `IMC_Menu` contexts for other modes.

## C++ binding

```cpp
// Character.h
UPROPERTY(EditDefaultsOnly, Category="Input") TObjectPtr<UInputMappingContext> DefaultContext;
UPROPERTY(EditDefaultsOnly, Category="Input") TObjectPtr<UInputAction> MoveAction;
UPROPERTY(EditDefaultsOnly, Category="Input") TObjectPtr<UInputAction> LookAction;
UPROPERTY(EditDefaultsOnly, Category="Input") TObjectPtr<UInputAction> JumpAction;

// Character.cpp
#include "EnhancedInputComponent.h"
#include "EnhancedInputSubsystems.h"

void AMyCharacter::NotifyControllerChanged()
{
    Super::NotifyControllerChanged();
    if (const APlayerController* PC = Cast<APlayerController>(Controller))
        if (auto* Subsystem = ULocalPlayer::GetSubsystem<UEnhancedInputLocalPlayerSubsystem>(PC->GetLocalPlayer()))
            Subsystem->AddMappingContext(DefaultContext, /*Priority*/0);
}

void AMyCharacter::SetupPlayerInputComponent(UInputComponent* PlayerInputComponent)
{
    auto* EIC = CastChecked<UEnhancedInputComponent>(PlayerInputComponent);
    EIC->BindAction(MoveAction, ETriggerEvent::Triggered, this, &AMyCharacter::Move);
    EIC->BindAction(LookAction, ETriggerEvent::Triggered, this, &AMyCharacter::Look);
    EIC->BindAction(JumpAction, ETriggerEvent::Started, this, &ACharacter::Jump);
    EIC->BindAction(JumpAction, ETriggerEvent::Completed, this, &ACharacter::StopJumping);
}

void AMyCharacter::Move(const FInputActionValue& Value)
{
    const FVector2D Axis = Value.Get<FVector2D>();
    const FRotator Yaw(0, Controller->GetControlRotation().Yaw, 0);
    AddMovementInput(FRotationMatrix(Yaw).GetUnitAxis(EAxis::X), Axis.Y);
    AddMovementInput(FRotationMatrix(Yaw).GetUnitAxis(EAxis::Y), Axis.X);
}

void AMyCharacter::Look(const FInputActionValue& Value)
{
    const FVector2D Axis = Value.Get<FVector2D>();
    AddControllerYawInput(Axis.X);
    AddControllerPitchInput(Axis.Y);
}
```

Add `"EnhancedInput"` to Build.cs dependencies.

## Blueprint

PlayerController/Character `BeginPlay` → *Get Enhanced Input Local Player Subsystem* → *Add Mapping Context*. Then right-click → `EnhancedInputAction IA_Move` event node; use *Triggered* for continuous, *Started* for press, *Completed* for release.

## Trigger events

`Started` (first frame) → `Ongoing` (e.g. while holding before Hold threshold) → `Triggered` (condition met; every frame for continuous) → `Completed` / `Canceled` (released before threshold).

Triggers: *Pressed*, *Released*, *Hold*, *Hold And Release*, *Tap*, *Pulse* (repeat fire), *Chorded Action* (Shift+Key combos), *Combo*.

## Context switching

```cpp
Subsystem->RemoveMappingContext(IMC_OnFoot);
Subsystem->AddMappingContext(IMC_Vehicle, 1);
```

Higher priority contexts consume keys first. For menus, also `PC->SetInputMode(FInputModeUIOnly())` and show the cursor.

## Rebinding

Enable *Player Mappable* on mappings (UE 5.3+: *Player Mappable Key Settings* on the action/mapping), and use `UEnhancedInputUserSettings` to remap keys and save profiles. Display key names via `UEnhancedInputLocalPlayerSubsystem::QueryKeysMappedToAction`.

## Pitfalls

- Forgetting to add the mapping context (no input at all) — often because the pawn wasn't possessed yet in `BeginPlay`; `NotifyControllerChanged`/`PawnClientRestart` is safer.
- Mouse look multiplied by DeltaTime → framerate-dependent sensitivity. Don't.
- Using `Triggered` for a jump causes repeat calls while held — use `Started`.
