---
name: save-systems
description: Design robust save/load systems for any engine — what to save (state vs definitions), stable IDs, serialization formats (JSON, binary, MessagePack), versioning and migrations, atomic writes and backups, autosave/checkpoints, multiple slots, cloud saves (Steam Cloud), settings vs progress, and anti-tamper considerations. Use whenever a game needs persistence.
---

# Save Systems

## What to save

Save **state**, not definitions:

- ✅ Player position, stats, inventory as `[{id: "sword_iron", qty: 1}]`, quest stages, world flags, opened chests by ID, RNG seed + step, play time.
- ❌ Whole objects, textures, item definitions, scene trees, engine references.

Every saveable entity needs a **stable ID** (authored GUID or deterministic `level_id:object_name`) — never array indices or runtime instance IDs.

## Architecture

```
SaveData (plain data, versioned)
  ↑ capture()        ↓ restore()
ISaveable components (player, inventory, chests, quests…)
  ↑                  ↓
SaveManager: collect → serialize → write atomically → backup
```

```python
class SaveManager:
    VERSION = 3

    def save(self, slot):
        data = {"version": self.VERSION, "timestamp": now(), "play_time": self.play_time,
                "systems": {s.save_key: s.capture() for s in self.saveables}}
        write_atomic(slot_path(slot), serialize(data))

    def load(self, slot):
        data = deserialize(read(slot_path(slot)))
        data = migrate(data)                       # see below
        for s in self.saveables:
            if s.save_key in data["systems"]:
                s.restore(data["systems"][s.save_key])
```

Restore order matters (world before entities that reference it). Load the scene/level first, then restore state.

## Versioning & migration

```python
def migrate(data):
    v = data.get("version", 1)
    if v < 2:
        data["systems"]["player"]["max_hp"] = 100        # new field default
        v = 2
    if v < 3:
        inv = data["systems"]["inventory"]
        inv["items"] = [{"id": i, "qty": 1} for i in inv.pop("item_ids")]   # shape change
        v = 3
    data["version"] = v
    return data
```

Never delete migration steps; keep test fixtures of every old version's save and test loading them.

## Atomic writes & backups

Corrupted saves from crashes/power loss are common. Write to `save.tmp`, flush/fsync, then rename over `save.dat` (atomic on most filesystems). Keep `save.bak` (previous good save) and fall back to it if parsing fails. Add a checksum to detect truncation.

## Formats

- **JSON**: debuggable, diff-able, easy migrations. Default choice.
- Binary (MessagePack, protobuf, engine binary serializers): smaller/faster for huge worlds.
- Compress big saves (gzip/zstd).
- Avoid formats that can execute code on load (Godot `.tres` with scripts, Python pickle, .NET BinaryFormatter) — dangerous for shared saves.

## Locations

Use the platform user-data path: Godot `user://`, Unity `Application.persistentDataPath`, Unreal `SaveGame` slots (Saved/SaveGames), LÖVE `love.filesystem`, browser `localStorage`/IndexedDB (size-limited, can be cleared). Never write next to the executable (often read-only).

## When to save

- **Autosave** at checkpoints, level transitions, after important events, and on quit — with a visible indicator ("Saving…" icon; don't quit during it on consoles).
- Manual slots: show thumbnail, location, play time, date.
- Roguelikes: save on run start and between rooms; "save & quit" suspend saves deleted on load to prevent save-scumming (if desired).
- Avoid saving mid-physics/mid-cutscene; snap to a consistent state.

## Settings ≠ progress

Settings (volume, keybinds, graphics, language, accessibility) go in a separate file, loaded before the main menu, shared across slots.

## Cloud saves

Steam Auto-Cloud: configure paths in Steamworks (no code); keep files small and few. Handle conflicts (show both timestamps). Consoles have their own APIs and size limits.

## Tamper resistance

Single-player: don't bother beyond checksums — players modding their own saves is fine. Online/competitive economies must be server-authoritative; encrypting local saves won't stop determined users.

## Test checklist

- [ ] Save → quit → load restores identical state (automated round-trip test)
- [ ] Load old-version fixtures
- [ ] Kill the process during save → game recovers from backup
- [ ] Missing/corrupt file → graceful new game / error message, never crash
- [ ] Disk full / permission denied handled
