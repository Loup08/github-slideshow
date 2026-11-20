---
name: roblox-luau
description: Write idiomatic, typed Luau for Roblox — script types (Script, LocalScript, ModuleScript) and RunContext, where scripts live, services via GetService, --!strict type annotations, OOP with metatables, task library, signals and connections cleanup, and common performance pitfalls. Use for any Roblox scripting.
---

# Roblox Luau

## Where code lives

| Container | Runs on | Use |
|---|---|---|
| `ServerScriptService` | Server (`Script`) | Game logic, data, anti-cheat |
| `ServerStorage` | Server only (not replicated) | Server modules, templates |
| `ReplicatedStorage` | Both (replicated) | Shared modules, RemoteEvents, shared assets |
| `StarterPlayerScripts` | Client (`LocalScript`) | Input, camera, client controllers |
| `StarterCharacterScripts` | Client, per character spawn | Character-local behavior |
| `StarterGui` | Client | UI (copied into PlayerGui) |

Prefer a few entry-point scripts that `require` ModuleScripts ("single-script architecture"). `Script.RunContext = Client/Server` lets a Script run anywhere it's placed.

## Typed module

```lua
--!strict
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")

export type Weapon = {
	name: string,
	damage: number,
	cooldown: number,
	_lastFire: number,
	fire: (self: Weapon, target: Humanoid) -> boolean,
}

local Weapon = {}
Weapon.__index = Weapon

function Weapon.new(name: string, damage: number, cooldown: number): Weapon
	local self = setmetatable({}, Weapon) :: any
	self.name = name
	self.damage = damage
	self.cooldown = cooldown
	self._lastFire = 0
	return self
end

function Weapon.fire(self: Weapon, target: Humanoid): boolean
	local now = os.clock()
	if now - self._lastFire < self.cooldown then return false end
	self._lastFire = now
	target:TakeDamage(self.damage)
	return true
end

return Weapon
```

Use `--!strict` in new code; `--!nonstrict` for legacy. Type casts with `::`. Generics and `typeof()` supported.

## task library (not legacy wait/spawn/delay)

```lua
task.wait(0.5)                 -- yields; returns actual elapsed
task.spawn(fn, ...)            -- run now in new thread
task.defer(fn)                 -- run at end of current resumption cycle
task.delay(2, fn)
local dt = RunService.Heartbeat:Wait()
```

## Events & cleanup

```lua
local conn = part.Touched:Connect(function(hit: BasePart)
	local humanoid = hit.Parent and hit.Parent:FindFirstChildOfClass("Humanoid")
	if humanoid then humanoid:TakeDamage(10) end
end)
conn:Disconnect()
part.Destroying:Once(function() print("bye") end)
```

Destroying an Instance disconnects its own events, but connections to *other* objects held in closures leak. Use a cleanup helper (Trove/Maid/Janitor) per object lifetime.

Custom signals: `BindableEvent` or a pure-Luau Signal module (GoodSignal) for performance.

## Waiting for instances

On the client, children may not have replicated yet: `local remotes = ReplicatedStorage:WaitForChild("Remotes")`. On the server for server-created instances, direct indexing is fine. Use `FindFirstChild` for optional children.

## Attributes & tags

`part:SetAttribute("Damage", 20)`, `part:GetAttributeChangedSignal("Damage")`. `CollectionService:GetTagged("Lava")` + `GetInstanceAddedSignal("Lava")` to attach behavior to tagged parts — better than a script inside each part.

## Performance

- Cache services and frequently used instances in locals.
- Avoid per-frame `FindFirstChild` chains and creating tables/closures in `Heartbeat`.
- Use `table.create`, `table.clear`, integer `for` loops; `buffer` type for compact binary data.
- `--!native` (Native Code Generation) at top of math-heavy server modules.
- Profile with MicroProfiler (Ctrl+F6) and Script Profiler.

## Pitfalls

- `wait()`/`spawn()`/`delay()` are deprecated — use `task.*`.
- Never trust the client (see `roblox-networking`).
- `Instance.new("Part", parent)` with parent argument is slower; set properties then `Parent` last.
