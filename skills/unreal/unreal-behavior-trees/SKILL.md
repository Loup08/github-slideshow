---
name: unreal-behavior-trees
description: Unreal Engine 5 AI — AIController, Blackboard and Behavior Trees (composites, decorators with observer aborts, services, custom tasks), AI Perception (sight/hearing/damage), EQS queries for cover and positioning, NavMesh, and when to use StateTree instead. Use for enemy/NPC AI in Unreal.
---

# Unreal AI: Behavior Trees, EQS, StateTree

## Wiring

1. `BP_EnemyAIController` (AIController). Enemy Pawn: *AI Controller Class* = it, *Auto Possess AI* = *Placed in World or Spawned*.
2. Controller `OnPossess` → `Run Behavior Tree(BT_Enemy)` (it uses the BT's Blackboard asset).
3. Place a **NavMeshBoundsVolume** in the level (press P to visualize). Use *Runtime Generation = Dynamic* for procedural/movable geometry.

## Blackboard keys

`TargetActor` (Object/Actor), `LastKnownLocation` (Vector), `HasLineOfSight` (Bool), `PatrolIndex` (Int), `State` (Enum).

## Tree structure (typical)

```
Root
└── Selector
    ├── [Decorator: Blackboard TargetActor Is Set, Observer Aborts: Both]
    │   Sequence "Combat"
    │   ├── Service: UpdateLineOfSight (0.2s)
    │   ├── MoveTo TargetActor (Acceptable Radius 300)
    │   └── Task: Attack
    ├── [Decorator: LastKnownLocation Is Set, Aborts: Lower Priority]
    │   Sequence "Investigate"
    │   ├── MoveTo LastKnownLocation
    │   ├── Wait 3s
    │   └── Task: Clear LastKnownLocation
    └── Sequence "Patrol"
        ├── Task: GetNextPatrolPoint
        ├── MoveTo PatrolPoint
        └── Wait 1–2s (random deviation)
```

- **Selector** = first child that succeeds (priority list). **Sequence** = all in order until one fails. **Simple Parallel** = main task + background subtree.
- **Observer Aborts**: *Self* (abort this branch when condition becomes false), *Lower Priority* (interrupt lower branches when condition becomes true), *Both*. This is what makes the AI react instantly.
- **Services** tick on an interval while their branch is active — use for sensing/updating keys, not Tick.

## Perception

Add `AIPerceptionComponent` to the controller with *AI Sight* (radius, lose sight radius, peripheral angle, detection by affiliation — check Neutrals/Enemies or set team IDs via `IGenericTeamAgentInterface`), *AI Hearing* (`Report Noise Event`), *AI Damage*. Bind `OnTargetPerceptionUpdated`:

- Sensed & sight → set `TargetActor`.
- Lost → clear `TargetActor`, set `LastKnownLocation`.

Players need an `AIPerceptionStimuliSourceComponent` (or auto-register pawns setting).

## Custom task (C++)

```cpp
UCLASS()
class UBTTask_Attack : public UBTTaskNode
{
    GENERATED_BODY()
public:
    UBTTask_Attack() { NodeName = "Attack"; bNotifyTick = false; }
    virtual EBTNodeResult::Type ExecuteTask(UBehaviorTreeComponent& OwnerComp, uint8* NodeMemory) override
    {
        APawn* Pawn = OwnerComp.GetAIOwner()->GetPawn();
        if (auto* Combat = Pawn ? Pawn->FindComponentByClass<UCombatComponent>() : nullptr)
        {
            Combat->Attack();
            return EBTNodeResult::Succeeded;   // or InProgress + FinishLatentTask later
        }
        return EBTNodeResult::Failed;
    }
};
```

Blueprint tasks: `BTTask_BlueprintBase` → *Event Receive Execute AI* → must call *Finish Execute*. Forgetting it hangs the tree.

## EQS (Environment Query System)

Queries generate items (points on grid/circle, actors of class) and score them with tests (distance to target, trace visibility, pathfinding cost, dot product). Use for "find cover not visible to player within 1500 units", flanking positions, or best pickup. Run via *Run EQS Query* BT task writing to a Blackboard key. Debug with the `EQSTestingPawn`.

## StateTree vs Behavior Tree

**StateTree** (production-ready in UE5; used by Mass and Smart Objects) is a hierarchical state machine with selectors — easier for clearly state-based AI (Idle/Patrol/Combat/Flee), lower overhead, good for many agents. **Behavior Trees** remain strong for reactive priority-driven AI with complex blackboards. Choose one per AI type; don't nest both.

## Debugging

Press `'` (apostrophe) in PIE for the AI debugger (BT, perception, EQS, nav), Visual Logger (`vislog`), BT editor shows the active branch live when an instance is selected.

## Pitfalls

- AI doesn't move: no NavMesh, pawn not possessed, or `MoveTo` acceptance radius smaller than capsule.
- Tree keeps restarting: decorator aborts flickering — add cooldowns or hysteresis on keys.
- Heavy logic in Services every frame — raise interval.
