---
name: unity-animation
description: Unity animation with Animator Controllers (state machines, blend trees, layers, avatar masks), parameters and hashed IDs, animation events, root motion, 2D sprite animation, and code-driven tweening. Use for character animation, attack timing, or UI motion in Unity.
---

# Unity Animation

## Animator Controller essentials

- **Parameters**: `Speed` (float), `Grounded` (bool), `Attack` (trigger), `VelocityY` (float).
- **States** linked by transitions with conditions. For responsive gameplay: uncheck *Has Exit Time* on input-driven transitions, transition duration 0–0.1 s.
- **Blend Tree** for locomotion (1D on Speed, or 2D Freeform Directional on MoveX/MoveY for strafing).
- **Layers** + **Avatar Mask** to play upper-body attacks while legs run (layer weight 1, blending Override).
- **Any State** transitions for hit/death — add a condition guard to avoid re-triggering every frame.
- Sub-state machines to group (Combat, Locomotion).

## Driving from code

```csharp
private static readonly int SpeedId = Animator.StringToHash("Speed");
private static readonly int GroundedId = Animator.StringToHash("Grounded");
private static readonly int AttackId = Animator.StringToHash("Attack");

void Update()
{
    animator.SetFloat(SpeedId, horizontalSpeed, 0.1f, Time.deltaTime); // damped
    animator.SetBool(GroundedId, grounded);
    if (attackPressed) animator.SetTrigger(AttackId);
}
```

Reset stale triggers (`ResetTrigger`) when an action is cancelled, or they fire later unexpectedly.

## Animation events & state behaviours

- Animation Events on clips call a public method on a component on the same GameObject (e.g. `EnableHitbox()`, `Footstep()`). Great for frame-accurate hits and sounds.
- `StateMachineBehaviour` scripts on states: `OnStateEnter/Exit/Update` for state-scoped logic (lock movement during attack).

## Root motion

`Apply Root Motion` on the Animator lets clips move the character. Override with `OnAnimatorMove()`:

```csharp
void OnAnimatorMove() => controller.Move(animator.deltaPosition);
```

Use for melee lunges, realistic locomotion; avoid for twitchy platformers.

## 2D sprite animation

Slice sprite sheets in the Sprite Editor, drag frames into the scene to create a clip (set sample rate, e.g. 12 fps). For flipping, set `SpriteRenderer.flipX` from code rather than duplicating clips. 2D Animation package supports bone rigs (PSD Importer + Sprite Skin).

## Retargeting & IK

Humanoid rigs (Avatar) allow sharing clips across characters. Foot/hand IK via `OnAnimatorIK` (enable IK Pass on the layer) or the Animation Rigging package (Two Bone IK, Multi-Aim constraints).

## Tweening

For UI/juice use a tween library (DOTween, PrimeTween, LitMotion) or Unity 6 `Awaitable` with manual lerp. Avoid Animator on UI elements that change every frame (dirtying Canvas each frame).

## Timeline

Cutscenes: Timeline with Animation, Activation, Audio, Signal tracks; Cinemachine track for cameras.

## Pitfalls

- An Animator writing a property each frame overrides your script (e.g. scale, color). Remove the property from clips or disable *Write Defaults* consistently across the controller.
- Culling Mode `Cull Completely` stops off-screen animators — events don't fire.
- Many Animators are CPU-heavy; for crowds, use simpler Animation component, GPU animation, or disable when far.
