---
name: prototype-fast
description: Rapid game prototyping to find the fun — defining a testable question, greyboxing with primitives, hardcoding and cutting corners on purpose, tuning knobs and debug UI, quick playtest loops, paper prototypes, multiple parallel prototypes, killing ideas early, and deciding when to move to production. Use when exploring a new mechanic or game idea before committing to full development.
---

# Prototype Fast

A prototype answers **one question**. It is disposable. Optimize for learning speed, not code quality.

## 1. Write the question

Bad: "Make my game." Good:

- "Is swinging with a grappling hook between moving platforms fun for 2 minutes?"
- "Do players understand that time only moves when they move?"
- "Is placing towers on the enemy's path (mazing) interesting with 3 tower types?"

Define success: "Testers want to play again" / "They discover X without being told."

## 2. Pick the fastest tool

- The engine you know best. Not the one you'll ship with if switching costs days.
- Paper prototype for card/board/economy/turn-based ideas (index cards, dice) — minutes, not days.
- Spreadsheet for economy/progression curves.

## 3. Build only the core loop

- Primitives and colored shapes, no art (`create-game-assets` placeholders).
- One level/arena that exercises the mechanic.
- Hardcode values, global state, copy-paste code — fine. No menus, no saving, no settings.
- Timebox: 1–3 days per prototype. If it isn't fun after the timebox, it probably needs a different core, not more features.

## 4. Expose tuning knobs

Make every number live-tweakable: exported variables in the inspector, an in-game debug panel (Dear ImGui, engine UI), or hot-reloaded JSON. Feel lives in the numbers (`physics-tuning`, `game-feel`). Keep a "presets" list of good values.

## 5. Playtest early and quietly

- Within the first day, put it in front of someone else.
- Don't explain. Watch. Note confusion, delight, boredom, repeated attempts.
- Ask: "What were you trying to do?", "What was most/least fun?", "Would you play more?"
- 3–5 testers reveal most big problems.

## 6. Iterate or kill

- Change one major variable per iteration; keep notes (date, change, result).
- Make several small prototypes in parallel (e.g. 3 × 1-day prototypes) and compare instead of polishing one.
- Kill without guilt: the knowledge transfers. Archive the prototype with a short write-up.

## 7. Juice test

A little juice (sound, hit flash, shake) can reveal whether a mechanic is satisfying — add a 30-minute juice pass before judging feel (see `game-feel`). But don't polish to compensate for a weak core.

## 8. Moving to production

When the core is proven:

- Write a short design doc (`game-docs`) capturing what worked, the tuned values, and the next questions.
- **Rewrite** cleanly rather than growing prototype code into production (reuse the learnings and the tuned numbers).
- Next prototype questions: content scalability (can we make 50 levels of this?), progression, meta.

## Anti-patterns

- Building menus, save systems, or tools first.
- Spending days on art before the mechanic is fun.
- Prototype without a question → endless tinkering.
- Asking testers "Do you like it?" (they'll be polite) instead of observing.
