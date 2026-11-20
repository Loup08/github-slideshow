---
name: roblox-datastores
description: Persist Roblox player data safely with DataStoreService — session locking, UpdateAsync vs SetAsync, retries and budgets, autosave and BindToClose, schema versioning, OrderedDataStore leaderboards, MemoryStoreService for cross-server state, and libraries like ProfileStore. Use for saving player progress, inventories, currencies, or leaderboards.
---

# Roblox DataStores

Enable *Game Settings → Security → Enable Studio Access to API Services* to test in Studio (use a separate test place/universe to protect live data).

## Recommendation

For production player data, use a battle-tested library — **ProfileStore** (successor to ProfileService) — which implements session locking, autosave, and data reconciliation. Hand-rolled saving is the #1 cause of data loss/duping. If writing your own, follow everything below.

## Minimal robust pattern

```lua
--!strict
local DataStoreService = game:GetService("DataStoreService")
local Players = game:GetService("Players")

local store = DataStoreService:GetDataStore("PlayerData_v1")
local TEMPLATE = { version = 1, coins = 0, inventory = {}, level = 1 }
local cache: { [Player]: typeof(TEMPLATE) } = {}

local function retry<T>(fn: () -> T, attempts: number): (boolean, T?)
	for i = 1, attempts do
		local ok, result = pcall(fn)
		if ok then return true, result end
		warn("DataStore attempt", i, result)
		task.wait(2 ^ i)
	end
	return false, nil
end

local function reconcile(data: any)
	for k, v in TEMPLATE do
		if data[k] == nil then data[k] = if type(v) == "table" then table.clone(v) else v end
	end
	-- migrations: if data.version < 2 then ... data.version = 2 end
	return data
end

Players.PlayerAdded:Connect(function(player)
	local ok, data = retry(function()
		return store:GetAsync("u_" .. player.UserId)
	end, 3)
	if not ok then
		player:Kick("Could not load data, please rejoin.")   -- never play on default data if load failed
		return
	end
	cache[player] = reconcile(data or table.clone(TEMPLATE))
end)

local function save(player: Player)
	local data = cache[player]
	if not data then return end
	retry(function()
		return store:UpdateAsync("u_" .. player.UserId, function(_old)
			return data
		end)
	end, 3)
end

Players.PlayerRemoving:Connect(function(player)
	save(player)
	cache[player] = nil
end)

game:BindToClose(function()                 -- server shutdown: ~30 s budget
	local pending = 0
	for _, p in Players:GetPlayers() do
		pending += 1
		task.spawn(function()
			save(p)
			pending -= 1
		end)
	end
	local deadline = os.clock() + 25
	while pending > 0 and os.clock() < deadline do task.wait() end
end)

task.spawn(function()                        -- autosave
	while true do
		task.wait(120)
		for _, p in Players:GetPlayers() do task.spawn(save, p) end
	end
end)
```

## Rules

- Key by **UserId**, never username.
- **UpdateAsync** over SetAsync: it's atomic and lets you check the old value (e.g. reject writes from a stale session).
- **Session locking**: store `{ lockedBy = game.JobId, lockTime = os.time() }` and refuse to load if another live server holds the lock — prevents duplication when players hop servers quickly. ProfileStore does this for you.
- Respect request budgets (`DataStoreService:GetRequestBudgetForRequestType`); don't save on every coin change — save on interval/leave.
- Values must be JSON-serializable: no Instances, Vector3, Color3 (convert to tables), string keys or array tables (not mixed), 4 MB max per key.
- Version schemas and migrate on load.
- Data deletion requests (Right to Erasure): be able to remove a user's keys.

## Leaderboards

`OrderedDataStore` for global top lists: `store:SetAsync(userId, score)`, `GetSortedAsync(false, 100)` → pages. Refresh every few minutes, not per request.

## Cross-server live data

**MemoryStoreService** (sorted maps, queues, hash maps) for matchmaking, live leaderboards, trading locks; data expires (max TTL ~45 days). **MessagingService** for cross-server broadcasts (announcements).

## Monetization

Developer product purchases: `MarketplaceService.ProcessReceipt` must grant the item and **persist** before returning `PurchaseGranted`; record processed `PurchaseId`s to avoid double grants.
