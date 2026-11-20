---
name: godot-export
description: Export Godot 4 games — installing export templates, export presets for Windows/macOS/Linux/Web/Android/iOS, command-line and CI exports, PCK and resource filters, code signing and notarization, and web hosting headers. Use when building or shipping a Godot project.
---

# Godot Export

## Setup

1. Editor → Manage Export Templates → download templates matching the **exact** editor version.
2. Project → Export → add presets. Presets live in `export_presets.cfg` (commit it; secrets go in `.godot/export_credentials.cfg`, which is not committed).

## CLI / CI

```bash
godot --headless --path . --export-release "Windows Desktop" build/win/game.exe
godot --headless --path . --export-release "Linux" build/linux/game.x86_64
godot --headless --path . --export-release "Web" build/web/index.html
godot --headless --path . --export-debug "Android" build/android/game.apk
godot --headless --path . --export-pack "Linux" build/patch.pck   # data only
```

Run `godot --headless --import --path .` (4.2+: `--import`) once before exporting on a fresh CI checkout so the `.godot/imported` cache exists. Community action: `abarichello/godot-ci` Docker images, or download the official editor + templates in your workflow.

## Platform notes

**Windows** — set icon (.ico) and version info; rcedit is no longer required in recent versions. Sign with `signtool` to reduce SmartScreen warnings.

**macOS** — export `.zip`/`.dmg`/`.app`; requires codesign + notarization for distribution outside the App Store (configure Apple ID / API key in preset or sign afterward with `codesign` + `notarytool`). Universal binary by default.

**Linux** — `x86_64` and `arm64`; ship as `.tar.gz`/AppImage/Flatpak or via Steam.

**Web** — Compatibility renderer. Threaded builds need cross-origin isolation headers:

```
Cross-Origin-Opener-Policy: same-origin
Cross-Origin-Embedder-Policy: require-corp
```

itch.io has a *SharedArrayBuffer support* checkbox; GitHub Pages can't set headers — use the **no-threads** export option (default since 4.3) or a service-worker shim. Audio requires a click to start.

**Android** — install OpenJDK 17 + Android SDK (Editor Settings → Export → Android). Use Gradle build for plugins. Create a release keystore and keep it backed up; AAB for Play Store. Set min/target SDK per Play requirements.

**iOS** — exports an Xcode project; build/sign on macOS.

## Resource filters & size

- *Resources* tab: export all, selected scenes, or exclude filters (`*.psd, docs/*`). Non-resource files (`.json`, `.txt`) need an include filter like `*.json`.
- Embed PCK into the executable for single-file distribution (desktop).
- Compression of textures: VRAM compressed for 3D (ETC2/ASTC for mobile, S3TC/BPTC for desktop); lossless for pixel art.
- Strip unused modules with a custom export template build (`scons` with `disable_3d=yes`, etc.) for small web builds.

## Feature tags

`OS.has_feature("web")`, `"mobile"`, `"debug"`, `"steam"` (custom tag on a preset) to branch behavior; project settings can be overridden per feature tag (`display/window/size/mode.mobile`).

## Pre-ship checklist

- [ ] Version in Project Settings → Application → Config → Version.
- [ ] Debug prints/cheats behind `OS.is_debug_build()`.
- [ ] Icons + boot splash.
- [ ] Test the **exported** build on each OS (missing files from filters are the #1 bug).
- [ ] `user://` save location tested; settings persist.
