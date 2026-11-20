---
name: threejs-materials-lighting
description: Materials and lighting in three.js — MeshStandard/Physical/Toon/Basic choice, PBR textures and color spaces, physically based light units, shadows tuning, environment maps with PMREM/RGBELoader, tone mapping, fog, post-processing (EffectComposer, bloom), custom ShaderMaterial and onBeforeCompile, and TSL node materials for WebGPU. Use for the look of a three.js scene.
---

# three.js Materials & Lighting

## Material choice

| Material | Use |
|---|---|
| `MeshBasicMaterial` | Unlit: UI in 3D, skyboxes, stylized flat, fastest |
| `MeshLambertMaterial` | Cheap diffuse (mobile) |
| `MeshStandardMaterial` | Default PBR (metalness/roughness) |
| `MeshPhysicalMaterial` | Clearcoat, transmission (glass), sheen, iridescence — costlier |
| `MeshToonMaterial` | Cel shading with `gradientMap` (NearestFilter texture) |
| `ShaderMaterial` / node materials | Custom effects |

Share material instances between meshes — each unique material is a separate program/state change.

## Textures & color space

```ts
const tl = new THREE.TextureLoader();
const albedo = tl.load('/tex/rock_albedo.jpg');
albedo.colorSpace = THREE.SRGBColorSpace;            // color textures only
const normal = tl.load('/tex/rock_normal.jpg');       // data textures stay linear (default)
const orm = tl.load('/tex/rock_orm.jpg');
for (const t of [albedo, normal, orm]) { t.wrapS = t.wrapT = THREE.RepeatWrapping; t.repeat.set(4, 4); t.anisotropy = renderer.capabilities.getMaxAnisotropy(); }
const mat = new THREE.MeshStandardMaterial({ map: albedo, normalMap: normal, roughnessMap: orm, metalnessMap: orm, aoMap: orm });
```

Pixel art textures: `magFilter = NearestFilter`, `generateMipmaps = false`.

## Lights (physically based units since r155)

- `DirectionalLight` intensity ~1–5 (sun), `HemisphereLight`/`AmbientLight` for fill (~0.5–2).
- `PointLight`/`SpotLight` intensity in candela, with `decay = 2` (physical) — values in the tens/hundreds for room-scale lights; set `distance` to limit range.
- Keep real-time lights few (each light adds shader cost to every lit material). Bake lighting into lightmaps for static scenes (`lightMap` + uv2/`channel`).

## Environment lighting (biggest quality win)

```ts
import { RGBELoader } from 'three/addons/loaders/RGBELoader.js'; // newer releases: HDRLoader
const hdr = await new RGBELoader().loadAsync('/env/studio_1k.hdr');
hdr.mapping = THREE.EquirectangularReflectionMapping;
scene.environment = hdr;           // IBL for all PBR materials
scene.background = hdr;            // optional; or blur: scene.backgroundBlurriness = 0.3
```

Or `RoomEnvironment` + `PMREMGenerator` for a no-download neutral studio environment. `scene.environmentIntensity` to balance.

## Shadows

```ts
sun.castShadow = true;
sun.shadow.mapSize.set(2048, 2048);
Object.assign(sun.shadow.camera, { left: -20, right: 20, top: 20, bottom: -20, near: 1, far: 60 });
sun.shadow.bias = -0.0005;      // acne
sun.shadow.normalBias = 0.02;   // peter-panning vs acne trade-off
```

Visualize with `new THREE.CameraHelper(sun.shadow.camera)`. Make the shadow camera follow the player for large worlds. `CSM` addon for cascaded shadows.

## Tone mapping & fog

`renderer.toneMapping = THREE.ACESFilmicToneMapping` (or `AgXToneMapping`, `NeutralToneMapping` for accurate product colors), `toneMappingExposure`. `scene.fog = new THREE.Fog(color, near, far)` or `FogExp2` — also hides far clipping.

## Post-processing

```ts
import { EffectComposer } from 'three/addons/postprocessing/EffectComposer.js';
import { RenderPass } from 'three/addons/postprocessing/RenderPass.js';
import { UnrealBloomPass } from 'three/addons/postprocessing/UnrealBloomPass.js';
import { OutputPass } from 'three/addons/postprocessing/OutputPass.js';
const composer = new EffectComposer(renderer);
composer.addPass(new RenderPass(scene, camera));
composer.addPass(new UnrealBloomPass(new THREE.Vector2(innerWidth, innerHeight), 0.6, 0.4, 0.85));
composer.addPass(new OutputPass());          // tone mapping + color space at the end
// loop: composer.render(); resize: composer.setSize(w, h)
```

The `postprocessing` npm package (pmndrs) merges effects into fewer passes — faster for multiple effects. WebGPU uses the node-based `PostProcessing` class.

## Custom shaders

`ShaderMaterial` with `uniforms: { uTime: { value: 0 } }` and GLSL. To tweak built-in PBR, use `material.onBeforeCompile = (shader) => { shader.vertexShader = shader.vertexShader.replace('#include <begin_vertex>', '...') }` (wind sway, dissolve). With WebGPU, write **TSL** node materials (`MeshStandardNodeMaterial` with `colorNode`, `positionNode`) — compiles to both WGSL and GLSL.

## Pitfalls

- Washed-out or too-dark colors: double sRGB conversion (setting `colorSpace` on normal maps, or adding a gamma pass after OutputPass).
- Everything black with `MeshStandardMaterial`: no lights and no environment.
- Changing `material.side`, adding a map, or toggling `fog` after first render requires `material.needsUpdate = true`.
