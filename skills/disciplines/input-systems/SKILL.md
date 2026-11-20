---
name: input-systems
description: Design input handling for any engine — action mapping instead of raw keys, keyboard/mouse/gamepad/touch support, input buffering, coyote time, dead zones and response curves, rebinding and glyph switching, input contexts (gameplay vs UI), local multiplayer device assignment, accessibility (hold vs toggle, remapping), and replay/determinism. Use when implementing controls or fixing unresponsive or inconsistent input.
---

# Input Systems

## Map actions, not keys

```
Action "jump": [Key.Space, Gamepad.South, Touch.ButtonJump]
Action "move": [WASD composite, Arrows composite, Gamepad.LeftStick]
```

Gameplay code asks `input.pressed("jump")`, never `key == SPACE`. Every engine has this: Godot InputMap, Unity Input System actions, Unreal Enhanced Input, Phaser key objects + your own mapping, Roblox ContextActionService.

## Input states per frame

- `down` (held), `pressed` (went down this frame), `released` (went up this frame).
- Sample once per frame/tick into a snapshot; logic reads the snapshot (consistency + replays).
- In fixed-step loops, latch `pressed` events until the next fixed step consumes them — otherwise presses between ticks get lost on high frame rates.

## Responsiveness tricks

- **Input buffering**: remember a press for 100–150 ms; execute when the action becomes valid (jump just before landing, attack during recovery).
- **Coyote time**: allow jump 80–120 ms after leaving a ledge.
- **Cancel windows**: allow dodge to cancel attack recovery after N frames.
- Act on press (not release) for core actions; release for charge attacks.
- Minimize latency: read input as late as possible before simulation; avoid extra frames of smoothing.

## Analog sticks

```python
def apply_radial_deadzone(v, dead=0.2, outer=0.95):
    mag = v.length()
    if mag < dead: return Vector2()
    scaled = (min(mag, outer) - dead) / (outer - dead)
    return v.normalized() * scaled           # rescale so movement starts at 0, not 0.2
```

- **Radial** dead zone for movement (axial dead zones snap to axes).
- Response curve (`scaled ** 1.5–2`) for aiming precision.
- Aim assist (slowdown near targets, slight magnetism) for gamepad shooters.
- Normalize keyboard diagonals so diagonal speed isn't √2 faster.

## Mouse

Raw delta for camera look (no `deltaTime` scaling), sensitivity setting, invert Y option, pointer lock/capture with a clear way out (Esc).

## Touch

On-screen virtual joystick (floating: spawns where the thumb lands), large buttons (≥ 44–48 pt), context-sensitive actions, gestures (swipe/tap) for simple games. Keep the thumb zones (bottom corners) for primary controls.

## Contexts

Separate maps for Gameplay, UI/Menu, Vehicle, Dialogue. Opening a menu switches context so "jump" doesn't fire while navigating. UI should consume input before gameplay (Godot `_unhandled_input`, Unity EventSystem).

## Glyphs

Detect the last used device and swap button prompts (keyboard ↔ Xbox ↔ PlayStation ↔ Switch). Show the bound key, not a hardcoded one (respect rebinding). Kenney's input prompts pack is CC0.

## Rebinding

Required for PC (and accessibility). Store overrides separately from defaults, detect conflicts, allow reset to defaults, persist in settings file.

## Local multiplayer

Assign devices on "press any button to join"; keyboard can be split (WASD vs arrows); handle disconnect/reconnect (pause and prompt).

## Accessibility

- Toggle vs hold options for sprint/aim/crouch.
- Remap every action, including menus.
- Avoid rapid button mashing requirements or offer alternatives.
- Adjustable stick sensitivity and dead zones; one-handed layouts if feasible.

## Determinism & replays

Record the per-tick input snapshots + RNG seed → replays, ghost runs, bug reproduction, and lockstep netcode for free (if simulation is deterministic).
