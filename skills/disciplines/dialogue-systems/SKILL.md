---
name: dialogue-systems
description: Design and implement game dialogue — linear and branching conversations, choices with conditions and consequences, variables/flags, barks, typewriter text and skipping, portraits and speaker data, localization-ready data formats, and authoring tools (Ink, Yarn Spinner, Dialogic, Twine). Use for NPC conversations, quests text, cutscenes, or visual-novel style scripts.
---

# Dialogue Systems

## Use an authoring language when content is non-trivial

| Tool | Engines | Notes |
|---|---|---|
| **Ink** (inkle) | Unity (ink-unity), Godot (inkgd/GodotInk), web (inkjs), Unreal (plugins) | Writer-friendly text, knots/stitches, variables, very mature |
| **Yarn Spinner** | Unity (official), Godot (C#), others | Node-based, commands, localization built in |
| **Dialogic 2** | Godot | Editor plugin with timelines, characters, portraits |
| **Twine** | Prototyping, web | Export to JSON (Twison/Harlowe) |
| Custom JSON | Any | Fine for small games; build a validator |

## Core data model (custom)

```json
{
  "id": "blacksmith_intro",
  "nodes": {
    "start": {
      "speaker": "blacksmith", "portrait": "neutral",
      "text": "dlg.blacksmith.greeting",
      "choices": [
        { "text": "dlg.blacksmith.ask_sword", "next": "sword", "if": "gold >= 50" },
        { "text": "dlg.blacksmith.ask_rumor", "next": "rumor", "if": "!flags.heard_rumor" },
        { "text": "dlg.common.bye", "next": null }
      ]
    },
    "sword": { "speaker": "blacksmith", "text": "dlg.blacksmith.sword", "do": ["gold -= 50", "give sword_iron"], "next": null },
    "rumor": { "speaker": "blacksmith", "text": "dlg.blacksmith.rumor", "do": ["set flags.heard_rumor"], "next": "start" }
  }
}
```

- Text by **localization key**, not inline strings (or inline + extracted later by a tool).
- Conditions (`if`) and effects (`do`) as a tiny expression language or a fixed set of commands — don't `eval` arbitrary code.
- Speakers in a separate table (name key, color, voice blip, portraits by emotion).

## Runtime

```
DialogueRunner
  start(dialogue_id)
  current node → emit line(speaker, text, portrait)
  wait for: advance input | choice selection
  apply effects → evaluate next → filter choices by conditions
  end → emit finished, restore player control
```

Keep runner logic (data, state) separate from the UI (view) so it's testable and reusable for barks/cutscenes.

## Presentation

- **Typewriter** reveal (30–60 chars/s); first press completes the line, second press advances. Pause briefly on punctuation.
- Voice "blips" per character (pitch-shifted per speaker) if no VO.
- Show the speaker name and portrait; highlight the active speaker in multi-person scenes.
- Choices: 2–4 options, short; indicate consequences or skill checks (`[Persuade 60%]`).
- Lock player movement during dialogue, but allow the pause menu.
- Log/backlog of recent lines (essential in VNs).

## State & consequences

- Store variables in a central game state (`flags`, `affinity.npc`, `quest.stage`) that both dialogue and gameplay read — and save it (see `save-systems`).
- Prefer a few meaningful branches that reconverge ("branch and bottleneck") over exponential trees.
- Track `seen` nodes to change NPC greetings on revisit.

## Barks

Short contextual lines (combat taunts, "I hear something"): weighted random from pools with cooldowns per speaker and per line, priority levels, no repeats within N plays.

## Localization

- Leave 30–40% extra space for longer languages (German, French).
- Avoid string concatenation for sentences; use placeholders: `"{name} gave you {count} {item}"` with plural rules (ICU MessageFormat or engine tr with plurals).
- Fonts covering target scripts (CJK fonts are large — subset them).

## Testing

Validate dialogue data in CI: every `next` exists, every key has a translation, conditions parse, no unreachable nodes, no dead ends without explicit end.
