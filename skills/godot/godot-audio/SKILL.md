---
name: godot-audio
description: Godot 4 audio — AudioStreamPlayer/2D/3D, audio buses and effects, volume sliders in dB, music crossfades, randomized SFX with AudioStreamRandomizer, polyphony, interactive music, and import settings. Use for sound effects, music systems, and audio settings menus.
---

# Godot Audio

## Players

- `AudioStreamPlayer` — non-positional (music, UI).
- `AudioStreamPlayer2D` — panned/attenuated by distance in 2D.
- `AudioStreamPlayer3D` — 3D positional with doppler, attenuation models.

Set `bus` on each player. Use `max_polyphony` > 1 to let one player overlap rapid repeats (gunfire, footsteps).

## Bus layout (Audio tab at bottom)

```
Master
├── Music
├── SFX   (+ Limiter / Compressor)
├── UI
└── Voice
```

Effects: Reverb on an `Ambience`/`SFX_Reverb` send bus, LowPassFilter on Music when paused/underwater, Compressor with sidechain from Voice to duck music.

## Volume settings (linear slider → dB)

```gdscript
func set_bus_volume(bus_name: String, linear: float) -> void:
	var idx := AudioServer.get_bus_index(bus_name)
	AudioServer.set_bus_volume_db(idx, linear_to_db(linear))   # linear 0..1
	AudioServer.set_bus_mute(idx, linear <= 0.001)
```

Persist in a `ConfigFile`; apply on boot.

## Variation

Use `AudioStreamRandomizer`: add several footstep clips, set `random_pitch` (e.g. 1.1) and `random_volume_offset_db` (2). Assign it as the player's stream — no code needed.

## SFX helper (autoload)

```gdscript
extends Node
var _pool: Array[AudioStreamPlayer] = []

func _ready() -> void:
	for i in 12:
		var p := AudioStreamPlayer.new()
		p.bus = "SFX"
		add_child(p)
		_pool.append(p)

func play(stream: AudioStream, pitch_jitter := 0.05) -> void:
	for p in _pool:
		if not p.playing:
			p.stream = stream
			p.pitch_scale = randf_range(1.0 - pitch_jitter, 1.0 + pitch_jitter)
			p.play()
			return
```

For positional one-shots, spawn an `AudioStreamPlayer2D` at the position and `queue_free` on `finished`; avoid attaching death sounds to the node being freed.

## Music

Crossfade:

```gdscript
func crossfade(to: AudioStream, time := 1.5) -> void:
	var next := $MusicB if $MusicA.playing else $MusicA
	var prev := $MusicA if next == $MusicB else $MusicB
	next.stream = to
	next.volume_db = -60
	next.play()
	var t := create_tween().set_parallel()
	t.tween_property(next, "volume_db", 0.0, time)
	t.tween_property(prev, "volume_db", -60.0, time)
	t.chain().tween_callback(prev.stop)
```

Godot 4.3+ adds `AudioStreamInteractive` (clip transitions on beat), `AudioStreamPlaylist`, and `AudioStreamSynchronized` (vertical layering stems) for adaptive music without code.

## Import

- SFX: WAV (low latency). Music: Ogg Vorbis (streams, small). MP3 supported.
- Loop music: import dock → Loop, set loop offset.
- Music players must keep playing while paused: `process_mode = ALWAYS`.

## Pitfalls

- Linear volumes in dB fields: `-6 dB` ≈ half perceived loudness; never set `volume_db` to linear values.
- Web exports: audio starts only after a user gesture.
- Too many simultaneous voices → mud. Limit per-sound concurrency (see `audio-design`).
