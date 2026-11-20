---
name: game-docs
description: Write game design documentation that people actually use — one-page pitch, concept doc, lean living GDD, feature specs, systems/balance docs, level briefs, technical design docs, art/audio bibles, task breakdowns and milestones, and post-mortems — with templates. Use when the user asks for a GDD, pitch, design doc, spec, roadmap, or wants to plan a game project.
---

# Game Docs

Documentation exists to **align people and record decisions**. Keep it short, living, and linked. A 100-page GDD nobody reads is worse than a one-pager everyone knows.

## 1. One-page pitch (start here)

```markdown
# <Title> — one-pager
**Genre / platform / target audience:** Roguelite deckbuilder · PC & Steam Deck · fans of Slay-the-Spire-likes who like tactics
**Hook (1 sentence):** Your deck is also your map: every card you play reshapes the dungeon.
**Core loop:** Draw → play cards to move & fight on a 5×5 grid → shape the next room → collect cards → descend.
**Pillars (3):** 1) Every card has two uses 2) Readable, fair combat 3) Runs under 40 minutes
**Player fantasy:** A cartographer-wizard bending the dungeon to their will.
**Comparable titles:** (2–3 games and what we take/differ from each)
**Scope:** 1 developer, 9 months, 120 cards, 3 acts.
**Unique selling points:** …
**Risks & open questions:** Is dual-use cards too complex? → prototype by <date>.
```

Pillars are the most useful part: every feature must serve at least one pillar or be cut.

## 2. Lean living GDD (wiki or folder of short pages)

```
00-overview.md          (the one-pager, kept current)
01-core-mechanics.md    (verbs, rules, controls)
02-progression.md       (meta, unlocks, difficulty curve)
03-content/             (one page per enemy/item family, data tables link)
04-ui-ux.md             (screen flow diagram, HUD layout)
05-art-direction.md     (palette, references, do/don't examples)
06-audio-direction.md
07-tech.md              (engine, architecture, platforms, tools)
08-production.md        (milestones, risks)
decisions.md            (date, decision, reason — append-only log)
```

Each page: purpose, current state, open questions, last-updated date. Link to the actual data (spreadsheets, resource files) instead of copying numbers.

## 3. Feature spec template

```markdown
# Feature: Grappling Hook
**Goal / pillar served:** Movement mastery (Pillar 1)
**Player story:** As a player, I can latch onto hook points to swing across gaps.
**Rules:** Range 8 tiles; only "hook" tiles; release keeps momentum; one hook at a time.
**Controls:** Hold RB/Shift to aim (auto-target nearest in cone), release to detach.
**Feedback:** Rope VFX, attach sound, controller rumble on attach.
**Edge cases:** Hook point destroyed while attached → detach with current velocity.
**Tuning values:** (link to data file)
**Out of scope:** Hooking enemies (v2).
**Acceptance criteria:** [ ] can complete test level 3-2 [ ] works at 30/60/144 FPS [ ] gamepad + KB/M
**Open questions:** Should it refresh the dash?
```

## 4. Level brief

Level goal, mechanics introduced/tested, beat chart (sequence of moments), intensity curve, rough map sketch, secrets, estimated play time, dependencies (assets, enemies).

## 5. Technical design doc

Architecture overview (diagram), engine version, key systems and data flow, save format & versioning, platforms and performance budgets, build/CI pipeline, third-party SDKs, risks.

## 6. Production

- Milestones: **Prototype → Vertical slice** (one polished level representing final quality) → **Alpha** (feature complete) → **Beta** (content complete, bug fixing) → **Release candidate**.
- Task breakdown per milestone with estimates; track in an issue tracker/kanban.
- Risk list with mitigation and owner.
- Cut list prepared in advance (what goes first if late).

## 7. Post-mortem

What went right (3–5), what went wrong (3–5), numbers (time spent, scope delivered, sales/ratings), lessons for next project. Write within two weeks of shipping.

## Writing rules

- Lead with the decision/answer; details after.
- Use tables, diagrams, GIFs/screens over prose.
- Mark speculative content clearly (`TBD`, `Proposal`).
- One owner per document; date every change; delete outdated sections rather than piling up.
