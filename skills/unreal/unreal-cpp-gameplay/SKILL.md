---
name: unreal-cpp-gameplay
description: Unreal Engine 5 C++ gameplay programming — UCLASS/UPROPERTY/UFUNCTION reflection, Actor and Component classes, BlueprintCallable/ImplementableEvent bridges, delegates, timers, GC and pointer types (TObjectPtr, TWeakObjectPtr, TSoftObjectPtr), subsystems, logging, replication basics, and build/module setup. Use when writing or debugging UE C++.
---

# Unreal C++ Gameplay

## Actor skeleton

```cpp
// HealthComponent.h
#pragma once
#include "CoreMinimal.h"
#include "Components/ActorComponent.h"
#include "HealthComponent.generated.h"   // must be the last include

DECLARE_DYNAMIC_MULTICAST_DELEGATE_TwoParams(FOnHealthChanged, float, NewHealth, float, Delta);
DECLARE_DYNAMIC_MULTICAST_DELEGATE(FOnDied);

UCLASS(ClassGroup=(Gameplay), meta=(BlueprintSpawnableComponent))
class MYGAME_API UHealthComponent : public UActorComponent
{
    GENERATED_BODY()
public:
    UHealthComponent();

    UPROPERTY(BlueprintAssignable) FOnHealthChanged OnHealthChanged;
    UPROPERTY(BlueprintAssignable) FOnDied OnDied;

    UFUNCTION(BlueprintCallable, Category="Health")
    void ApplyDamage(float Amount);

    UFUNCTION(BlueprintPure, Category="Health")
    float GetHealth() const { return Health; }

protected:
    virtual void BeginPlay() override;

    UPROPERTY(EditDefaultsOnly, BlueprintReadOnly, Category="Health", meta=(ClampMin="1"))
    float MaxHealth = 100.f;

private:
    UPROPERTY(VisibleInstanceOnly, Category="Health")
    float Health = 0.f;
};
```

```cpp
// HealthComponent.cpp
#include "HealthComponent.h"

UHealthComponent::UHealthComponent() { PrimaryComponentTick.bCanEverTick = false; }

void UHealthComponent::BeginPlay()
{
    Super::BeginPlay();
    Health = MaxHealth;
}

void UHealthComponent::ApplyDamage(float Amount)
{
    if (Amount <= 0.f || Health <= 0.f) return;
    const float Old = Health;
    Health = FMath::Clamp(Health - Amount, 0.f, MaxHealth);
    OnHealthChanged.Broadcast(Health, Health - Old);
    if (Health <= 0.f) OnDied.Broadcast();
}
```

## Specifier cheat sheet

- `EditAnywhere` / `EditDefaultsOnly` (class defaults only) / `EditInstanceOnly`; `VisibleAnywhere` for component pointers.
- `BlueprintReadOnly` / `BlueprintReadWrite`; `Category="..."` required for BP-exposed members.
- `UFUNCTION(BlueprintCallable)`, `BlueprintPure`, `BlueprintImplementableEvent` (BP implements), `BlueprintNativeEvent` (C++ default `_Implementation`, BP may override).
- `Replicated`, `ReplicatedUsing=OnRep_X`; `UFUNCTION(Server, Reliable)`.

## Pointers & GC

- `UPROPERTY() TObjectPtr<UFoo> Foo;` — strong ref visible to GC (always `UPROPERTY` UObject pointers, or they can be collected → dangling).
- `TWeakObjectPtr<AActor>` — non-owning, check `IsValid()`.
- `TSoftObjectPtr` / `TSoftClassPtr` — asset path, load on demand (`LoadSynchronous()` or `FStreamableManager` async).
- `IsValid(Obj)` checks null and pending-kill.
- Non-UObject data: `TUniquePtr`, `TSharedPtr`.

## Components in constructor

```cpp
AMyCharacter::AMyCharacter()
{
    Health = CreateDefaultSubobject<UHealthComponent>(TEXT("Health"));
    SpringArm = CreateDefaultSubobject<USpringArmComponent>(TEXT("SpringArm"));
    SpringArm->SetupAttachment(RootComponent);
}
```

Never `NewObject`/`SpawnActor` in constructors; use `BeginPlay`.

## Timers, delegates, spawning

```cpp
GetWorldTimerManager().SetTimer(RegenHandle, this, &UThis::Regen, 1.f, true);
Health->OnDied.AddDynamic(this, &AMyCharacter::HandleDied);   // UFUNCTION required
FActorSpawnParameters P; P.Owner = this; P.Instigator = GetInstigator();
AProjectile* Proj = GetWorld()->SpawnActor<AProjectile>(ProjectileClass, Xf, P);
```

`ProjectileClass` is `UPROPERTY(EditDefaultsOnly) TSubclassOf<AProjectile>` — set to a Blueprint child.

## Subsystems

`UGameInstanceSubsystem`, `UWorldSubsystem`, `ULocalPlayerSubsystem` — auto-instanced singletons with proper lifetime: `GetGameInstance()->GetSubsystem<UQuestSubsystem>()`.

## Logging & asserts

```cpp
DEFINE_LOG_CATEGORY_STATIC(LogMyGame, Log, All);
UE_LOG(LogMyGame, Warning, TEXT("Hit %s for %.1f"), *Other->GetName(), Damage);
check(Ptr); ensureMsgf(Value > 0, TEXT("bad value"));
```

## Build

- Module dependencies in `MyGame.Build.cs`: `PublicDependencyModuleNames.AddRange(new[] { "Core", "CoreUObject", "Engine", "InputCore", "EnhancedInput", "AIModule", "GameplayTasks", "UMG" });`
- Live Coding (Ctrl+Alt+F11) for .cpp changes; **header/reflection changes need an editor restart + full build**.
- Regenerate project files after adding files outside the editor.

## Pitfalls

- Missing `Super::BeginPlay()` / `Super::Tick()` calls.
- Raw `UObject*` members without `UPROPERTY` get GC'd.
- Hot Reload is deprecated — use Live Coding, and close the editor for structural changes.
- Hard references in `ConstructorHelpers::FObjectFinder` load assets always; prefer `TSoftObjectPtr`/BP-assigned classes.
