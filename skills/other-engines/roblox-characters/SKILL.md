---
name: roblox-characters
description: Work with Roblox characters — Player vs Character lifecycle (CharacterAdded, respawn), Humanoid properties and states, R15 vs R6, playing animations through the Animator, custom movement abilities (dash, double jump, sprint), StarterCharacter overrides, HumanoidDescription, NPCs with PathfindingService, and the newer physics-based character controllers. Use for movement, animation, NPCs, or avatar customization on Roblox.
---

# Roblox Characters

## Lifecycle

```lua
Players.PlayerAdded:Connect(function(player)
	player.CharacterAdded:Connect(function(character)
		local humanoid = character:WaitForChild("Humanoid") :: Humanoid
		humanoid.Died:Connect(function() onDied(player) end)
	end)
end)
```

A new Character model is created on every respawn — re-bind per character. Client: `Players.LocalPlayer.CharacterAdded` / `player.Character or player.CharacterAdded:Wait()`. Scripts in `StarterCharacterScripts` run fresh for each spawn (easy cleanup).

`Players.CharacterAutoLoads = false` + `player:LoadCharacter()` for custom spawn flows (menus, rounds).

## Humanoid

Key props: `WalkSpeed` (16), `JumpPower`/`JumpHeight` (`UseJumpPower`), `Health`/`MaxHealth`, `AutoRotate`, `HipHeight`. Methods: `TakeDamage` (respects ForceField), `MoveTo(pos)` (NPCs), `ChangeState`, `SetStateEnabled(Enum.HumanoidStateType.Climbing, false)`, `GetState()`. Events: `StateChanged`, `Running`, `Jumping`, `FreeFalling`, `Died`.

Movement speed changes for gameplay (sprint) are set on the client for responsiveness, but validate on the server (exploiters can change their own WalkSpeed).

## Animations

```lua
local animator = humanoid:WaitForChild("Animator") :: Animator
local anim = Instance.new("Animation")
anim.AnimationId = "rbxassetid://1234567890"
local track = animator:LoadAnimation(anim)    -- load once, reuse the track
track.Priority = Enum.AnimationPriority.Action
track:Play(0.1)
track:GetMarkerReachedSignal("Hit"):Connect(function() applyHitbox() end)
```

- Animations must be owned by the experience owner (user or group) to play.
- Play player animations from the **client** — they replicate automatically through the Animator. NPC animations from the server.
- Override default walk/run/idle by replacing the IDs in the `Animate` LocalScript (copy from a playtest character into StarterCharacterScripts).

## Abilities (client LocalScript)

```lua
-- Double jump
local jumps, MAX = 0, 2
humanoid.StateChanged:Connect(function(_, new)
	if new == Enum.HumanoidStateType.Landed then jumps = 0 end
end)
UserInputService.JumpRequest:Connect(function()
	local state = humanoid:GetState()
	if state == Enum.HumanoidStateType.Freefall and jumps < MAX - 1 then
		jumps += 1
		humanoid:ChangeState(Enum.HumanoidStateType.Jumping)
	elseif state ~= Enum.HumanoidStateType.Freefall then
		jumps = 0
	end
end)

-- Dash
local root = character:WaitForChild("HumanoidRootPart") :: BasePart
root:ApplyImpulse(root.CFrame.LookVector * root.AssemblyMass * 60)
```

Use `ContextActionService:BindAction` for actions that need mobile buttons + gamepad automatically.

## R15 vs R6

R15 (15 parts, default, layered clothing, smoother animation) vs R6 (6 parts, retro, simpler). Set in Game Settings → Avatar. Write code against `HumanoidRootPart` and `Humanoid`, not specific limbs, to stay rig-agnostic.

## Custom characters

- Put a model named `StarterCharacter` in StarterPlayer to replace the avatar (needs Humanoid + HumanoidRootPart + rig).
- `HumanoidDescription` + `humanoid:ApplyDescription(desc)` (server) to set outfits, scale, accessories.
- Disable avatar scaling or limit it in Game Settings for competitive fairness.

## NPCs & pathfinding

```lua
local path = PathfindingService:CreatePath({ AgentRadius = 2, AgentHeight = 5, AgentCanJump = true, Costs = { Water = 20 } })
path:ComputeAsync(npcRoot.Position, target)
if path.Status == Enum.PathStatus.Success then
	for _, wp in path:GetWaypoints() do
		if wp.Action == Enum.PathWaypointAction.Jump then npcHumanoid.Jump = true end
		npcHumanoid:MoveTo(wp.Position)
		npcHumanoid.MoveToFinished:Wait()
	end
end
```

Recompute on `path.Blocked`. Set NPC network owner to the server (`root:SetNetworkOwner(nil)`) to avoid jitter.

## New character controllers

Roblox's physics-based **ControllerManager** (GroundController, AirController, SwimController, ClimbController) replaces Humanoid-driven movement for custom feel and physics interactions. Consider it for movement-heavy games; it requires more setup (sensors, state switching).
