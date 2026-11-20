---
name: create-game-assets
description: Create or source game art and placeholder assets — art style decisions, resolution and palette for pixel art, sprite sheets and atlases, generating placeholder sprites/tiles procedurally in code or SVG, free asset sources and licenses (CC0, CC-BY), tools (Aseprite, LibreSprite, Blender, Krita), naming and export pipelines. Use when a game needs sprites, tiles, icons, UI art, 3D models, or placeholders.
---

# Create Game Assets

## Decide the style first (constraints = consistency)

| Decision | Typical choice |
|---|---|
| Base resolution | 320×180 or 480×270 (pixel art, scales ×4/×6 to 1080p), 1920×1080 (HD 2D) |
| Tile/sprite grid | 16×16 or 32×32 pixel art; character ≈ 1–2 tiles tall |
| Palette | Fixed palette of 16–32 colors (Lospec: PICO-8, Endesga 32, Resurrect 64) |
| Outline | Dark outline (readable) or selective outline |
| Light direction | Pick one (top-left) and apply everywhere |
| 3D | Low-poly flat-shaded + palette texture (one 256×256 gradient atlas) is fast and cohesive |

Readability > detail: silhouettes must read at gameplay zoom; player, enemies, hazards, pickups need distinct value/hue contrast from the background.

## Placeholders that don't block progress

- Colored rectangles/circles with a letter, generated in code — size them exactly like final art so swapping is drop-in.
- Programmatic SVG/PNG generation (Python + Pillow, or canvas) for tilesets with consistent palette:

```python
from PIL import Image, ImageDraw
PALETTE = {"bg": "#1d1d2b", "grass": "#4a8f3a", "dirt": "#7a5230", "edge": "#2e5a24"}
def tile(fill, edge=None, size=16):
    im = Image.new("RGBA", (size, size), fill)
    if edge:
        ImageDraw.Draw(im).rectangle([0, 0, size - 1, 3], fill=edge)   # grass top
    return im
sheet = Image.new("RGBA", (16 * 4, 16))
for i, t in enumerate([tile(PALETTE["grass"], PALETTE["edge"]), tile(PALETTE["dirt"])]):
    sheet.paste(t, (i * 16, 0))
sheet.save("assets/tiles.png")
```

- Engine primitives (CSG/ProBuilder/BSP) for greyboxing 3D levels.

## Sprite sheets & atlases

- Uniform grid sheets for animations (frame size fixed; name animations in the engine) or packed atlases (TexturePacker, free-tex-packer, Aseprite *Export Sprite Sheet* with JSON).
- Add 1–2 px padding/extrusion to prevent bleeding.
- Power-of-two atlas sizes (1024, 2048) for broad GPU compatibility and compression.
- Frame counts: idle 2–6, run 6–8, attack 3–6 (with anticipation, contact, recovery frames), at 8–12 fps for pixel art.

## Tools

- Pixel art: **Aseprite** (paid, best), LibreSprite (free fork), Pixelorama (free).
- Painting/HD: Krita (free), Photoshop, Procreate.
- Vector UI/icons: Inkscape, Figma.
- 3D: **Blender** (free) → glTF export; Blockbench for voxel/low-poly game models.
- Tilemaps/levels: Tiled, LDtk.
- Fonts: free pixel fonts with OFL license (e.g. from Google Fonts / itch.io).

## Free asset sources (check each license)

- **Kenney.nl** — huge CC0 packs (2D, 3D, UI, audio).
- **OpenGameArt.org** — mixed licenses (CC0, CC-BY, CC-BY-SA, GPL); read per asset.
- **itch.io game assets** — many free/paid packs; license on each page.
- **Quaternius**, **Poly Pizza** — CC0 low-poly 3D.
- **ambientCG**, **Poly Haven** — CC0 PBR textures/HDRIs/models.

License rules: CC0 = no attribution required. CC-BY = credit the author (keep a `CREDITS.md`). CC-BY-SA = derivatives must share alike (affects modified assets). "Non-commercial" = cannot sell the game. Don't use ripped assets from commercial games.

## AI-generated art

If used, check the tool's license terms and platform rules (Steam requires disclosure of AI-generated content in the store submission). Keep consistency by post-editing to the fixed palette/resolution.

## Pipeline & naming

```
art/source/   (.aseprite, .blend, .kra — committed via Git LFS)
assets/       (exported .png, .glb, .ogg — what the engine loads)
```

Naming: `category_name_variant_state.png` → `enemy_slime_green_run.png`. Lowercase, no spaces. Automate exports (Aseprite CLI: `aseprite -b in.aseprite --sheet out.png --data out.json --format json-array`; Blender CLI with a Python export script).

## Checklist

- [ ] Palette and resolution documented in the GDD
- [ ] Silhouette/contrast test (greyscale screenshot)
- [ ] CREDITS.md with every third-party asset + license
- [ ] Source files versioned (Git LFS for binaries)
