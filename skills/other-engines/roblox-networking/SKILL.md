---
name: roblox-networking
description: Roblox client-server networking — replication model, RemoteEvent/RemoteFunction/UnreliableRemoteEvent usage, server-side validation and rate limiting, anti-exploit principles, network ownership, attributes for replicated state, and bandwidth optimization with buffers. Use for any feature that crosses the client/server boundary on Roblox.
---

# Roblox Networking

## Model

- The **server** is authoritative. Changes made by the server to replicated Instances (Workspace, ReplicatedStorage) replicate to clients.
- Changes made by a client (LocalScript) do **not** replicate (FilteringEnabled), except: physics of parts the client has **network ownership** of (its own character!), and some character/animation properties.
- Exploiters can run arbitrary client code and fire any remote with any arguments. Design accordingly.

## Remotes

Create in `ReplicatedStorage/Remotes` (server-side, at edit time or startup).

```lua
-- Server
local Remotes = game.ReplicatedStorage.Remotes
local BuyItem: RemoteEvent = Remotes.BuyItem

local lastBuy: { [Player]: number } = {}
BuyItem.OnServerEvent:Connect(function(player: Player, itemId: unknown)
	-- 1. type-check every argument
	if typeof(itemId) ~= "string" then return end
	-- 2. rate limit
	local now = os.clock()
	if lastBuy[player] and now - lastBuy[player] < 0.25 then return end
	lastBuy[player] = now
	-- 3. validate against server state (price, ownership, distance to shop)
	local item = ShopCatalog[itemId]
	if not item then return end
	local data = PlayerData.get(player)
	if data.coins < item.price then return end
	-- 4. apply on server, replicate result
	data.coins -= item.price
	table.insert(data.inventory, itemId)
	Remotes.InventoryChanged:FireClient(player, data.inventory)
end)

-- Client
Remotes.BuyItem:FireServer("sword_01")
```

- `RemoteEvent`: fire-and-forget (`FireServer`, `FireClient`, `FireAllClients`).
- `RemoteFunction`: request/response (`InvokeServer`). **Never `InvokeClient`** from the server — a malicious client can hang it forever.
- `UnreliableRemoteEvent`: for frequent, loss-tolerant data (aim direction, cosmetic effects); ordering/delivery not guaranteed, small payload limit.

## Validation checklist

- Types (`typeof`), NaN/inf checks for numbers (`x ~= x`), string length limits.
- Ownership: does this player own the item/plot/pet they reference?
- Distance/line-of-sight for interactions (`(char.HumanoidRootPart.Position - shop.Position).Magnitude < 15`).
- Cooldowns tracked on the server.
- Never accept "damage", "coins", or "position" values from clients as truth. Client says *"I swung at target X"*; server checks range, cooldown, and computes damage.

## Combat with latency

Client plays animation/VFX immediately (feels responsive), sends intent; server validates (with some lag tolerance) and applies damage; other clients get replicated results / VFX via `FireAllClients`.

## Replicated state

- Attributes (`SetAttribute`) and `ValueObject`s replicate automatically — good for small per-object state (health, team, level).
- `leaderstats` folder under Player → shows in the leaderboard.
- Server → client streaming of big tables: send diffs, not whole tables.

## Network ownership

The server can assign physics ownership: `part:SetNetworkOwner(nil)` (server simulates — authoritative, use for important physics like balls in sports games) or `SetNetworkOwner(player)` (smooth for that player — vehicles they drive). Automatic ownership by proximity otherwise.

## Bandwidth

- Fire remotes on change, not per frame. Batch updates (send every 0.1 s).
- `buffer` type to pack data compactly; libraries like **ByteNet**/**Zap**/**Blink** generate typed, compact remotes.
- `StreamingEnabled` on Workspace for large maps (clients only receive nearby parts — use `WaitForChild`/`ModelStreamingMode` carefully).

## Pitfalls

- Trusting `Humanoid.Health` or `WalkSpeed` set on the client — clients can set their own character values; enforce speed/teleport checks server-side.
- Remotes created by the client are invisible to the server.
- Yielding inside `OnServerEvent` handlers is fine, but don't leave player-keyed tables uncleaned on `PlayerRemoving`.
