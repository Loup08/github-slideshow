---
name: threejs-gltf-loading
description: Load and use glTF/GLB models in three.js — GLTFLoader with Draco, Meshopt and KTX2 compression, LoadingManager progress, cloning skinned meshes with SkeletonUtils, AnimationMixer playback and crossfades, shadows/material fixes after load, and asset pipeline optimization with gltf-transform. Use when importing 3D models or animated characters into three.js.
---

# three.js glTF Loading

glTF 2.0 (`.glb` binary) is the standard format for three.js. Export from Blender with *Apply Modifiers*, +Y up, and animations as NLA actions.

## Loader with compression support

```ts
import * as THREE from 'three';
import { GLTFLoader } from 'three/addons/loaders/GLTFLoader.js';
import { DRACOLoader } from 'three/addons/loaders/DRACOLoader.js';
import { KTX2Loader } from 'three/addons/loaders/KTX2Loader.js';
import { MeshoptDecoder } from 'three/addons/libs/meshopt_decoder.module.js';

const manager = new THREE.LoadingManager();
manager.onProgress = (_url, loaded, total) => setProgress(loaded / total);

const draco = new DRACOLoader(manager).setDecoderPath('/draco/');      // copy from three/examples/jsm/libs/draco/
const ktx2 = new KTX2Loader(manager).setTranscoderPath('/basis/').detectSupport(renderer);

const gltfLoader = new GLTFLoader(manager)
  .setDRACOLoader(draco)
  .setKTX2Loader(ktx2)
  .setMeshoptDecoder(MeshoptDecoder);

const gltf = await gltfLoader.loadAsync('/models/hero.glb');
scene.add(gltf.scene);
```

## Post-load setup

```ts
gltf.scene.traverse((o) => {
  if ((o as THREE.Mesh).isMesh) {
    const m = o as THREE.Mesh;
    m.castShadow = true;
    m.receiveShadow = true;
  }
});
const box = new THREE.Box3().setFromObject(gltf.scene);    // size/center for placement
const named = gltf.scene.getObjectByName('Muzzle');        // empties from Blender as sockets
```

Colors look right by default (sRGB color management is on); don't override `texture.colorSpace` on glTF textures.

## Animation

```ts
const mixer = new THREE.AnimationMixer(gltf.scene);
const actions = Object.fromEntries(gltf.animations.map(c => [c.name, mixer.clipAction(c)]));
let current = actions['Idle'];
current.play();

function fadeTo(name: string, duration = 0.25) {
  const next = actions[name];
  if (next === current) return;
  next.reset().setEffectiveWeight(1).fadeIn(duration).play();
  current.fadeOut(duration);
  current = next;
}

actions['Jump'].setLoop(THREE.LoopOnce, 1);
actions['Jump'].clampWhenFinished = true;
mixer.addEventListener('finished', (e) => { if (e.action === actions['Jump']) fadeTo('Idle'); });

// in loop:
mixer.update(dt);
```

## Multiple instances

```ts
import * as SkeletonUtils from 'three/addons/utils/SkeletonUtils.js';
const enemy = SkeletonUtils.clone(gltf.scene);   // .clone() breaks skinned meshes' skeleton binding
```

Each clone needs its own `AnimationMixer`. For hundreds of static props, use `InstancedMesh` built from the source geometry/material.

## Caching

Load each file once, keep a `Map<string, GLTF>`, clone on demand. Enable `THREE.Cache.enabled = true` for repeat file loads.

## Optimize assets (CLI)

```bash
npx @gltf-transform/cli optimize in.glb out.glb --compress meshopt --texture-compress ktx2
npx @gltf-transform/cli inspect out.glb
```

- Meshopt or Draco for geometry; KTX2 (Basis) for GPU-compressed textures (huge VRAM savings).
- Resize textures (`--texture-size 1024`), join meshes sharing materials, prune unused nodes.
- `gltf.report` / three.js editor to inspect.

## Pitfalls

- Decoder path wrong → silent hangs/errors; serve decoder files from your public folder.
- Model too big/small: Blender unit scale vs export; fix at export, not by scaling at runtime every load.
- Dispose loaded resources when changing levels (geometries, materials, textures).
- Animation names differ across exports — log `gltf.animations.map(a => a.name)`.
