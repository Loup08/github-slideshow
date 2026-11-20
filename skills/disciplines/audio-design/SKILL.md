---
name: audio-design
description: Game audio design across engines — SFX for feedback and game feel, variation (pitch/volume randomization, round-robin), mixing with buses and ducking, loudness targets, voice limiting and priorities, adaptive/layered music, spatial audio, UI sounds, formats and compression, accessibility (subtitles, mono, visual cues), and sourcing/creating sounds (sfxr/jsfxr, Bfxr, Freesound, middleware like FMOD/Wwise). Use when adding or improving sound and music in a game.
---

# Audio Design

Sound is half of game feel. Every player action and game event needs audible feedback.

## SFX inventory (minimum)

- Player: footsteps (per surface), jump, land, attack/shoot, hurt, death, dash.
- Interaction: pickup (distinct per value), door, switch, chest, error/denied.
- Enemies: alert/notice, attack telegraph (**crucial for fairness**), hurt, death.
- UI: hover/move, confirm, back, open/close menu, purchase, notification.
- World: ambience loops per area, environmental one-shots.

## Variation

Never play the exact same sample repeatedly:

- Random pitch ±5–10% and volume ±2 dB per play.
- 3–5 variants per frequent sound, round-robin or random-no-repeat.
- Layer: e.g. gunshot = transient click + body + tail + mechanical; vary layers independently.

## Mixing

- Buses: Master → Music, SFX (→ Player, Enemies, World), UI, Voice/Dialogue, Ambience.
- **Ducking/sidechain**: lower music and SFX under dialogue; brief duck on big impacts.
- Priority & voice limits: cap simultaneous instances per sound (e.g. max 4 coin pickups, steal oldest); global voice cap. Prevents mud and clipping.
- Limiter on Master to avoid clipping.
- Loudness: aim around **-16 to -14 LUFS integrated** for the overall mix on PC/console reference, consistent across scenes; normalize assets before import.
- Settings menu: Master, Music, SFX, Voice sliders (perceptual — map linear slider to dB), and mute when unfocused option.

## Music

- Loop seamlessly (cut on bar boundaries, export with exact loop points).
- **Vertical layering**: same tempo stems (drums, bass, melody) faded in by intensity (combat, danger).
- **Horizontal resequencing**: switch sections on the next bar/beat (explore → combat → victory sting).
- Stingers for events (level up, discovery).
- Crossfade 1–2 s between tracks; silence is also a tool.

Engine tools: Godot AudioStreamInteractive/Synchronized, Unity Audio Random Container + mixer snapshots, Unreal MetaSounds + Quartz, or middleware **FMOD**/**Wwise** (free tiers for indies) for parameters, adaptive music, and live tuning.

## Spatial audio

- 2D: stereo pan by screen position + distance attenuation; off-screen enemies audible but quieter.
- 3D: attenuation curves (inverse with min/max distance), occlusion low-pass when behind walls, reverb zones per room.
- Listener on the camera (or between camera and player for third-person).

## Formats

- SFX: WAV 44.1/48 kHz 16-bit source; compressed Vorbis/ADPCM in builds; short sounds decompressed in memory.
- Music/ambience: Ogg Vorbis (or Opus) streamed, ~128–192 kbps.
- Web: provide Ogg + MP3/AAC for Safari compatibility where needed; audio unlock on first user input.

## Sourcing & creating

- Retro SFX generators: **jsfxr**/sfxr, Bfxr, ChipTone — instant placeholder or final for pixel games.
- Libraries: Freesound.org (check CC license per sound), Kenney audio packs (CC0), Sonniss GDC bundles (royalty-free).
- Record/edit: Audacity (free), Reaper (cheap DAW). Foley with a phone mic is fine for prototypes.
- Music: commission, royalty-free libraries (check license for streaming/YouTube), or tools like BeepBox/FamiStudio for chiptune.

## Accessibility

- Subtitles/captions with speaker names and size options; captions for important non-speech sounds ("[footsteps behind]").
- Mono audio option; don't rely solely on audio cues — pair with visual indicators.
- Separate volume sliders; avoid sudden extreme loudness.

## Checklist

- [ ] Every input and state change has feedback sound
- [ ] Enemy attacks have audible telegraphs
- [ ] No sample repeats identically more than twice in a row
- [ ] Voice limits on frequent sounds
- [ ] Mix tested on headphones, laptop speakers, and TV
