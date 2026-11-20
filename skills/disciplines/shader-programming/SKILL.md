---
name: shader-programming
description: Engine-agnostic shader programming — GPU pipeline basics, vertex vs fragment work, coordinate spaces, UV tricks, SDFs, noise, lighting models (Lambert, Blinn-Phong, toon, rim), common game effects (hit flash, dissolve, outline, water, scrolling, palette swap, CRT), post-processing, translating between GLSL/HLSL/WGSL/Godot/Unity Shader Graph, and performance on mobile/web. Use when writing or porting shaders in any engine.
---

# Shader Programming

## Mental model

- **Vertex shader**: runs per vertex; transforms positions (object → world → view → clip), passes data (UV, normal, color) to fragments. Do displacement here (wind, waves).
- **Fragment/pixel shader**: runs per pixel covered; computes color. Cost scales with screen area × overdraw.
- Data in: **uniforms** (same for whole draw: time, color, textures), **attributes** (per vertex), **varyings** (interpolated vertex → fragment).

## Translation table

| Concept | GLSL | HLSL (Unity/Unreal) | Godot | WGSL |
|---|---|---|---|---|
| vec types | `vec2/3/4` | `float2/3/4` | `vec2/3/4` | `vec2f/vec3f/vec4f` |
| Lerp | `mix` | `lerp` | `mix` | `mix` |
| Fract | `fract` | `frac` | `fract` | `fract` |
| Texture sample | `texture(s, uv)` | `tex.Sample(sampler, uv)` / `SAMPLE_TEXTURE2D` | `texture(s, uv)` | `textureSample(t, s, uv)` |
| Time | uniform you pass | `_Time.y` (Unity), Time node (Unreal) | `TIME` | uniform |
| Output color | `fragColor`/`gl_FragColor` | return `float4` | `COLOR`/`ALBEDO` | return `vec4f` |
| Matrix mul | `M * v` | `mul(M, v)` | `M * v` | `M * v` |

Node-based equivalents: Unity Shader Graph, Unreal Material Editor, Godot Visual Shader, three.js TSL.

## Building blocks

```glsl
// Remap & shaping
float remap(float v, float a, float b, float c, float d) { return c + (v - a) * (d - c) / (b - a); }
float s = smoothstep(0.4, 0.6, x);          // soft threshold
uv = uv * 2.0 - 1.0;                        // center UVs at 0

// Circle SDF with antialiasing
float d = length(uv) - 0.5;
float aa = fwidth(d);
float circle = 1.0 - smoothstep(-aa, aa, d);

// Scrolling texture
vec2 suv = uv + vec2(time * 0.1, 0.0);

// Hash noise (cheap, for dithering/sparkle)
float hash(vec2 p) { return fract(sin(dot(p, vec2(127.1, 311.7))) * 43758.5453); }
```

Value/Perlin/Simplex noise: prefer a **noise texture** lookup over procedural noise on mobile/web.

## Lighting models

```glsl
float ndl = max(dot(N, L), 0.0);                       // Lambert diffuse
vec3 H = normalize(L + V);
float spec = pow(max(dot(N, H), 0.0), shininess);      // Blinn-Phong specular
float toon = floor(ndl * bands) / bands;               // cel bands (or sample a ramp texture)
float rim = pow(1.0 - max(dot(N, V), 0.0), 3.0);       // rim/fresnel glow
```

PBR: use the engine's built-in lighting and only customize albedo/normal/roughness — rewriting PBR rarely pays off.

## Effect recipes

- **Hit flash**: `color.rgb = mix(color.rgb, flashColor, amount)` driven 1→0 over 0.1 s.
- **Dissolve**: `if (noise(uv) < threshold) discard;` + emissive edge band where `noise < threshold + width`.
- **Outline (2D)**: sample alpha at 4–8 offsets; if any neighbor opaque and self transparent → outline color.
- **Outline (3D)**: inverted hull (second pass, front-face culling, extrude along normals) or post-process edge detection on depth/normals.
- **Palette swap**: grayscale index texture → lookup row in a palette texture (`texture(palette, vec2(index, paletteRow))`).
- **Water**: scrolling normal maps ×2 in different directions, depth-based color/foam (scene depth − fragment depth), refraction via screen texture offset.
- **Wind sway (vertex)**: `pos.x += sin(time * speed + worldPos.x) * strength * uv.y` (weight by height).
- **CRT post**: barrel distortion of UV, scanlines `sin(uv.y * res.y * π)`, chromatic aberration, vignette.

## Post-processing

Render scene to a texture, run full-screen passes (bloom = bright pass → blur downsample chain → add; vignette; color grading via LUT). Combine effects into one pass where possible.

## Performance

- Move work: per-pixel → per-vertex → per-draw (uniform) → precomputed texture.
- Avoid dynamic loops and heavy branching on per-pixel data; branching on uniforms is fine.
- Texture reads dominate on mobile; minimize samples, use mipmaps, compressed formats.
- Overdraw: big transparent particles/full-screen layers are the #1 GPU cost in 2D.
- Use half precision (`mediump`, `half`) on mobile where acceptable.
- Variant explosion: each keyword/toggle combination compiles a shader — keep options few.
- Precompile/warm shaders to avoid first-use hitches.

## Debugging

Output intermediate values as color (`COLOR = vec4(vec3(value), 1.0)`), visualize UVs (`vec4(uv, 0, 1)`), normals (`N * 0.5 + 0.5`). Tools: RenderDoc, PIX, Xcode GPU capture, Spector.js (WebGL).
