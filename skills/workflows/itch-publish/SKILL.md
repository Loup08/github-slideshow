---
name: itch-publish
description: Publish games on itch.io — project page setup, HTML5/web builds (embed size, fullscreen, SharedArrayBuffer/COOP-COEP option, zip structure), downloadable builds per OS, butler CLI uploads and channels with CI, pricing/pay-what-you-want and revenue share, devlogs, jams, page design, and troubleshooting common web export issues. Use when uploading a game or jam entry to itch.io.
---

# itch.io Publishing

## Create the project

Dashboard → Create new project:

- **Kind of project**: HTML (playable in browser) or Downloadable.
- Pricing: Free, Paid (minimum price), or "No payments" / pay-what-you-want with suggested donation. Open revenue sharing lets you set itch's cut (default 10%).
- Visibility: Draft → Restricted (secret URL for testers) → Public.

## Web (HTML5) builds

Zip the **contents** of the export folder with `index.html` at the zip root (not inside a subfolder).

- Upload → tick *This file will be played in the browser*.
- Embed options: viewport size matching the game's resolution (e.g. 960×540 or 1280×720), *Mobile friendly*, *Fullscreen button*, *Automatically start on page load* (off if it needs a click for audio anyway).
- **SharedArrayBuffer support** checkbox: enable for engines that need threads/COOP-COEP (threaded Godot web exports, some Unity/Emscripten builds). Note it may break third-party content in the iframe (ads, external fonts/CDNs).
- Files: itch limits the number of files and the size per file in HTML zips (check current limits; large Unity builds should use Brotli/Gzip compression with *Decompression Fallback* enabled since itch doesn't set Content-Encoding).

Engine notes:

- Godot: Web export; use no-threads export unless you enable SharedArrayBuffer; test audio starts after click.
- Unity: WebGL; Compression format Gzip/Brotli + *Decompression Fallback*, or disabled compression.
- Phaser/PixiJS/three.js (Vite): `vite build` with `base: './'` so asset paths are relative.
- Ebitengine: `index.html` + `wasm_exec.js` + `.wasm` in the zip root.
- pygame: pygbag output `build/web` zipped.
- LÖVE: love.js output.

## Downloadable builds

One file per platform (Windows .zip, macOS .zip/.dmg, Linux .tar.gz/.zip), tagged with the platform checkboxes so the itch app installs the right one.

## butler (CLI uploads)

```bash
butler login
butler push build/web       username/my-game:html5   --userversion 1.0.3
butler push build/windows   username/my-game:windows --userversion 1.0.3
butler push build/linux     username/my-game:linux   --userversion 1.0.3
butler push build/mac       username/my-game:mac     --userversion 1.0.3
butler status username/my-game
```

Channels named with `windows`, `linux`, `mac`, `osx`, `android`, `html5`/`web` auto-tag platforms. butler uploads only diffs (fast patches) and builds are installable/updatable via the itch app. In CI, set `BUTLER_API_KEY` (from itch.io API keys) as a secret.

For an HTML channel pushed via butler, still set *played in browser* on the upload in the edit page once.

## Page that converts

- Cover image (630×500, readable title), banner/background, theme colors matching the game.
- GIF or trailer at the top, 3–5 screenshots.
- First line: genre + hook. Controls clearly listed. Short feature list.
- Credits + licenses for third-party assets.
- Tags, genre, made-with (engine), accessibility notes, average session length.
- Community: comments or a discussion board enabled.

## Jams

Join the jam page → *Submit your project* → select the project (it must be published or restricted per jam rules). Edits after the deadline may be locked — make sure the submitted build is final. See `game-jam`.

## After release

Devlogs (posts appear in followers' feeds and boost visibility), updates via butler, bundles and sales, collections. Watch analytics (views → downloads/plays) to judge the page.

## Troubleshooting

- Black screen / 404 on assets: index.html not at zip root, or absolute paths (`/assets/...`) — make them relative.
- "SharedArrayBuffer is not defined": enable the itch checkbox or export without threads.
- Audio silent: needs user gesture; add a click-to-start screen.
- Game too big for the embed: set proper viewport size and fullscreen button; handle resize in the engine.
- Keyboard input scrolls the page: prevent default on arrow keys/space in the canvas, or click the iframe to focus (tell players).
