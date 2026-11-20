---
name: godot-signals-groups
description: Decouple Godot 4 code with signals, Callables, event-bus autoloads, and groups — declaring typed signals, connect flags, one-shot connections, broadcasting to groups. Use when nodes need to communicate or when wiring UI to gameplay.
---

# Godot Signals & Groups

## Declare, emit, connect

```gdscript
signal died
signal damaged(amount: int, source: Node)

func take_damage(amount: int, source: Node) -> void:
	health -= amount
	damaged.emit(amount, source)
	if health <= 0:
		died.emit()
```

```gdscript
# in parent / HUD
player.damaged.connect(_on_player_damaged)
player.died.connect(_on_player_died, CONNECT_ONE_SHOT)
enemy.died.connect(_on_enemy_died.bind(enemy))      # extra args appended
button.pressed.connect(func(): start_game())        # lambda
```

Disconnect with `sig.disconnect(callable)`; check with `sig.is_connected(callable)`. Connections to a freed object are cleaned up automatically.

Connect flags: `CONNECT_ONE_SHOT`, `CONNECT_DEFERRED` (call at idle time — useful when the handler modifies the tree or physics state).

## Editor vs code connections

Editor connections (Node dock) are fine for UI inside one scene. Prefer code connections across scenes so the wiring is greppable.

## Event bus (global signals)

```gdscript
# autoload/events.gd  (Autoload name: Events)
extends Node
signal coin_collected(value: int)
signal level_completed(level_id: StringName)
```

```gdscript
Events.coin_collected.emit(5)                 # anywhere
Events.coin_collected.connect(_on_coin)       # HUD
```

Use the bus for cross-cutting events (score, achievements, audio cues). Don't route everything through it — direct signals are easier to trace.

## Groups

```gdscript
add_to_group("enemies")                 # or Node dock → Groups (persistent)
get_tree().get_nodes_in_group("enemies")
get_tree().call_group("enemies", "freeze", 2.0)
if body.is_in_group("player"): ...
```

Use groups for tagging (`"damageable"`, `"interactable"`) and broadcasts. For type checks with methods, prefer `if body is Damageable` / `has_method("take_damage")`.

## Await signals

```gdscript
await player.died
var result = await dialog.choice_made   # signal args returned (array if >1)
```

## Pitfalls

- Signal name typos are runtime errors with string API (`emit_signal("dide")`); always use the signal property (`died.emit()`).
- Circular emits (A→B→A) cause stack overflows; use `CONNECT_DEFERRED` or redesign.
- Emitting before listeners connect (e.g. in `_ready` of a child) is lost — connect in parent `_ready` before children emit, or emit deferred.
