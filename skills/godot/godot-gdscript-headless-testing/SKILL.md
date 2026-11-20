---
name: godot-gdscript-headless-testing
description: Test Godot 4 GDScript headlessly — GUT and gdUnit4 setup, writing unit and scene tests, simulating input and frames, running from the command line, and CI with GitHub Actions. Use when adding automated tests or verifying GDScript changes without opening the editor.
---

# Headless GDScript Testing

## Option 1: zero-dependency smoke script

```gdscript
# tests/run_tests.gd   →   godot --headless --path . -s tests/run_tests.gd
extends SceneTree

func _initialize() -> void:
	var failures := 0
	failures += _test_damage()
	print("FAILURES: %d" % failures)
	quit(1 if failures > 0 else 0)

func _test_damage() -> int:
	var player: Node = load("res://entities/player/player.tscn").instantiate()
	root.add_child(player)
	player.take_damage(30)
	if player.health != 70:
		push_error("expected 70, got %d" % player.health)
		return 1
	player.free()
	return 0
```

Good for quick verification by an agent. Also `godot --headless --path . --check-only -s path/to/file.gd` for parse-checking a script, and `godot --headless --path . --quit-after 120` to boot the main scene for 120 frames and catch runtime errors.

## Option 2: GUT (Godot Unit Test)

Install `addons/gut` (AssetLib "GUT - Godot Unit Testing (Godot 4)"), enable plugin.

```gdscript
# test/unit/test_inventory.gd
extends GutTest

var inv: Inventory

func before_each() -> void:
	inv = Inventory.new()

func after_each() -> void:
	inv.free()   # or autofree(inv) / add_child_autofree(node)

func test_add_stacks() -> void:
	inv.add(&"potion", 3)
	inv.add(&"potion", 2)
	assert_eq(inv.count(&"potion"), 5)

func test_signal_emitted() -> void:
	watch_signals(inv)
	inv.add(&"coin", 1)
	assert_signal_emitted(inv, "changed")

func test_player_falls() -> void:
	var p = add_child_autofree(load("res://entities/player/player.tscn").instantiate())
	await wait_physics_frames(10)
	assert_gt(p.velocity.y, 0.0)
```

Run:

```bash
godot --headless --path . -s addons/gut/gut_cmdln.gd -gdir=res://test -ginclude_subdirs -gexit
```

Doubles/stubs: `double(MyClass)`, `stub(d, "method").to_return(5)`. Input simulation: `InputSender`.

## Option 3: gdUnit4

Install `addons/gdUnit4`. Tests extend `GdUnitTestSuite`, fluent asserts (`assert_int(x).is_equal(5)`), scene runner (`scene_runner("res://...")` with `simulate_key_pressed`, `simulate_frames`). Run with `addons/gdUnit4/runtest.sh -a res://test` (or `.cmd` on Windows). Has an official GitHub Action (`MikeSchulze/gdUnit4-action`).

## CI (GitHub Actions) sketch

```yaml
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Install Godot
        run: |
          wget -q https://github.com/godotengine/godot/releases/download/${GODOT_VERSION}-stable/Godot_v${GODOT_VERSION}-stable_linux.x86_64.zip
          unzip -q Godot_*.zip && mv Godot_*_linux.x86_64 /usr/local/bin/godot
      - run: godot --headless --path . --import
      - run: godot --headless --path . -s addons/gut/gut_cmdln.gd -gdir=res://test -gexit
    env:
      GODOT_VERSION: "4.x.y"   # pin to the project's version
```

## What to test

- Pure logic (inventory, damage formulas, save serialization, procedural gen with fixed seeds) — fast unit tests.
- Scene smoke tests: instantiate every level, run N frames, assert no errors.
- Avoid pixel-perfect render tests; headless uses a dummy renderer.

## Pitfalls

- Missing `--import` on fresh checkout → resources fail to load in CI.
- Leaked nodes produce "ObjectDB instances leaked" warnings — free nodes or use autofree helpers.
- `class_name` globals need the import/scan step to register.
