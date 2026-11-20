---
name: game-jam
description: Plan and finish a game jam entry (24–72 h, Ludum Dare, GMTK, Global Game Jam, itch.io jams) — theme brainstorming, scoping to one mechanic, hour-by-hour schedule, team roles, engine and tool choice, cutting features, asset rules, build early and often, web export, submission page, and post-jam. Use when the user is participating in or preparing for a game jam.
---

# Game Jam Workflow

## Before the jam

- Read the rules: solo/team, duration, asset rules (Ludum Dare Compo requires all assets made during the jam; Jam/most itch jams allow pre-made/free assets with credit), allowed tools, AI-content policy.
- Prepare a **starter template** in your engine: input map, scene switcher, title/pause/game-over screens, audio manager, web export preset tested on itch.io. (Allowed in almost all jams; check rules.)
- Install and test: engine + export templates, art tool, sfx generator (jsfxr), screen recorder.

## Theme → idea (first 1–2 hours max)

1. Write 15–20 raw interpretations of the theme (literal, metaphorical, mechanical, inverted).
2. For each promising one, describe the **core verb** and the 10-second loop.
3. Pick the idea that is (a) fun to do in 10 seconds, (b) buildable with your skills in ~40% of the time, (c) clearly tied to the theme.
4. Write a one-paragraph pitch + a must/should/could list.

## Scope rules

- **One mechanic**, done well, with 5–10 minutes of content.
- If you can't prototype the core in the first 25% of the time, cut or pivot.
- No multiplayer, no complex save systems, no big narrative trees, no custom engines (unless that's your thing).
- Cut half of "should" by the halfway point. Always.

## Schedule (48 h example)

| Time | Goal |
|---|---|
| 0–2 h | Theme, idea, plan |
| 2–8 h | Core mechanic playable with placeholders (rectangles) |
| 8–10 h | **First web build uploaded (private)** |
| 10–24 h | Content: levels/waves, win/lose, restart |
| 24–32 h | Art & audio pass, menus, tutorial prompts |
| 32–40 h | Juice (see `game-feel`), playtest with others, balance |
| 40–44 h | Bug fixes only, **feature freeze** |
| 44–46 h | Final builds, itch page, screenshots/GIF |
| 46–48 h | Buffer. Submit early |

Sleep. Tired developers create bugs that cost more time than they save.

## Team roles (2–4 people)

Programmer(s), artist, audio/music, designer/level builder/producer. One person owns the build and merges; use Git with small commits, separate scenes per person to avoid merge conflicts (scene files conflict badly).

## Make it playable for judges

- Web build (most jam players won't download executables). See `itch-publish`, engine export skills.
- Title screen shows controls; first level teaches the mechanic without text walls.
- Short: 5–15 minutes. Easy to start, a satisfying ending screen.
- Restart button; no way to soft-lock.
- Volume isn't too loud; mute key.
- Test on a different machine/browser before submitting.

## Submission page

Title, 3–5 screenshots + an animated GIF, controls, how it fits the theme, credits (all assets + licenses, tools), team members, known issues. Ratings depend a lot on first impressions.

## After

Play and rate other entries (most jams surface games of people who rate), fix critical bugs (usually allowed after rating period or as a separate post-jam version), write a post-mortem: what went right, what went wrong, what to reuse in the template.

Load also: `prototype-fast` (finding the fun), `game-feel` (polish hour), `create-game-assets` (placeholders), `audio-design` (jsfxr).
