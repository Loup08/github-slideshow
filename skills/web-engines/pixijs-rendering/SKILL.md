---
name: pixijs-rendering
description: 2D rendering with PixiJS v8 — async Application init, Assets loading and bundles, Sprites/Containers/Graphics with the v8 API, spritesheets and AnimatedSprite, Ticker-driven game loops, text (Text, BitmapText), filters, render layers, batching and performance, plus v7→v8 migration. Use for PixiJS games or 2D WebGL/WebGPU rendering.
---

# PixiJS v8

PixiJS is a renderer, not a full engine: you bring your own physics (e.g. a simple AABB system, matter-js, Rapier) and input handling.

## App setup

```ts
import { Application, Assets, Sprite, Container } from 'pixi.js';

const app = new Application();
await app.init({                       // v8: init is async; options go here, not in constructor
  resizeTo: window,
  background: '#1d1d2b',
  antialias: false,
  preference: 'webgl',                 // or 'webgpu'
  resolution: window.devicePixelRatio,
  autoDensity: true,
});
document.body.appendChild(app.canvas); // v8: app.canvas (not app.view)
```

## Assets

```ts
await Assets.init({ manifest: {
  bundles: [
    { name: 'game', assets: [
      { alias: 'hero', src: 'assets/hero.json' },       // spritesheet JSON (TexturePacker)
      { alias: 'tiles', src: 'assets/tiles.png' },
      { alias: 'font', src: 'assets/pixel.fnt' },
    ]},
  ],
}});
const game = await Assets.loadBundle('game', p => bar.scale.x = p);
```

Pixel art: `texture.source.scaleMode = 'nearest'` or set `TextureSource.defaultOptions.scaleMode = 'nearest'` before loading.

## Scene graph

```ts
const world = new Container();
app.stage.addChild(world);

const hero = new Sprite(Assets.get('tiles'));
hero.anchor.set(0.5);
hero.position.set(100, 100);
world.addChild(hero);

import { AnimatedSprite, Spritesheet } from 'pixi.js';
const sheet = Assets.get<Spritesheet>('hero');
const run = new AnimatedSprite(sheet.animations['run']);
run.animationSpeed = 0.15; run.play();
```

v8: only `Container` can have children (Sprites/Graphics are leaf nodes). Group with Containers.

## Graphics (v8 API)

```ts
import { Graphics } from 'pixi.js';
const g = new Graphics()
  .rect(0, 0, 50, 20).fill(0xff3355)
  .circle(80, 10, 10).fill({ color: 0xffffff, alpha: 0.5 })
  .moveTo(0, 30).lineTo(100, 30).stroke({ width: 2, color: 0x00ff00 });
```

Shape first, then `fill()`/`stroke()` (v7's `beginFill/drawRect/endFill` are deprecated). Reuse `GraphicsContext` for many identical shapes.

## Game loop

```ts
const FIXED = 1 / 60;
let acc = 0;
app.ticker.add((ticker) => {
  acc += ticker.deltaMS / 1000;          // ticker passed as argument in v8
  while (acc >= FIXED) { step(FIXED); acc -= FIXED; }
  render(acc / FIXED);                   // interpolate visuals
});
```

## Input

`sprite.eventMode = 'static'; sprite.on('pointerdown', e => ...)`. Keyboard: plain `window.addEventListener('keydown')` into an input state map. Set `eventMode = 'none'` on large non-interactive containers to skip hit-testing.

## Text

`Text` (canvas-rendered, re-rasterized on change — avoid updating every frame), `BitmapText` (fast for scores/damage numbers), `HTMLText` for rich formatting.

## Camera

Move a `world` Container: `world.pivot.set(cam.x, cam.y); world.position.set(app.screen.width/2, app.screen.height/2); world.scale.set(zoom)`. Keep UI in a separate container on `app.stage`.

## Performance

- Batch: same texture source (atlas) + same blend mode → one draw call. Avoid interleaving textures.
- `ParticleContainer` (v8 rewritten, uses `Particle` objects) for thousands of sprites.
- `cullable = true` + `Culler` / `cullArea` for large worlds.
- `cacheAsTexture()` (v8 replaces `cacheAsBitmap`) for static complex containers.
- Destroy what you remove: `sprite.destroy()`, `texture.destroy(true)`; `Assets.unload(alias)` between levels.

## Filters

`import { BlurFilter, ColorMatrixFilter } from 'pixi.js'` + `pixi-filters` package (Glow, CRT, Outline). Filters cost a render-texture pass each — apply to a container, not hundreds of sprites.
