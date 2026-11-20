---
name: godot-shaders
description: Write Godot Shading Language shaders for Godot 4 — canvas_item (2D) and spatial (3D) shaders, uniforms, hint_* annotations, screen-reading effects, visual shaders, and common effects like hit flash, dissolve, outline, water and toon shading. Use for any .gdshader work.
---

# Godot Shaders

GLSL-like language. First line declares the type: `shader_type canvas_item;` (2D/UI), `spatial;` (3D), `particles;`, `sky;`, `fog;`.

## 2D: hit flash + dissolve

```glsl
shader_type canvas_item;

uniform vec4 flash_color : source_color = vec4(1.0);
uniform float flash_amount : hint_range(0.0, 1.0) = 0.0;
uniform sampler2D noise_tex;
uniform float dissolve : hint_range(0.0, 1.0) = 0.0;
uniform vec4 edge_color : source_color = vec4(1.0, 0.5, 0.0, 1.0);

void fragment() {
	vec4 c = texture(TEXTURE, UV);
	c.rgb = mix(c.rgb, flash_color.rgb, flash_amount);
	float n = texture(noise_tex, UV).r;
	if (n < dissolve) discard;
	if (n < dissolve + 0.05) c.rgb = edge_color.rgb;
	COLOR = c;
}
```

Drive from GDScript: `sprite.material.set_shader_parameter("flash_amount", 1.0)`. Make materials **Local to Scene** (or duplicate) when each instance needs its own values; or use `instance uniform` (3D only) to avoid material duplication.

## 2D outline

```glsl
shader_type canvas_item;
uniform vec4 outline_color : source_color = vec4(0,0,0,1);
uniform float width = 1.0;
void fragment() {
	vec2 px = TEXTURE_PIXEL_SIZE * width;
	float a = texture(TEXTURE, UV).a;
	float around = max(max(texture(TEXTURE, UV + vec2(px.x,0)).a, texture(TEXTURE, UV - vec2(px.x,0)).a),
	                   max(texture(TEXTURE, UV + vec2(0,px.y)).a, texture(TEXTURE, UV - vec2(0,px.y)).a));
	COLOR = mix(vec4(outline_color.rgb, around), texture(TEXTURE, UV), a);
}
```

Needs transparent padding around sprites in the atlas.

## Screen-space post effects

```glsl
shader_type canvas_item;
uniform sampler2D screen_tex : hint_screen_texture, filter_linear_mipmap;
uniform float strength = 0.004;
void fragment() {
	vec2 uv = SCREEN_UV;
	COLOR = vec4(texture(screen_tex, uv + vec2(strength,0)).r,
	             texture(screen_tex, uv).g,
	             texture(screen_tex, uv - vec2(strength,0)).b, 1.0);
}
```

Put it on a full-rect `ColorRect` inside a `CanvasLayer` above the game. (`SCREEN_TEXTURE` built-in was removed in Godot 4 — use the `hint_screen_texture` uniform.)

## 3D: spatial toon

```glsl
shader_type spatial;
uniform vec4 albedo : source_color = vec4(1);
uniform int bands = 3;
void fragment() { ALBEDO = albedo.rgb; }
void light() {
	float ndl = clamp(dot(NORMAL, LIGHT), 0.0, 1.0);
	float stepped = floor(ndl * float(bands)) / float(bands);
	DIFFUSE_LIGHT += stepped * ATTENUATION * LIGHT_COLOR / PI;
}
```

Render modes: `render_mode unshaded;`, `cull_disabled;`, `depth_draw_always;`, `blend_add;`. Vertex displacement in `vertex()` (e.g. waves: `VERTEX.y += sin(VERTEX.x * 2.0 + TIME) * 0.2;`).

## Uniform hints cheat sheet

`source_color` (sRGB color), `hint_range(min,max,step)`, `hint_normal`, `hint_default_white/black`, `filter_nearest` (pixel art), `repeat_enable`, `hint_screen_texture`, `hint_depth_texture`. Global uniforms: Project Settings → Shader Globals, `global uniform vec3 wind;`.

## Visual Shaders

Same capabilities, node-graph UI. Good for artists; convert to text via *Convert to Shader* when it gets big.

## Pitfalls

- Pixel art shimmering: sample with `filter_nearest`.
- Branching on uniforms is cheap; branching on per-pixel noise in loops is not — keep mobile shaders simple and use the Compatibility renderer budget in mind.
- Shader compile stutter on first use: pre-warm by rendering objects once during a loading screen, or rely on Godot 4.4+ ubershader pipeline compilation.
