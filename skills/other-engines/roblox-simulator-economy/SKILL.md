---
name: roblox-simulator-economy
description: Design and implement the economy of Roblox simulator-style games — core click/collect loop, currencies and sinks, upgrade cost curves, rebirth/prestige, pets and eggs with drop tables (and odds disclosure), zones and gating, offline earnings, gamepasses and developer products, inflation control, number formatting, and server-authoritative implementation. Use when building or balancing a progression/incremental economy on Roblox.
---

# Roblox Simulator Economy

## Core loop

```
Action (click/swing/mine) → earn Currency → buy Upgrades (more per action)
→ unlock Zones (higher yield) → Rebirth (reset for multiplier) → repeat
          ↘ Eggs/Pets (multipliers, collection goals) ↗
```

Each loop iteration should take ~1–5 min early, lengthening gradually. Always show the *next* goal on screen.

## Currencies

- **Soft currency** (Coins): earned constantly, spent on upgrades/eggs.
- **Secondary** (Gems): rarer, from zones/quests/playtime rewards, spent on premium-feeling items.
- **Prestige currency** (Rebirth tokens): earned on reset, buys permanent multipliers.
- **Robux** via developer products/gamepasses — never sell what breaks the core loop for non-payers.

## Cost & yield curves

Exponential cost, polynomial/exponential yield with a lower base:

```lua
local function upgradeCost(level: number): number
	return math.floor(10 * 1.15 ^ level)          -- base * growth^level
end
local function perClick(level: number, rebirths: number, petMult: number): number
	return (1 + level) * (1 + rebirths * 0.5) * petMult
end
```

- Growth 1.07–1.15 per level for frequent purchases; 1.5–3× per zone.
- Zone N requires ~3–10× the currency of zone N-1 and yields ~2–5× more per action.
- Prototype curves in a spreadsheet: target time-to-unlock per milestone, then fit parameters.

## Rebirth

Requirement grows (`base * 2^rebirths` or polynomial), resets coins/upgrades, grants permanent multiplier and rebirth tokens. First rebirth should land within the first session (~20–40 min).

## Pets & eggs

```lua
local Egg = {
	price = 500,
	drops = { { pet = "Dog", weight = 60 }, { pet = "Cat", weight = 30 }, { pet = "Dragon", weight = 9 }, { pet = "Unicorn", weight = 1 } },
}
local function roll(drops, rng: Random)
	local total = 0
	for _, d in drops do total += d.weight end
	local r = rng:NextNumber() * total
	for _, d in drops do
		r -= d.weight
		if r <= 0 then return d.pet end
	end
	return drops[#drops].pet
end
```

- Roll **on the server** with a server-side `Random`. The client only plays the hatch animation.
- Display odds clearly in the egg UI — Roblox policy requires disclosing odds for paid random items (and some regions restrict paid loot boxes; check current policy, use `PolicyService:GetPolicyInfoForPlayerAsync` → `ArePaidRandomItemsRestricted`).
- Equip limit + "equip best" button; craft duplicates into gold/rainbow tiers as a sink.

## Sinks (inflation control)

Upgrades, eggs, crafting/fusing, zone tolls, cosmetic shops, rebirth resets. If currency outpaces sinks, late-game numbers explode and purchases feel meaningless.

## Number formatting

```lua
local SUFFIX = { "", "K", "M", "B", "T", "Qa", "Qi", "Sx", "Sp", "Oc", "No", "Dc" }
local function format(n: number): string
	if n < 1000 then return tostring(math.floor(n)) end
	local i = math.floor(math.log10(n) / 3)
	return string.format("%.2f%s", n / 10 ^ (i * 3), SUFFIX[i + 1] or ("e" .. i * 3))
end
```

Luau numbers are doubles (precise integers up to 2^53, max ~1.8e308). For beyond-e308 games, use a big-number library (mantissa/exponent).

## Monetization (non-predatory)

- Gamepasses: 2× coins, auto-click, extra pet slots, faster hatch — permanent convenience.
- Developer products: currency packs, boosts (timed 2× for 15 min), server luck boosts.
- Free rewards: daily login streak, playtime gifts, group join bonus, likes/favorites goal codes.
- Grant purchases in `ProcessReceipt` and persist before returning `PurchaseGranted` (see `roblox-datastores`).

## Implementation rules

- All currency math on the server; client sends "clicked" (rate-limited ≤ ~10–20/s, validated by tool cooldown).
- Auto-clickers: allow them via an official auto-click gamepass/feature rather than fighting them.
- Offline earnings: store `lastLogout` timestamp, grant `rate * min(elapsed, cap)` on join.
- Log economy telemetry (`AnalyticsService:LogEconomyEvent`) to see sources/sinks and tune.
