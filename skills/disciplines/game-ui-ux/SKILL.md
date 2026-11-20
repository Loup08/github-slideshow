---
name: game-ui-ux
description: Game UI/UX design for HUDs and menus in any engine — information hierarchy, diegetic vs non-diegetic UI, HUD layout and safe areas, menu flow (title, pause, settings), controller-first navigation, feedback states, onboarding/tutorials, inventory and shop patterns, readability and scaling, accessibility (colorblind, text size, subtitles, remapping), and localization. Use when designing or reviewing any player-facing interface.
---

# Game UI/UX

## Hierarchy

Show only what the player needs **now**. Rank information:

1. Critical, always visible: health, ammo/resource for the current action.
2. Contextual: interaction prompts, boss bar, objective marker — appear when relevant.
3. On demand: map, inventory, stats, quest log.

Put critical info near where eyes already are (near the character, crosshair, or the corners). Fade HUD elements when full/unchanged (e.g. full stamina bar hides).

## HUD layout

- Keep within **safe areas** (~5% margin; TVs overscan; phone notches).
- Corners: top-left health/status, top-right minimap/objective, bottom-right ammo/abilities, bottom-center prompts/subtitles.
- Consistent visual language: colors (red damage, green heal, gold currency), icons + numbers.
- Diegetic UI (ammo on the gun, health as character state) increases immersion but must stay readable.

## Menus

```
Title → [Continue | New Game | Load | Settings | Credits | Quit]
Pause → [Resume | Settings | Controls | Restart/Return to checkpoint | Quit to Title]
Settings → Audio | Video | Controls | Gameplay | Accessibility | Language
```

- "Continue" first when a save exists; skip unskippable logos after first boot.
- Every screen: a clear way back (B/Esc), focus starts on the most likely option.
- Confirm destructive actions (quit without saving, overwrite save, reset settings).
- Apply settings instantly with preview; resolution changes get a "keep changes?" timeout.

## Controller-first navigation

Design for D-pad/stick + confirm/back first; mouse/touch second. Visible focus state, predictable grid navigation, shoulder buttons for tabs, hold-to-confirm for dangerous actions, button prompts that match the current device.

## Feedback & states

Every interactive element needs normal, hover/focus, pressed, disabled, and selected states. Purchases/actions confirm with sound + animation; errors explain *why* ("Not enough gold — need 30 more").

## Onboarding

Teach by doing: contextual prompts at the moment the mechanic is needed ("Press A to jump" near the first gap), disappearing after success. Avoid text walls; allow re-reading in a help menu. Never block the screen with a tutorial during danger.

## Inventory / shop patterns

- Grid with sort/filter, comparison deltas (+5 ATK in green), equip from the same screen.
- Show cost, owned quantity, and affordability state.
- Tooltips on focus, not only hover (controllers).
- Avoid more than 2 nested menus deep for frequent actions; provide quick-use slots/radial menus.

## Readability

- Minimum text size: ~28 px at 1080p for TV/console, ~18–20 px PC; scale with resolution.
- Contrast ratio ≥ 4.5:1 for text; outlines/drop shadows on text over gameplay.
- Pixel fonts at integer scales only.
- Numbers: thousand separators or abbreviations (1.2K), consistent decimal rules.

## Accessibility (baseline)

- Text size and UI scale options.
- Colorblind-friendly: never encode meaning by color alone — add shapes/icons/patterns; test with simulators (deuteranopia/protanopia/tritanopia).
- Subtitles (size, background, speaker names), captions for important sounds.
- Full remapping, hold/toggle options, reduced motion/shake, adjustable game speed or assist modes where suitable.
- Screen reader support for menus when feasible (some engines/platforms support it).

## Localization

- Externalize strings with keys from day one; avoid text baked into images.
- Layouts that grow (auto-size containers), ~35% expansion budget.
- Right-to-left support if targeting Arabic/Hebrew; CJK font fallback.
- Date/number formatting per locale.

## Review checklist

- [ ] Can a new player tell health, objective, and current resource in 1 second?
- [ ] Every screen navigable with gamepad only and with mouse only
- [ ] Back works everywhere; focus never lost
- [ ] Readable on a 13" laptop and a TV from the couch
- [ ] Colorblind simulation passes
- [ ] All strings localizable
