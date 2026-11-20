---
name: visual-novel
description: Compositional template for visual novels and narrative games — script format and tooling (Ren'Py, Ink, Yarn, Dialogic, custom), branching structure and flags, routes and endings, characters/sprites/expressions, backgrounds and CGs, text box UX (auto, skip, backlog, rollback), choice design, save/load slots, gallery and unlocks, localization, and which skills to load. Use when building a visual novel, dating sim, or dialogue-heavy narrative game.
---

# Visual Novel Template

## Tool choice

| Tool | When |
|---|---|
| **Ren'Py** | Pure VN; Python-based script, everything built in (saves, rollback, menus, gallery), exports desktop/mobile/web |
| **Ink / Yarn Spinner** + engine | VN segments inside a larger game (Unity/Godot/web) |
| **Dialogic 2** (Godot) | Godot projects with VN features |
| Custom | Only if you need unusual presentation; budget for all the UX features below |

## Ren'Py minimal script

```renpy
define m = Character("Mira", color="#c8a2ff", image="mira")
default affinity_mira = 0

label start:
    scene bg cafe with fade
    show mira happy at center with dissolve
    m "You came! I wasn't sure you would."
    menu:
        "Of course I came.":
            $ affinity_mira += 1
            m smile "I'm glad."
        "I almost didn't.":
            m sad "Oh..."
    if affinity_mira >= 3:
        jump mira_route
    jump common_day2
```

## Structure

- **Common route** → branches into character/plot **routes** based on accumulated flags/points → **endings** (good/normal/bad/true).
- "Branch and bottleneck": choices change scenes and flags but reconverge to keep content manageable; big splits at a few key moments.
- Track flags in a central state; show consequences (character reacts, later callback lines).
- True ending often gated by having seen other endings (persistent data).

## Choice design

- 2–4 options, each with a distinct intent (not right/wrong/funny).
- Avoid "gotcha" bad ends from innocuous choices; foreshadow.
- Optional: show which choices were previously selected (on replays).

## Presentation assets

- Character sprites: base + expressions (neutral, happy, sad, angry, surprised, blush) as layered images to save memory/space; consistent canvas size and anchor.
- Backgrounds: 1920×1080 (or 16:9 at target res), day/evening/night variants.
- CGs: key illustrated moments, unlockable in a gallery.
- UI: name box, text box with adjustable opacity, quick menu (Back, History, Skip, Auto, Save, Q.Save, Q.Load, Prefs).

## Required UX features

- Text speed + auto-advance speed settings; click to complete line, click to advance.
- **Skip** (read text only, or all) and **backlog/history** with voice replay.
- **Rollback** (scroll back to previous lines/choices) — expected by VN players.
- Save/load slots with screenshot thumbnails and timestamps; quick save/load.
- Hide UI to view art; screenshot key.
- Persistent unlocks: gallery, music room, endings list, "read" text tracking across playthroughs.

## Audio

Music per scene mood, ambience, SFX for emphasis, optional voice acting (huge cost: scope carefully, voice key lines only). See `audio-design`.

## Writing workflow

Outline flowchart (routes, key choices, endings) → draft in plain text/Ink → import to script → playtest reading time (≈ 200–250 words/min; a 50k-word VN ≈ 3–4 hours) → proofread pass → localization.

## Localization

Ren'Py has built-in translation generation; for custom engines extract all lines with IDs (see `dialogue-systems`). Plan text box size for longer languages and font coverage (CJK).

## Scope tiers

- **Jam**: 5k–15k words, 1–2 characters, 2–3 endings, 3–5 backgrounds.
- **Small commercial**: 40k–100k words, 3–4 routes, 10+ CGs.

## Testing

Flowchart coverage: every route and ending reached; flags at each branch logged; save/load from every chapter; text overflow check per language.
