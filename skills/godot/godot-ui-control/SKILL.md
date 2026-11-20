---
name: godot-ui-control
description: Build Godot 4 UI with Control nodes — anchors and containers, themes, responsive scaling, focus/gamepad navigation, HUD over gameplay with CanvasLayer, menus, pause, and tweened transitions. Use for HUDs, menus, inventories, dialogs, and settings screens.
---

# Godot UI (Control)

## Layout: containers over manual positions

- Root UI under a `CanvasLayer` (layer 10 for HUD, 100 for pause/menus) so it ignores the game camera.
- Root `Control` → *Layout → Full Rect* anchors.
- Use containers: `MarginContainer` (padding), `VBoxContainer`/`HBoxContainer`, `GridContainer`, `CenterContainer`, `PanelContainer`, `ScrollContainer`, `AspectRatioContainer`.
- Children of containers ignore their own position; control them via *Size Flags* (`Fill`, `Expand`, `Shrink Center`) and `custom_minimum_size`.

## Responsive scaling

Project Settings → Display → Window → Stretch: Mode `canvas_items`, Aspect `expand`, base size e.g. 1920×1080 (or 320×180 for pixel art with Mode `viewport` and integer scaling). Anchor HUD elements to corners so they hug edges on ultrawide.

## Themes

Create a `Theme` resource, assign it to the root Control; all children inherit. Define default font + size, `StyleBoxFlat` for Button normal/hover/pressed/focus, Panel. Use **theme type variations** (e.g. `HeaderLabel` based on `Label`) instead of per-node overrides.

## HUD binding

```gdscript
extends CanvasLayer
@onready var hp_bar: ProgressBar = %HPBar
@onready var score_label: Label = %Score

func _ready() -> void:
	Events.score_changed.connect(func(s: int): score_label.text = str(s))

func set_health(value: int, max_value: int) -> void:
	hp_bar.max_value = max_value
	create_tween().tween_property(hp_bar, "value", value, 0.2)
```

## Focus & gamepad navigation

- Call `first_button.grab_focus()` when a menu opens — otherwise gamepad/keyboard does nothing.
- Set `focus_neighbor_*` when automatic neighbor detection picks wrong.
- Make the focus StyleBox clearly visible.
- `ui_accept`, `ui_cancel`, `ui_up`… are built-in actions; add gamepad buttons to them.

## Pause menu

```gdscript
extends Control   # process_mode = PROCESS_MODE_ALWAYS (Inspector → Process → Mode)

func _unhandled_input(event: InputEvent) -> void:
	if event.is_action_pressed("pause"):
		toggle()
		get_viewport().set_input_as_handled()

func toggle() -> void:
	get_tree().paused = not get_tree().paused
	visible = get_tree().paused
	if visible: %ResumeButton.grab_focus()
```

## Mouse filtering

`mouse_filter`: `STOP` (eats clicks), `PASS` (handles + passes up), `IGNORE`. Full-screen decorative Controls must be `IGNORE` or they block gameplay clicks. Gameplay input in `_unhandled_input` automatically respects UI consumption.

## Text

`Label` for plain text, `RichTextLabel` with `bbcode_enabled` for inline color/icons/typewriter (`visible_ratio` tween). Use `tr("KEY")` and CSV/PO translations from the start if localization is likely.

## Pitfalls

- UI drawn by a `Camera2D`-affected node moves with the camera — always use a CanvasLayer.
- Setting `size` on a container child is overwritten next layout pass.
- Long lists: `ItemList`/`Tree` or recycle rows; thousands of child Controls get slow.
