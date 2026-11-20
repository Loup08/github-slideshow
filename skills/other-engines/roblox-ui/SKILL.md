---
name: roblox-ui
description: Build Roblox UI — ScreenGui/SurfaceGui/BillboardGui, UDim2 scale vs offset, UIListLayout/UIGridLayout/UIAspectRatioConstraint/UIScale for responsive layouts across phone/PC/console, TextScaled caveats, TweenService animations, gamepad selection, safe areas, and declarative libraries (React-lua, Fusion). Use for HUDs, shops, menus, and inventories on Roblox.
---

# Roblox UI

## GUI containers

- `ScreenGui` in `StarterGui` → copied into each player's `PlayerGui` (set `ResetOnSpawn = false` for persistent HUD/menus).
- `SurfaceGui` on parts (in-world screens), `BillboardGui` (nameplates, health bars above heads).
- `ScreenGui.ScreenInsets = DeviceSafeInsets` / `CoreUISafeInsets` to avoid notches and the Roblox top bar.

## Responsive layout

- `UDim2.fromScale(0.3, 0.1)` scales with screen; `UDim2.fromOffset(200, 50)` fixed pixels. Mix: scale for position/size, offset for padding.
- `AnchorPoint = Vector2.new(0.5, 0.5)` to center on position.
- `UIAspectRatioConstraint` to keep buttons square across aspect ratios.
- Layouts: `UIListLayout` (with `Padding`, `SortOrder = LayoutOrder`), `UIGridLayout`, `UIPadding`, `UIFlexItem` (flex grow/shrink), `UISizeConstraint`, `UITextSizeConstraint`.
- `UIScale` on the root, driven by viewport size, for consistent sizing on phones vs 4K.
- Test with Studio's **Device Emulator** (phone, tablet, console 10-foot UI).

## Text

`TextScaled` is convenient but leads to inconsistent sizes; prefer fixed `TextSize` + `UITextSizeConstraint`, or `TextScaled` with `MaxTextSize`. Use `RichText` for inline color. Fonts: `Font.new("rbxasset://fonts/families/GothamSSm.json", Enum.FontWeight.Bold)` via `FontFace`.

## Wiring a HUD (LocalScript)

```lua
--!strict
local Players = game:GetService("Players")
local TweenService = game:GetService("TweenService")
local player = Players.LocalPlayer
local hud = player:WaitForChild("PlayerGui"):WaitForChild("HUD") :: ScreenGui
local coinsLabel = hud:WaitForChild("Coins") :: TextLabel

local leaderstats = player:WaitForChild("leaderstats")
local coins = leaderstats:WaitForChild("Coins") :: IntValue

local function update()
	coinsLabel.Text = tostring(coins.Value)
	coinsLabel.Size = UDim2.fromScale(0.22, 0.09)
	TweenService:Create(coinsLabel, TweenInfo.new(0.15, Enum.EasingStyle.Back, Enum.EasingDirection.Out),
		{ Size = UDim2.fromScale(0.2, 0.08) }):Play()
end
coins.Changed:Connect(update)
update()
```

Server is authoritative for values; UI only displays and sends requests via RemoteEvents/Functions (see `roblox-networking`).

## Buttons & input

`button.Activated` (works for mouse, touch, and gamepad) instead of `MouseButton1Click`. For gamepad: set `Selectable`, `GuiService.SelectedObject = firstButton` when opening a menu, `NextSelectionUp/Down` for explicit navigation, `SelectionGroup` on containers.

## Animations

`TweenService:Create(obj, TweenInfo.new(t, style, dir), goals):Play()`. Common: menu slide-in (Position), popup scale (`UIScale.Scale` 0→1 with Back easing), fade (`GroupTransparency` on a `CanvasGroup`).

## Declarative UI

For complex UI (shops, inventories), use **React-lua** (Roblox's official port, via Wally `jsdotlua/react`) or **Fusion** (reactive state). They make state-driven UI far easier than manual instance mutation.

## Performance

- Avoid updating UI every frame; update on change events.
- Many frames with `ClipsDescendants`/`CanvasGroup` are costly; CanvasGroup renders to a texture (memory).
- Use `ScrollingFrame` with `AutomaticCanvasSize` and only create visible rows for large lists.

## Pitfalls

- Editing `StarterGui` at runtime doesn't change players' UI — edit `PlayerGui`.
- UI hidden under top bar on mobile: use `ScreenInsets` / `GuiService:GetGuiInset()`.
- `ResetOnSpawn = true` (default) recreates UI on respawn, breaking script references.
