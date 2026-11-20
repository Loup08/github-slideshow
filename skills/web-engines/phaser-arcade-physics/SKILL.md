---
name: phaser-arcade-physics
description: Phaser Arcade Physics — dynamic and static bodies, colliders vs overlaps, groups and pooling, tilemap collision with Tiled maps, platformer movement (coyote time, jump buffer, variable jump), one-way platforms, body sizing, and world bounds. Use for movement and collisions in Phaser games.
---

# Phaser Arcade Physics

Arcade = fast AABB/circle physics, no rotation-based collision. Use Matter.js (`physics: { default: 'matter' }`) only when you need polygons, joints, or realistic stacking.

## Bodies

```ts
const player = this.physics.add.sprite(x, y, 'hero');
player.setCollideWorldBounds(true);
player.body!.setSize(10, 14).setOffset(3, 2);   // trim hitbox to the sprite's feet/body
player.setMaxVelocity(200, 600);
player.setDragX(1200);                          // friction when no acceleration

const platforms = this.physics.add.staticGroup();
platforms.create(240, 260, 'ground').refreshBody();   // refresh after scale/position change
```

Dynamic body: moved by velocity/acceleration. Static body: never moves (call `refreshBody()` after changing). Immovable dynamic body (`setImmovable(true)` + `body.allowGravity = false`) for moving platforms.

## Collide vs overlap

```ts
this.physics.add.collider(player, platforms);                       // separates
this.physics.add.overlap(player, coins, (_p, c) => {                // detects only
  (c as Phaser.Physics.Arcade.Sprite).disableBody(true, true);
  this.registry.inc('score', 1);
});
this.physics.add.collider(bullets, enemies, onHit, (b, e) => (e as any).alive);  // processCallback filter
```

## Platformer controller

```ts
const SPEED = 160, ACCEL = 1200, JUMP_V = -330, COYOTE = 100, BUFFER = 120;
let coyoteMs = 0, bufferMs = 0;

update(_t: number, dt: number) {
  const body = this.player.body as Phaser.Physics.Arcade.Body;
  const onGround = body.blocked.down || body.touching.down;
  const dir = (this.keys.right.isDown ? 1 : 0) - (this.keys.left.isDown ? 1 : 0);

  this.player.setAccelerationX(dir * ACCEL);
  if (dir === 0) this.player.setAccelerationX(0);       // drag takes over
  if (dir !== 0) this.player.setFlipX(dir < 0);
  body.setMaxVelocityX(SPEED);

  coyoteMs = onGround ? COYOTE : coyoteMs - dt;
  bufferMs = Phaser.Input.Keyboard.JustDown(this.keys.jump) ? BUFFER : bufferMs - dt;

  if (bufferMs > 0 && coyoteMs > 0) {
    this.player.setVelocityY(JUMP_V);
    bufferMs = coyoteMs = 0;
  }
  if (!this.keys.jump.isDown && body.velocity.y < 0) body.velocity.y *= 0.85;  // variable height
  body.setGravityY(body.velocity.y > 0 ? 600 : 0);                              // extra fall gravity
}
```

## Tilemaps (Tiled)

```ts
const map = this.make.tilemap({ key: 'level1' });
const tiles = map.addTilesetImage('tileset-name-in-tiled', 'tiles', 16, 16, 1, 2); // margin/spacing if extruded
const ground = map.createLayer('Ground', tiles!)!;
ground.setCollisionByProperty({ collides: true });     // custom bool property set in Tiled
this.physics.add.collider(this.player, ground);
this.physics.world.setBounds(0, 0, map.widthInPixels, map.heightInPixels);

// spawn objects from an object layer
map.getObjectLayer('Entities')!.objects.forEach(o => { if (o.name === 'spawn') this.player.setPosition(o.x!, o.y!); });
```

Extrude tilesets (`tile-extruder`) to stop bleeding lines.

One-way platforms: `tile.setCollision(false, false, true, false)` (only top), or for sprites set `body.checkCollision.down = false; body.checkCollision.left = body.checkCollision.right = false`.

## Pools

```ts
this.bullets = this.physics.add.group({ classType: Bullet, maxSize: 64, runChildUpdate: true });
const b = this.bullets.get(x, y) as Bullet | null;
if (b) b.fire(dir);
// Bullet: when off-screen → this.disableBody(true, true)
```

## Tunneling & stability

- Fast small objects tunnel through thin tiles: lower speed, thicker colliders, or raise `fps` in `arcade: { fps: 120 }`.
- `this.physics.world.TILE_BIAS` (default 16) increase if falling through tiles at high speed.
- Use `fixedStep: true` (default) for deterministic-ish simulation.

## Debug

`arcade: { debug: true }` draws bodies and velocity; toggle at runtime with `this.physics.world.drawDebug`.
