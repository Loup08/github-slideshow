---
name: phaser-core
description: Build browser games with Phaser (v4, Phaser 3-compatible scene API) — game config and scaling, Scene lifecycle (preload/create/update), loader and texture atlases, sprites and animations, input (keyboard, pointer, gamepad), cameras, tweens, timers, events, scene switching, and Vite/TypeScript setup. Use for any Phaser project.
---

# Phaser Core

Phaser 4 keeps the Phaser 3 Scene / GameObject / Loader API with a new WebGL renderer; most Phaser 3 code ports directly. Visual effects moved from FX pipelines to **filters** — check the official v3→v4 migration guide when using custom pipelines/FX.

## Project setup

```bash
npm create vite@latest my-game -- --template vanilla-ts
cd my-game && npm i phaser
```

```ts
// src/main.ts
import Phaser from 'phaser';
import { Boot } from './scenes/Boot';
import { Game } from './scenes/Game';

new Phaser.Game({
  type: Phaser.AUTO,
  parent: 'app',
  width: 480,
  height: 270,
  pixelArt: true,                         // nearest filtering + roundPixels
  backgroundColor: '#1d1d2b',
  scale: { mode: Phaser.Scale.FIT, autoCenter: Phaser.Scale.CENTER_BOTH },
  physics: { default: 'arcade', arcade: { gravity: { x: 0, y: 900 }, debug: false } },
  scene: [Boot, Game],
});
```

## Scene lifecycle

```ts
export class Game extends Phaser.Scene {
  private player!: Phaser.Physics.Arcade.Sprite;
  private keys!: Record<'left' | 'right' | 'jump', Phaser.Input.Keyboard.Key>;

  constructor() { super('Game'); }

  init(data: { level: number }) { /* receives scene.start data */ }

  preload() {   // usually done in a Boot/Preloader scene instead
    this.load.atlas('hero', 'assets/hero.png', 'assets/hero.json');
    this.load.tilemapTiledJSON('level1', 'assets/level1.json');
    this.load.audio('jump', ['assets/jump.ogg', 'assets/jump.mp3']);
  }

  create() {
    this.anims.create({
      key: 'run',
      frames: this.anims.generateFrameNames('hero', { prefix: 'run_', start: 0, end: 5 }),
      frameRate: 12, repeat: -1,
    });
    this.player = this.physics.add.sprite(64, 64, 'hero', 'idle_0');
    this.keys = this.input.keyboard!.addKeys({ left: 'A', right: 'D', jump: 'SPACE' }) as any;
    this.cameras.main.startFollow(this.player, true, 0.1, 0.1);
  }

  update(time: number, delta: number) {
    // per-frame logic; delta in ms
  }
}
```

Lifecycle: `init → preload → create → update(time, delta)`; scene events `shutdown` (on stop) and `destroy`.

## Loading screen

Do loading in a `Preloader` scene: `this.load.on('progress', (p: number) => bar.width = 200 * p);` then `this.scene.start('Menu')` in `create`. Use **texture atlases** (TexturePacker / free-tex-packer) instead of many single images.

## Input

- Keyboard: `this.input.keyboard!.createCursorKeys()`, `Phaser.Input.Keyboard.JustDown(key)`.
- Pointer: `sprite.setInteractive(); sprite.on('pointerdown', fn)`; `this.input.on('pointermove', (p) => p.worldX)`.
- Gamepad: enable `input: { gamepad: true }` in config, then `this.input.gamepad!.once('connected', pad => ...)`.

## Tweens, timers, events

```ts
this.tweens.add({ targets: coin, y: coin.y - 8, yoyo: true, repeat: -1, duration: 400, ease: 'Sine.inOut' });
this.time.addEvent({ delay: 1000, loop: true, callback: this.spawnEnemy, callbackScope: this });
this.time.delayedCall(300, () => this.scene.restart());
this.events.emit('score', 10);                  // scene-local bus
this.game.events.emit('global-thing');          // game-wide
this.registry.set('score', 0);                  // shared data across scenes ('changedata-score' event)
```

## Scenes management

`this.scene.start('Game', { level: 2 })` (stop current, start other), `this.scene.launch('HUD')` (run in parallel — common for UI), `this.scene.pause()/resume()`, `this.scene.stop()`. A HUD scene with its own camera avoids UI scrolling with the world (or use `setScrollFactor(0)`).

## Cameras

`this.cameras.main.setBounds(0, 0, mapW, mapH)`, `.setZoom(2)`, `.shake(100, 0.01)`, `.flash()`, `.fadeOut(300)` + `'camerafadeoutcomplete'` event.

## Pitfalls

- Creating objects in `update` without destroying → leaks. Use Groups with `maxSize` and `get()` for pooling.
- Arrow functions vs `callbackScope` — use arrows to keep `this`.
- Audio only starts after a user gesture (Phaser unlocks automatically on first input; show a "click to start" menu).
- Event listeners on `this.game.events` survive scene restarts — remove them on `shutdown`.
