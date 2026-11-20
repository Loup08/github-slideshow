---
name: unity-build-size
description: Reduce Unity build size and download size — reading the Build Report, texture/audio/mesh compression, sprite atlases, managed code stripping, removing unused packages, Addressables for on-demand content, and platform-specific limits (WebGL, mobile stores). Use when a Unity build is too large.
---

# Unity Build Size

## 1. Measure first

- After a build, open the **Editor.log** (Console → ⋮ → Open Editor Log) and search for "Build Report": size per category and the largest assets list.
- Install the *Build Report Inspector* package or use `BuildReport.packedAssets` from a build script for per-asset breakdown.
- Compare: Textures usually dominate, then Audio, Meshes, then code (IL2CPP binaries).

## 2. Textures (largest win)

- **Max Size**: set per asset to what's on screen (UI icons 128–256, not 2048). Use platform overrides.
- **Compression**: ASTC (mobile), BC7/DXT (desktop), *Crunch* for smaller downloads at some quality cost. Avoid *None* except for pixel art that truly needs it (pixel art is small anyway).
- Disable **Generate Mip Maps** for UI and 2D sprites.
- Disable **Read/Write** (doubles memory) unless needed.
- Pack 2D sprites in **Sprite Atlases** (fewer textures, less padding waste, fewer draw calls).
- NPOT textures can't compress on some formats — keep power-of-two or multiples of 4.

## 3. Audio

- Music: Vorbis, quality 50–70%, *Streaming* load type, force mono for ambience if fine.
- SFX: Vorbis/ADPCM, *Decompress On Load* for short clips, sample rate override 22050 Hz for non-critical sounds.
- Remove unused takes; audio files are often left in project unused.

## 4. Meshes & animation

- Mesh Compression: Medium/High; disable *Read/Write*; strip unused vertex channels (Player Settings → Optimize Mesh Data).
- Animation compression: *Optimal*, with error tolerances; remove scale curves when unused.

## 5. Code

- IL2CPP + **Managed Stripping Level** High (test thoroughly; use `link.xml` for reflection).
- Player Settings → *Strip Engine Code*.
- Remove unused packages in Package Manager (Visual Scripting, Timeline, Test Framework in runtime, etc.) — each adds code and assets.
- Avoid huge generic instantiations and LINQ-heavy code on IL2CPP.

## 6. Content strategy

- Everything in `Resources/` is included **even if unused** — move assets out and reference directly or via Addressables.
- **Addressables** remote groups: ship a small base build and download levels/DLC on demand. Required for staying under store install limits (Google Play base AAB limit; use Play Asset Delivery for larger content).
- Split shared assets into their own Addressable group to avoid duplication (*Analyze → Check Duplicate Bundle Dependencies*).

## 7. Shaders

Shader variants bloat size and build time. Use URP's shader stripping settings, remove unused features in the URP asset (e.g. no SSAO → strip), and a Shader Variant Collection / `IPreprocessShaders` to strip unneeded keywords.

## WebGL specifics

- Compression: Brotli (best) with correct `Content-Encoding` hosting; Code Optimization *Disk Size* (Unity 6: *Disk Size with LTO*).
- Disable exceptions (or *Explicitly Thrown Only*), strip engine code, avoid large fonts (use TMP SDF atlases with limited character sets).
- Target < 20–30 MB initial download for web portals.

## Checklist

- [ ] Build report reviewed, top 20 assets justified
- [ ] Texture max sizes/platform overrides set
- [ ] No unused assets in Resources
- [ ] Unused packages removed
- [ ] Stripping enabled and tested
- [ ] Audio load types set
