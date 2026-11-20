---
name: threejs-scene-setup
description: Set up three.js games and interactive 3D (r16x–r18x) — Vite project, renderer/scene/camera boilerplate, resize and pixel ratio, setAnimationLoop with Timer/Clock, OrbitControls and PointerLockControls, raycasting picking, WebGPURenderer vs WebGLRenderer, disposal, and physics integration (Rapier). Use when starting or structuring any three.js project.
---

# three.js Scene Setup

```bash
npm create vite@latest my3d -- --template vanilla-ts && cd my3d && npm i three && npm i -D @types/three
```

## Boilerplate

```ts
import * as THREE from 'three';
import { OrbitControls } from 'three/addons/controls/OrbitControls.js';

const renderer = new THREE.WebGLRenderer({ antialias: true, powerPreference: 'high-performance' });
renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2));   // cap for perf
renderer.setSize(window.innerWidth, window.innerHeight);
renderer.shadowMap.enabled = true;
renderer.shadowMap.type = THREE.PCFSoftShadowMap;
renderer.toneMapping = THREE.ACESFilmicToneMapping;               // or AgXToneMapping
document.body.appendChild(renderer.domElement);

const scene = new THREE.Scene();
scene.background = new THREE.Color(0x87a7c7);

const camera = new THREE.PerspectiveCamera(60, window.innerWidth / window.innerHeight, 0.1, 500);
camera.position.set(6, 5, 8);

const controls = new OrbitControls(camera, renderer.domElement);
controls.enableDamping = true;

scene.add(new THREE.HemisphereLight(0xffffff, 0x444466, 1.5));
const sun = new THREE.DirectionalLight(0xffffff, 3);
sun.position.set(10, 15, 5);
sun.castShadow = true;
scene.add(sun);

const ground = new THREE.Mesh(new THREE.PlaneGeometry(50, 50), new THREE.MeshStandardMaterial({ color: 0x556b2f }));
ground.rotation.x = -Math.PI / 2;
ground.receiveShadow = true;
scene.add(ground);

const timer = new THREE.Timer();      // r16x+: Timer (preferred) — or THREE.Clock
renderer.setAnimationLoop((t) => {
  timer.update(t);
  const dt = Math.min(timer.getDelta(), 0.05);   // clamp after tab switches
  update(dt);
  controls.update();
  renderer.render(scene, camera);
});

window.addEventListener('resize', () => {
  camera.aspect = window.innerWidth / window.innerHeight;
  camera.updateProjectionMatrix();
  renderer.setSize(window.innerWidth, window.innerHeight);
});
```

Use `setAnimationLoop` (required for WebXR, works with WebGPU) rather than raw `requestAnimationFrame`. If `THREE.Timer` isn't exported in the installed version, import it from `three/addons/misc/Timer.js` or use `Clock`.

## WebGPU

```ts
import * as THREE from 'three/webgpu';
const renderer = new THREE.WebGPURenderer({ antialias: true });
await renderer.init();
```

Falls back to WebGL2 automatically. Pairs with TSL (node materials). Don't mix `three` and `three/webgpu` imports in one app.

## Units & conventions

1 unit = 1 meter, Y up, right-handed, camera looks down -Z. Angles in radians (`THREE.MathUtils.degToRad`).

## Picking

```ts
const ray = new THREE.Raycaster();
const ndc = new THREE.Vector2();
renderer.domElement.addEventListener('pointerdown', (e) => {
  ndc.set((e.clientX / innerWidth) * 2 - 1, -(e.clientY / innerHeight) * 2 + 1);
  ray.setFromCamera(ndc, camera);
  const hit = ray.intersectObjects(pickables, true)[0];
  if (hit) onPick(hit.object, hit.point);
});
```

Raycast against a small list or low-poly proxies, not the whole scene. Use `three-mesh-bvh` for heavy meshes.

## First-person

`PointerLockControls` + manual WASD velocity; lock on click (`controls.lock()`). For collision, use a physics engine or capsule-vs-BVH (`three-mesh-bvh` character controller example).

## Physics

**Rapier** (`@dimforge/rapier3d-compat`): step a fixed 1/60 world in the loop, copy body transforms to meshes. Keep physics colliders simple (boxes/capsules/convex hulls), not render meshes.

## Structure

Entity objects with `update(dt)`; a `World` that owns scene + physics; `dispose()` on teardown:

```ts
mesh.geometry.dispose();
(mesh.material as THREE.Material).dispose();
texture.dispose();
renderer.renderLists.dispose();
```

GPU memory isn't GC'd — dispose when unloading levels. Check `renderer.info.memory` and `renderer.info.render.calls`.

## Performance

- Draw calls are the budget: `InstancedMesh` for repeats, merge static geometry (`BufferGeometryUtils.mergeGeometries`), shared materials.
- Shadows: one shadow-casting directional light, tight `shadow.camera` frustum, `shadow.mapSize` 1024–2048.
- `stats.js` / `renderer.info` for monitoring.
